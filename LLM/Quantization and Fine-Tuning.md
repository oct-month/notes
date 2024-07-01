# LLM Quantization and Fine-Tuning

## 前置条件

- Ubuntu 20.04/22.04
- Linux Kernel >= 5.5.0，如果使用的是20.04，可能需要安装`linux-generic-hwe-20.04`以使用更新的内核
- Python >= 3.10，如果默认的版本较低，可以通过`pyenv`来管理多版本Python，以安装高版本
- CUDA 11.8，cudnn 8.9.7

> 硬件支持（Hardware Enablement, HWE）的内核会始终使用更新的版本，比`linux-generic`的内核版本高，但稳定性可能略低。

## FT方法分类

### Parameter Efficient Model Fine-Tuning (PEFT)

保持整个预训练模型参数冻结，并仅向模型中添加微小的可学习参数/层。通过这种方式，只需要训练一小部分参数。目前的方法有`LoRA`、`QLoRA`、`LLaMA Adapter`和`Prefix-tuning`等。

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
mv ./trl/examples/scripts/sft.py ./
rm -r trl
# qlora脚本获取
git clone https://github.com/artidoro/qlora.git
mv ./qlora/qlora.py ./
rm -r qlora
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

mmlu：https://github.com/artidoro/qlora/tree/main/data/mmlu

3. 评估模块下载

accuracy：https://huggingface.co/spaces/evaluate-metric/accuracy/tree/main

放在工程根目录下

4. llama_recipes模块/qlora脚本 修改

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

```python
# qlora.py
# 588行
elif dataset_name == 'oasst1':
    return load_dataset("./datasets/openassistant-guanaco")
# 725-733行
if args.mmlu_dataset == 'mmlu-zs':
    mmlu_dataset = load_dataset("json", data_files={
        'eval': 'datasets/mmlu/zero_shot_mmlu_val.json',
        'test': 'datasets/mmlu/zero_shot_mmlu_test.json',
    })
    mmlu_dataset = mmlu_dataset.remove_columns('subject')
# MMLU Five-shot (Eval/Test only)
elif args.mmlu_dataset == 'mmlu' or args.mmlu_dataset == 'mmlu-fs':
    mmlu_dataset = load_dataset("json", data_files={
        'eval': 'datasets/mmlu/five_shot_mmlu_val.json',
        'test': 'datasets/mmlu/five_shot_mmlu_test.json',
    })
# 745行
accuracy = evaluate.load("./accuracy")
```

5. 开始LoRA微调

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

6. 尝试QLoRA微调

如果报错，可能需要调整依赖包：https://github.com/artidoro/qlora/blob/main/requirements.txt 。

或者尝试修改`qlora.py`文件。

```shell
python qlora.py \
    --model_name_or_path ../Llama-2-13b-hf/ \
    --use_auth \
    --output_dir ../Llama-2-13b-my-lora/ \
    --logging_steps 10 \
    --save_strategy steps \
    --data_seed 42 \
    --save_steps 500 \
    --save_total_limit 40 \
    --evaluation_strategy steps \
    --eval_dataset_size 1024 \
    --max_eval_samples 1000 \
    --per_device_eval_batch_size 1 \
    --max_new_tokens 32 \
    --dataloader_num_workers 1 \
    --group_by_length \
    --logging_strategy steps \
    --remove_unused_columns False \
    --do_train \
    --do_eval \
    --do_mmlu_eval \
    --lora_r 64 \
    --lora_alpha 16 \
    --lora_modules all \
    --double_quant \
    --quant_type nf4 \
    --bf16 \
    --bits 8 \
    --warmup_ratio 0.03 \
    --lr_scheduler_type constant \
    --gradient_checkpointing \
    --dataset oasst1 \
    --source_max_len 16 \
    --target_max_len 512 \
    --per_device_train_batch_size 1 \
    --gradient_accumulation_steps 16 \
    --max_steps 1875 \
    --eval_steps 187 \
    --learning_rate 0.0002 \
    --adam_beta2 0.999 \
    --max_grad_norm 0.3 \
    --lora_dropout 0.1 \
    --weight_decay 0.0 \
    --seed 0
```

（经测试，双3090微调，显存总占用大约21GB）

## Chinese-LLaMA-Alpaca-2 微调

1. 环境配置

```shell
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
pip install packaging ninja
MAX_JOBS=16 pip install flash-attn --no-build-isolation
pip install peft transformers accelerate sentencepiece bitsandbytes datasets deepspeed
```

