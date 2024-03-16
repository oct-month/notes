# Ollama

`Ollama`用于在本地运行开源LLM，如`Llama 2`。

Github：https://github.com/ollama/ollama

## 安装

### 脚本安装

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

### 手动安装

1. 下载可执行文件：

```shell
sudo curl -L https://ollama.com/download/ollama-linux-amd64 -o /usr/local/bin/ollama
sudo chmod a+x /usr/local/bin/ollama
```

2. 设置为系统服务：

添加ollama用户

```shell
sudo useradd -r -s /bin/false -m -d /usr/share/ollama ollama
```

编写服务文件：/etc/systemd/system/ollama.service

```ini
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3

[Install]
WantedBy=default.target
```

启用服务

```shell
sudo systemctl daemon-reload
sudo systemctl enable ollama
```

3. 驱动/软件安装

- **nvidia**

CUDA：https://developer.nvidia.com/cuda-downloads

- **amd**

amdgpu：https://www.amd.com/en/support/linux-drivers

ROCm v6：https://rocm.docs.amd.com/projects/install-on-linux/en/latest/tutorial/quick-start.html

4. 启动服务

```shell
sudo systemctl start ollama
```

### 更新方式

再次运行脚本：

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

或者直接更新ollama文件：

```shell
sudo curl -L https://ollama.com/download/ollama-linux-amd64 -o /usr/bin/ollama
sudo chmod +x /usr/bin/ollama
```

### 日志查看

```shell
journalctl -u ollama
```

### 卸载

```shell
sudo systemctl stop ollama
sudo systemctl disable ollama
sudo rm /etc/systemd/system/ollama.service

sudo rm $(which ollama)

sudo rm -r /usr/share/ollama  # 删除下载的模型
sudo userdel ollama
sudo groupdel ollama
```

### Docker

https://hub.docker.com/r/ollama/ollama

## 库

在Python/JS中运行Ollama，用于项目集成。

Python：https://github.com/ollama/ollama-python

JS：https://github.com/ollama/ollama-js

## 运行Llama 2

https://ollama.com/library/llama2

```shell
ollama run llama2:13b-chat   # 运行指定模型
```

支持的其他LLM：Gemma、Qwen等，完整列表见：https://ollama.com/library

## 定制模型

### 导入GGUF

1. `Modelfile`编写

```dockerfile
FROM ./vicuna-33b.Q4_0.gguf
TEMPLATE "[INST] {{ .Prompt }} [/INST]"
```

vicuna-33b.Q4_0.gguf是本地模型文件，33b应该是参数规模，Q4应该是4-bit量化。

TEMPLATE用于指定默认提示模板，有些chat模型需要。

2. 创建模型

```shell
ollama create example_model -f Modelfile
```

创建了一个名为example_model的模型。

3. 运行模型

```shell
ollama run example_model "这里可以放prompt"
```

### 导入PyTorch或Safetensors

https://github.com/ollama/ollama/blob/main/docs/import.md

### 通过Prompt定制

Modelfile的写法如下：

```dockerfile
FROM llama2

# set the temperature to 1 [higher is more creative, lower is more coherent]
PARAMETER temperature 1

# set the system message
SYSTEM """
You are Mario from Super Mario Bros. Answer as Mario, the assistant, only.
"""
```

## CLI命令

模型拉取/更新

```shell
ollama pull llama2
```

模型删除

```shell
ollama rm llama2
```

模型复制

```shell
ollama cp llama2 my-llama2
```

列出已pull的模型

```shell
ollama list
```

启动Ollama服务

```shell
ollama serve
```

## 模型输入

多行输入

```
>>> """Hello,
... world!
... """
I'm a basic program that prints the famous "Hello, world!" message to the console.
```

多模态输入

```
>>> What's in this image? /home/ubuntu/smile.png
The image features a yellow smiley face, which is likely the central focus of the picture.
```

参数传入（这是`shell`本身的功能）

```shell
$ ollama run llama2 "Summarize this file: $(cat README.md)"
 Ollama is a lightweight, extensible framework for building and running language models on the local machine. It provides a simple API for creating, running, and managing models, as well as a library of pre-built models that can be easily used in a variety of applications.
```

## API

Ollama有一个用于运行和管理模型的REST API。包括与模型聊天/文本生成的API。

详细文档见：https://github.com/ollama/ollama/blob/main/docs/api.md

