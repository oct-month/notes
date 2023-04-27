# Kubernetes

Kubernetes 是用于自动部署，扩展和管理容器化应用程序的开源系统。详情可见 [官方介绍](https://kubernetes.io/zh/)。

## kubectl安装

Kubernetes 命令行工具，`kubectl`，使得你可以对 Kubernetes 集群运行命令。 你可以使用 `kubectl` 来部署应用、监测和管理集群资源以及查看日志。

### 使用包管理器安装

```sh
# 使用apt安装
sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2 curl
# tuna镜像中没有doc目录，所以从aliyun下载pat-key.gpg
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
echo "deb https://mirrors.tuna.tsinghua.edu.cn/kubernetes/apt kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

```sh
# 验证
kubectl version --client
```

### 直接下载可执行文件

下载最新的发行版本：

```sh
# linux
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

安装该可执行文件：

```sh
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

如果你并不拥有目标系统的 root 访问权限，你仍可以将 kubectl 安装到 `~/.local/bin` 目录下：

```sh
mkdir -p ~/.local/bin/kubectl
mv ./kubectl ~/.local/bin/kubectl
# 之后将 ~/.local/bin/kubectl 添加到环境变量 $PATH 中
```

**ps**：`windows`版本的下载地址为：https://dl.k8s.io/release/v1.20.0/bin/windows/amd64/kubectl.exe。下载后放到`PATH`环境变量中就行。

### 启用shell的自动补全功能

检查`bash-completion`是否安装：

```sh
type _init_completion
```

安装`bash-completion`：

```sh
sudo apt-get install bash-completion
```

将下面的内容添加到你的 `~/.bashrc` 文件中：

```sh
source /usr/share/bash-completion/bash_completion
```

之后，重新加载你的 Shell 并运行 `type _init_completion` 来检查 bash-completion 是否已正确安装。

如果遇到 ==source: not found==：

```sh
# 查看sh是dash还是bash
ls -l `which sh`
```

如果是`dash`则需要改成`bash`：

```sh
sudo dpkg-reconfigure dash
# 在界面中选择no
```

启用`kubectl`自动补全：

```sh
# 需要root
kubectl completion bash >/etc/bash_completion.d/kubectl
```

> bash-completion 会自动源引 `/etc/bash_completion.d` 下的所有自动补齐脚本。
>
> 重新加载 Shell 之后，kubectl 的自动补齐应该能够使用了。

## minikube安装

[`minikube`](https://minikube.sigs.k8s.io/) 是一个工具， 能让你在本地运行 Kubernetes。 `minikube` 在你本地的个人计算机（包括 Windows、macOS 和 Linux PC）运行一个单节点的 Kubernetes 集群，以便你来尝试 Kubernetes 或者开展每天的开发工作。

Github下载：https://github.com/kubernetes/minikube/releases。

tuna镜像下载：https://mirrors.tuna.tsinghua.edu.cn/github-release/kubernetes/minikube/LatestRelease/。

阿里云下载：https://developer.aliyun.com/article/221687

预装环境：

`minikube`需要容器或者虚拟机管理器，所以需要提前安装`docker`、`virtualbox`、`vmware`、`Podman`、`kvm`中的任意一个。

下载并安装最新的发行版：

```sh
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
# sudo dpkg -i minikube_latest_amd64.deb
sudo apt-get install ./minikube_latest_amd64.deb
```

启动集群：

```sh
# 指定采用docker驱动
minikube start --driver=docker --image-mirror-country=cn
```

停止集群：

```sh
minikube stop
```

删除所有`minikube`集群：

```sh
minikube delete --all
```