2. 微调脚本获取

```shell
git clone https://github.com/ymcui/Chinese-LLaMA-Alpaca-2.git
mv ./Chinese-LLaMA-Alpaca-2/scripts/merge_llama2_with_chinese_lora_low_mem.py ./
mv ./Chinese-LLaMA-Alpaca-2/scripts/inference/inference_hf.py ./
mv ./Chinese-LLaMA-Alpaca-2/scripts/attn_and_long_ctx_patches.py ./
mv ./Chinese-LLaMA-Alpaca-2/scripts/training/build_dataset.py ./
mv ./Chinese-LLaMA-Alpaca-2/scripts/training/run_clm_sft_with_peft.py ./
mv ./Chinese-LLaMA-Alpaca-2/scripts/training/ds_zero2_no_offload.json ./
rm -r Chinese-LLaMA-Alpaca-2
```

```python
# run_clm_sft_with_peft.py
# 第68行（以safetensors格式保存会导致无法合并）
kwargs["model"].save_pretrained(peft_model_path, safe_serialization=False)
# 第77行
kwargs["model"].save_pretrained(peft_model_path, safe_serialization=False)
# 第405行
model = LlamaForCausalLM.from_pretrained(
    model_args.model_name_or_path,
    config=config,
    cache_dir=model_args.cache_dir,
    revision=model_args.model_revision,
    use_auth_token=True if model_args.use_auth_token else None,
    torch_dtype=torch_dtype,
    low_cpu_mem_usage=True,
    device_map=device_map,
    # 删除
    quantization_config=quantization_config,
    # 修改
    attn_implementation="flash_attention_2" if training_args.use_flash_attention_2 else None
)
```

3. 数据集相关

`chinese-llama-2`系列模型使用了纯文本格式的数据集文件，均为`txt`格式。

`chinese-alpaca-2`系列模型则使用了Stanford Alpaca格式的数据集文件，均为`json`格式。

```json
[
    {
        "instruction": "描述模型应该执行的任务",
        "input": "任务的可选上下文或输入",
        "output": "期望生成的回答"
    },
    ...
]
```

数据集可以放在`datasets/mydata`目录下，`datasets/mydata/train`下放多个json文件，`datasets/mydata/val`下放一个验证用的json文件。

4. 开始微调

```shell
# run_sft.sh
# 运行脚本前请仔细阅读wiki(https://github.com/ymcui/Chinese-LLaMA-Alpaca-2/wiki/sft_scripts_zh)
# Read the wiki(https://github.com/ymcui/Chinese-LLaMA-Alpaca-2/wiki/sft_scripts_zh) carefully before running the script
lr=1e-4
lora_rank=64
lora_alpha=128
lora_trainable="q_proj,v_proj,k_proj,o_proj,gate_proj,down_proj,up_proj"
modules_to_save="embed_tokens,lm_head"
lora_dropout=0.05

pretrained_model=../chinese-alpaca-2-13b/
chinese_tokenizer_path=../chinese-alpaca-2-13b/
dataset_dir=./datasets/mydata/train/
per_device_train_batch_size=1
per_device_eval_batch_size=1
gradient_accumulation_steps=8
max_seq_length=512
output_dir=./chinese-alpaca-2-13b-my-lora/
validation_file=./datasets/mydata/val/alpaca_data_val.json

deepspeed_config_file=./ds_zero2_no_offload.json

torchrun --nnodes 1 --nproc_per_node 2 run_clm_sft_with_peft.py \
    --use_flash_attention_2 \
    --deepspeed ${deepspeed_config_file} \
    --model_name_or_path ${pretrained_model} \
    --tokenizer_name_or_path ${chinese_tokenizer_path} \
    --dataset_dir ${dataset_dir} \
    --per_device_train_batch_size ${per_device_train_batch_size} \
    --per_device_eval_batch_size ${per_device_eval_batch_size} \
    --do_train \
    --do_eval \
    --seed 0 \
    --fp16 \
    --num_train_epochs 1 \
    --lr_scheduler_type cosine \
    --learning_rate ${lr} \
    --warmup_ratio 0.03 \
    --weight_decay 0 \
    --logging_strategy steps \
    --logging_steps 10 \
    --save_strategy steps \
    --save_total_limit 3 \
    --evaluation_strategy steps \
    --eval_steps 100 \
    --save_steps 200 \
    --gradient_accumulation_steps ${gradient_accumulation_steps} \
    --preprocessing_num_workers 8 \
    --max_seq_length ${max_seq_length} \
    --output_dir ${output_dir} \
    --overwrite_output_dir \
    --ddp_timeout 30000 \
    --logging_first_step True \
    --lora_rank ${lora_rank} \
    --lora_alpha ${lora_alpha} \
    --trainable ${lora_trainable} \
    --lora_dropout ${lora_dropout} \
    --torch_dtype float16 \
    --validation_file ${validation_file} \
    --load_in_kbits 8 \
    --save_safetensors False \
    --gradient_checkpointing \
    --ddp_find_unused_parameters False
    # --modules_to_save ${modules_to_save}
```

