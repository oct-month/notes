# 安装 solc 编译器

## 1、npm / Node.js

安装命令：

```sh
cnpm install -g solc
```

编译命令：

```sh
solcjs --optimize --bin test.sol
```

## 2、ppa

安装命令：

```sh
sudo add-apt-repository ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install solc
```

编译命令：

```sh
solc --optimize --bin test.sol
```



