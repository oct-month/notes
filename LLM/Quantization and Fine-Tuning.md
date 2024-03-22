# LLM Quantization and Fine-Tuning

## 前置条件

- Ubuntu 20.04/22.04
- Linux Kernel >= 5.5.0，如果使用的是20.04，可能需要安装`linux-generic-hwe-20.04`以使用更新的内核
- Python >= 3.10，如果默认的版本较低，可以通过`pyenv`来管理多版本Python，以安装高版本
- CUDA 11.8，cudnn 8.9.7

> 硬件支持（Hardware Enablement, HWE）的内核会始终使用更新的版本，比`linux-generic`的内核版本高，但稳定性可能略低。

## FT方法分类

### Parameter Efficient Model Fine-Tuning (PEFT)

保持整个预训练模型参数冻结，并仅向模型中添加微小的可学习参数/层。通过这种方式，只需要训练一小部分参数。目前的方法有`LoRA`、`QLoRA`、`LLaMA Adapter`和`Prefix-tuning`。

现有的PEFT库：https://github.com/huggingface/peft

### Full/Partial Parameter Fine-Tuning

这种微调方式有三种策略可选：

- 保持预训练模型冻结，只微调任务头。
- 保持预训练模型冻结，只在顶部添加几个全连接层。
- 对所有层进行微调。

<div style="display: flex;">
    <img src="./.img/feature-based_FN.png" alt="Image 1" width="250" />
    <img src="./.img/feature-based_FN_2.png" alt="Image 2" width="250" />
    <img src="./.img/full-param-FN.png" alt="Image 3" width="250" />
</div>
在这种情况下，可能需要不止一个GPU进行训练。这需要足够的GPU显存来保存模型参数、梯度和优化器状态。

## Fully Sharded Data Parallel (FSDP)

FSDP是PyTorch中的一个包，用于训练需要多个GPU的模型。

在FSDP之前是`Distributed Data Parallel (DDP)`，它使得每个GPU都保存完整的模型，只对数据进行分片，在反向传播结束后同步梯度。

而FSDP不仅对数据进行分片，还对模型参数、梯度和优化器状态进行分片，每个GPU都只保存模型的一个分片。这样可以节省大量显存，以便于将更大的模型放入多个GPU中。

安装方式：

```shell
pip3 install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu118
```

## Llama 2 微调

由于国内网络情况的特殊性，数据集和大模型均需使用本地路径，而这需要修改`llama_recipes`包。

1. 工程目录创建和依赖安装

```shell
mkdir peft-demo
cd peft-demo
# 依赖安装
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu118
pip install packaging ninja
MAX_JOBS=16 pip install flash-attn --no-build-isolation  # MAX_JOBS决定了编译并行数量和内存占用
pip install accelerate appdirs loralib bitsandbytes black "black[jupyter]" datasets fire peft "transformers>=4.34.1" sentencepiece py7zr scipy optimum matplotlib gradio scikit-learn trl
# llama_recipes包获取
git clone https://github.com/meta-llama/llama-recipes.git
mv ./llama-recipes/src/llama_recipes ./
mv ./llama-recipes/recipes/finetuning/finetuning.py ./
rm -r llama-recipes
# trl脚本获取
git clone https://github.com/huggingface/trl.git
mv trl/examples/scripts/sft.py ./
rm -r trl
# qlora脚本获取
git clone https://github.com/artidoro/qlora.git
# TODO
# 数据集目录创建
mkdir datasets
```

2. 数据集准备

下载后均放在`./datasets/`目录下。

alpaca：https://github.com/tatsu-lab/stanford_alpaca/raw/main/alpaca_data.json

samsum：https://huggingface.co/datasets/samsum/tree/main

```python
# datasets/samsum/samsum.py 修改78行
def _split_generators(self, dl_manager):
    """Returns SplitGenerators."""
    path = '/home/sun/work/peft-demo/datasets/samsum/data/corpus.7z'  # corpus.7z的路径
```

grammar：

https://huggingface.co/datasets/jhu-clsp/jfleg/tree/main

https://huggingface.co/datasets/liweili/c4_200m/tree/main

```python
# datasets/c4_200m/c4_200m.py 修改68行
def _split_generators(self, dl_manager):
    """Returns SplitGenerators."""
    data_dir = '/home/sun/work/peft-demo/datasets/c4_200m/'  # c4_200m的路径
```

```shell
# 需要合并
python grammar_gen.py
cd datasets
mkdir grammar
mv ../grammar_validation.csv ./grammar/
mv ../gtrain_10k.csv ./grammar/
cd ..
```