微调完成的模型存储在`./chinese-alpaca-2-13b-my-lora/sft_lora_model`目录下。

（经测试，双3090微调，显存总占用大约37GB）

5. 模型合并

```shell
python merge_llama2_with_chinese_lora_low_mem.py --base_model ../chinese-alpaca-2-13b/ --lora_model ./chinese-alpaca-2-13b-my-lora/sft_lora_model --output_type huggingface --output_dir ./chinese-alpaca-2-13b-my/ --verbose
```

6. 模型运行

```shell
python inference_hf.py --base_model ./chinese-alpaca-2-13b-my/ --with_prompt --interactive --gpus 0,1
```

## Chinese-LLaMA-Alpaca-2 量化

1. llama.cpp编译

生成`./main`（推理）和`./quantize`（量化）可执行文件。

```shell
git clone --recursive https://github.com/ggerganov/llama.cpp
cd llama
pip install torch torchvision torchaudio xformers --index-url https://download.pytorch.org/whl/cu118
pip install "einops~=0.7.0" "numpy~=1.24.4" "sentencepiece~=0.1.98" "transformers>=4.35.2,<5.0.0" "gguf>=0.1.0" "protobuf>=4.21.0,<5.0.0"
make LLAMA_CUBLAS=1
```

2. gguf格式转换

```shell
# 模型文件夹放在models目录下
python convert.py ./models/chinese-alpaca-2-13b/
# 默认生成FP32格式，可以添加“--outtype f16”选项修改为FP16
```

3. Q4_K量化

这里是进行了4-bit量化，推理速度较快。需要更强的推理性能可以考虑`Q6_K`或`Q8_0`。

```shell
./quantize ./models/chinese-alpaca-2-13b/ggml-model-f32.gguf ./models/chinese-alpaca-2-13b/ggml-model-Q4_K_M.gguf Q4_K_M
```

4. 运行模型

chat.sh：

```shell
#!/bin/bash

# temporary script to chat with Chinese Alpaca-2 model
# usage: ./chat.sh alpaca2-ggml-model-path your-first-instruction

SYSTEM_PROMPT='You are a helpful assistant. 你是一个乐于助人的助手。'
# SYSTEM_PROMPT='You are a helpful assistant. 你是一个乐于助人的助手。请你提供专业、有逻辑、内容真实、有价值的详细回复。' # Try this one, if you prefer longer response.
MODEL_PATH=$1
FIRST_INSTRUCTION=$2

./main -m "$MODEL_PATH" \
-ngl 40 -f ./prompts/alpaca.txt -n 2048 \
--color -i -c 4096 -t 24 --temp 0.5 --top-k 40 --top-p 0.9 --repeat-penalty 1.1 \
--in-prefix-bos --in-prefix ' [INST] ' --in-suffix ' [/INST]' -p \
"[INST] <<SYS>>
$SYSTEM_PROMPT
<</SYS>>

$FIRST_INSTRUCTION [/INST]"
```

相关参数说明：

```
-c 控制上下文的长度，值越大越能参考更长的对话历史（默认：512）
-f 指定prompt模板，alpaca模型请加载prompts/alpaca.txt
-n 控制回复生成的最大长度（默认：128）
-b 控制batch size（默认：512）
-t 控制线程数量（默认：8），可适当增加
--repeat_penalty 控制生成回复中对重复文本的惩罚力度
--temp 温度系数，值越低回复的随机性越小，反之越大
--top_p, top_k 控制解码采样的相关参数
```

开始聊天：

```shell
chmod a+x chat.sh
./chat.sh ./models/chinese-alpaca-2-13b/ggml-model-Q4_K_M.gguf "请列举5条发生火灾时的应对建议"
```

