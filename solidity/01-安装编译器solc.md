# 安装 solc 编译器

## 1、npm / Node.js

安装命令：

```sh
cnpm install -g solc
```

编译命令：

```sh
solcjs --bin test.sol
```

## 2、ppa

安装命令：

```sh
sudo add-apt-repository ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install solc
```

编译命令：

这个命令会生成 .bin 文件，里面有16进制的字节码。

```sh
solc --bin test.sol
```

这个命令不生成文件，直接在控制台输出16进制的字节码

```sh
solc --optimize --bin test.sol
```

这个命令不生成文件，直接在控制台输出 JSON 格式的合约 ABI

```sh
solc --abi test.sol
```