```python
# grammar_gen.py
import csv
from datasets import load_dataset
from pathlib import Path
import pandas as pd

list_replacements = [
  (" .", "."), 
  (" ,", ","),
  (" '", "'"),
  (" ?", "?"),
  (" !", "!"),
  (" :", ":"),
  (" ;", ";"),
  (" n't", "n't"),
  (" v", "v"),
  ("2 0 0 6", "2006"),
  ("5 5", "55"),
  ("4 0 0", "400"),
  ("1 7-5 0", "1750"),
  ("2 0 %", "20%"),
  ("5 0", "50"),
  ("1 2", "12"),
  ("1 0", "10"),
  ('" ballast water', '"ballast water')
]

def correct_spacing(item):
    """ we iterate through the list of all replacements per each item in dataset"""
    for fix in list_replacements:
        item = item.replace(fix[0], fix[1])
    return item

def generate_csv(csv_path, dataset):
    """ apply spacing corrections and save out matched pairs to csv file as dataset"""
    with open(csv_path, 'w', newline='') as csvfile:
        writer = csv.writer(csvfile)
        writer.writerow(["input", "target"])
        for case in dataset:
     	    # Adding the t5 task indication prefix to input 
            input_text = case["sentence"]
            input_text = correct_spacing(input_text)

            for correction in case["corrections"]:
                correction = correct_spacing(correction)
                # a few of the cases contain blank strings. 
                if input_text and correction:
                    writer.writerow([input_text, correction])

def c4_generate_csv(csv_path, iterator, num_examples):
    with open(csv_path, 'w', newline='') as csvfile:
        writer = csv.writer(csvfile)
        writer.writerow(["input", "target"])
        for i in range(0,num_examples):
            data = next(iterator)
            input_text = data["input"]
            input_text = correct_spacing(input_text)
            correction = correct_spacing(data["output"])
            if input_text and correction:
                writer.writerow([input_text, correction])


train_dataset = load_dataset("./datasets/jfleg/", split='validation[:]') 
eval_dataset = load_dataset("./datasets/jfleg/", split='test[:]')

clean22 = correct_spacing(train_dataset['sentence'][22])

jfleg_dir = Path.cwd()/'jfleg_dataset'  # if you only use 'jfleg', hf will try and use that and complain
jfleg_dir.mkdir(parents=True,exist_ok=True)
c4_dir = Path.cwd()/'c4_dataset'
c4_dir.mkdir(parents=True,exist_ok=True)

j_train_file = jfleg_dir/'jtrain.csv'
j_eval_file = jfleg_dir/'jeval.csv'

generate_csv(j_train_file, train_dataset)
generate_csv(j_eval_file, eval_dataset)

c4_dataset = load_dataset("./datasets/c4_200m", streaming = True)

iterator = iter(c4_dataset['train'])

c4_dir = Path.cwd()/'c4_dataset'
c4_dir.mkdir(parents=True,exist_ok=True)
c4_filename = c4_dir/'c4train_10k.csv'

c4_generate_csv(c4_filename, iterator, num_examples=10000)

merge_list = [j_train_file, c4_filename, ]

combined_csv = pd.concat([pd.read_csv(fn) for fn in merge_list])

merged_name = "gtrain_10k.csv"
combined_csv.to_csv(merged_name, index=False, encoding = 'utf-8-sig', )
eval_name = "grammar_validation.csv"
eval_csv = pd.read_csv(j_eval_file)
eval_csv.to_csv(eval_name, index=False, encoding = 'utf-8-sig', )
```

openassistant-guanaco：https://huggingface.co/datasets/timdettmers/openassistant-guanaco/tree/main

3. llama_recipes模块修改

```python
# llama_recipes/configs/datasets.py
# Copyright (c) Meta Platforms, Inc. and affiliates.
# This software may be used and distributed according to the terms of the Llama 2 Community License Agreement.
from dataclasses import dataclass

@dataclass
class samsum_dataset:
    dataset: str =  "samsum_dataset"
    train_split: str = "train"
    test_split: str = "validation"
    data_path: str = "datasets/samsum/"

@dataclass
class grammar_dataset:
    dataset: str = "grammar_dataset"
    train_split: str = "datasets/grammar/gtrain_10k.csv" 
    test_split: str = "datasets/grammar/grammar_validation.csv"

@dataclass
class alpaca_dataset:
    dataset: str = "alpaca_dataset"
    train_split: str = "train"
    test_split: str = "val"
    data_path: str = "datasets/alpaca_data.json"

@dataclass
class custom_dataset:
    dataset: str = "custom_dataset"
    file: str = "examples/custom_dataset.py"
    train_split: str = "train"
    test_split: str = "validation"
```

4. 开始LoRA微调

```shell
# 这里是使用了两张GPU进行微调，数据集的设置在最后一行
torchrun --nnodes 1 --nproc_per_node 2 finetuning.py \
    --enable_fsdp \
    --model_name ../Llama-2-7b-hf/ \
    --use_peft \
    --peft_method lora \
    --output_dir ../Llama-2-7b-my-lora/ \
    --use_fast_kernels \
    --batch_size_training 2 \
    --gradient_accumulation_steps 2 \
    --dataset grammar_dataset #samsum_dataset #alpaca_dataset
```

```shell
# 另一个微调脚本
torchrun --nnodes 1 --nproc_per_node 2 sft.py \
    --model_name ../Llama-2-7b-hf/ \
    --dataset_name ./datasets/openassistant-guanaco \
    --output_dir ../Llama-2-7b-my-lora/ \
    --learning_rate=1.41e-5 \
    --per_device_train_batch_size=64 \
    --gradient_accumulation_steps=2 \
    --logging_steps=1 \
    --num_train_epochs=3 \
    --max_steps=-1 \
    --gradient_checkpointing \
    --use_peft \
    --lora_r=64 \
    --lora_alpha=16 \
    --batch_size=4 \
    --load_in_4bit
```

5. 尝试QLoRA微调