## ChatGLM3 微调

1. 环境配置

```shell
sudo apt-get install mpi-default-dev
# 确保CUDA已安装
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
pip install -r requirements.txt
```

```
# requirements.txt

# base
protobuf>=4.25.3
transformers>=4.39.3
tokenizers>=0.15.0
cpm_kernels>=1.0.11
torch>=2.1.0
gradio>=4.26.0
sentencepiece>=0.2.0
sentence_transformers>=2.4.0
accelerate>=0.29.2
streamlit>=1.33.0
fastapi>=0.110.0
loguru~=0.7.2
mdtex2html>=1.3.0
latex2mathml>=3.77.0
jupyter_client>=8.6.1

# ft
jieba>=0.42.1
ruamel_yaml>=0.18.6
rouge_chinese>=1.0.3
jupyter>=1.0.0
datasets>=2.18.0
peft>=0.10.0
deepspeed==0.13.1
mpi4py>=3.1.5
nltk
```

2. 代码获取

```shell
git clone https://github.com/THUDM/ChatGLM3.git
mv ./ChatGLM3/finetune_demo/configs/ ./
mv ./ChatGLM3/finetune_demo/inference_hf.py ./
mv ./ChatGLM3/finetune_demo/finetune_hf.py ./
rm -r ChatGLM3
mkdir data
mkdir output
```

3. 数据集下载和转换

下载地址：https://cloud.tsinghua.edu.cn/f/b3f119a008264b1cabd1/?dl=1

解压于：`./data/AdvertiseGen`

**转换**：

```shell
cd data/AdvertiseGen
mv dev.json dev.jsonl
mv train.json train.jsonl
python trans.py
cd ../..
```

```python
# data/AdvertiseGen/trans.py
import json
import copy

data_temp = {
    "conversations": [
        {
            "role": "user",
            "content": ""
        },
        {
            "role": "assistant",
            "content": ""
        }
    ]
}

dev_data = []
with open("./dev.jsonl", "r", encoding="UTF-8") as f:
    for d in f.readlines():
        j = json.loads(d)
        qus = j.get("content")
        res = j.get("summary")
        if qus and res:
            data_temp["conversations"][0]["content"] = qus
            data_temp["conversations"][1]["content"] = res
            dev_data.append(copy.deepcopy(data_temp))
with open("./dev.json", "w", encoding="UTF-8") as f:
    json.dump(dev_data, f, indent=2, ensure_ascii=False)

train_data = []
with open("./train.jsonl", "r", encoding="UTF-8") as f:
    for d in f.readlines():
        j = json.loads(d)
        qus = j.get("content")
        res = j.get("summary")
        if qus and res:
            data_temp["conversations"][0]["content"] = qus
            data_temp["conversations"][1]["content"] = res
            train_data.append(copy.deepcopy(data_temp))
with open("./train.json", "w", encoding="UTF-8") as f:
    json.dump(train_data, f, indent=2, ensure_ascii=False)
```

4. LoRA微调

```shell
OMP_NUM_THREADS=1 torchrun --standalone --nnodes=1 --nproc_per_node=2 finetune_hf.py \
    data/AdvertiseGen/ \
    ../chatglm3-6b \
    configs/lora.yaml
```

微调后的模型文件保存在`./output`目录下。

`P-Turning v2`微调则需使用`configs/ptuning_v2.yaml`配置文件。

**从保存点继续微调**：

```shell
# 从最后一个checkpoint继续微调
OMP_NUM_THREADS=1 torchrun --standalone --nnodes=1 --nproc_per_node=2 finetune_hf.py \
    data/AdvertiseGen/ \
    ../chatglm3-6b \
    configs/lora.yaml \
    yes
# 从指定checkpoint继续微调
OMP_NUM_THREADS=1 torchrun --standalone --nnodes=1 --nproc_per_node=2 finetune_hf.py \
    data/AdvertiseGen/ \
    ../chatglm3-6b \
    configs/lora.yaml \
    500
```

5. 模型测试

```shell
python inference_hf.py ./output/checkpoint-3000/ --prompt "类型#裤*版型#宽松*风格#青春*风格#性感*图案#创意*裤腰型#高腰*裤口#毛边"
```

## GLM4-9B-Chat 微调

1. 环境配置

```shell
git clone https://github.com/THUDM/GLM-4.git
cd GLM-4
pip config set global.extra-index-url https://download.pytorch.org/whl/cu118
pip install -r basic_demo/requirements.txt
pip install -r finetune_demo/requirements.txt
cd finetune_demo
```

2. 数据集下载和转换

下载地址：https://cloud.tsinghua.edu.cn/f/b3f119a008264b1cabd1/?dl=1

下载地址2：https://huggingface.co/datasets/shibing624/AdvertiseGen/tree/main

解压于：`./data/AdvertiseGen`

**转换**：

```python
# data/AdvertiseGen/trans.py
import json
import copy

data_temp = {
    "messages": [
        {
            "role": "user",
            "content": ""
        },
        {
            "role": "assistant",
            "content": ""
        }
    ]
}

dev_data = []
with open("./dev.json", "r", encoding="UTF-8") as f:
    for d in f.readlines():
        j = json.loads(d)
        qus = j.get("content")
        res = j.get("summary")
        if qus and res:
            data_temp["messages"][0]["content"] = qus
            data_temp["messages"][1]["content"] = res
            dev_data.append(copy.deepcopy(data_temp))
with open("./dev.jsonl", "w", encoding="UTF-8") as f:
    for l in dev_data:
        f.write(json.dumps(l, ensure_ascii=False) + '\n')

train_data = []
with open("./train.json", "r", encoding="UTF-8") as f:
    for d in f.readlines():
        j = json.loads(d)
        qus = j.get("content")
        res = j.get("summary")
        if qus and res:
            data_temp["messages"][0]["content"] = qus
            data_temp["messages"][1]["content"] = res
            train_data.append(copy.deepcopy(data_temp))
with open("./train.jsonl", "w", encoding="UTF-8") as f:
    for l in train_data:
        f.write(json.dumps(l, ensure_ascii=False) + '\n')
```

3. Lora微调

```shell
OMP_NUM_THREADS=1 torchrun --standalone --nnodes=1 --nproc_per_node=4 finetune.py data/AdvertiseGen/ ../../glm-4-9b-chat configs/lora.yaml
```

训练后的权重文件保存于`output/checkpoint-3000/`目录下。

4. 模型测试

```python
python inference.py output/checkpoint-3000/
```

```python
# inference.py 修改44行的messages
messages = [
    {
        "role": "system",
        "content": ""
    },
    {
        "role": "user",
        "content": "类型#裙*版型#显瘦*图案#线条*裙下摆#开叉*裙长#半身裙*裙款式#不规则"
    }
]
```

## ChatGLM3 量化

1. 环境配置

```shell
git clone --recursive https://github.com/li-plus/chatglm.cpp.git && cd chatglm.cpp
# 确保CUDA已安装
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
pip install tabulate tqdm transformers accelerate sentencepiece
```

2. `q8_0`量化

```shell
python3 chatglm_cpp/convert.py -i ../chatglm3-6b/ -t q8_0 -o chatglm-ggml-q8_0.bin
```

3. chatglm.cpp编译

```shell
cmake -B build -DGGML_CUBLAS=ON -DCUDA_ARCHITECTURES="86"	# 8.6对应3090
# 具体数值参考 https://developer.nvidia.com/cuda-gpus
cmake --build build -j --config Release
```

生成的可执行文件在`./build/bin/`目录下。

4. 运行模型

```shell
# 将"-p 你好"替换为"-i"即可进入交互模式
./build/bin/main -m chatglm-ggml-q8_0.bin --top_p 0.8 --temp 0.8 --sp examples/system/default.txt -p 你好
```

5. Python Binding

安装`chatglm-cpp`包：

```shell
CMAKE_ARGS="-DGGML_CUBLAS=ON" pip install .
rm -r chatglm_cpp	# 这个目录和导入包名冲突了
```

Python运行：

```shell
python examples/cli_demo.py -m chatglm-ggml-q8_0.bin -i
```

调用示例：

```python
import chatglm_cpp

pipeline = chatglm_cpp.Pipeline(
    "./chatglm-ggml-q8_0.bin"
)

with open("./examples/system/default.txt", "r", encoding="UTF-8") as f:
    system_prompt = f.read()

msg = pipeline.chat([
        chatglm_cpp.ChatMessage(role="system", content=system_prompt),
        chatglm_cpp.ChatMessage(role="user", content="你好")
    ],
    temperature=0.95,
    max_context_length=512,
    max_length=2048,
    top_p=0.7
)

print(msg.content)
```

