# Vyper 初次使用

## 1、pip 安装 vyper

安装命令：

```sh
pip3 install vyper
```

编译命令：

产生字节码：

```sh
vyper test.vy
```

产生 abi ：

```sh
vyper -f json test.vy
```

## 2、第一个 Vyper 合约

test.vy：

```python
# 声明一个名为theBool的变量
theBool: public(bool)

@internal
def topFunction() -> bool:
    self.theBool = True
    return self.theBool

@external
def lowerFunction():
    assert self.topFunction()
```

字节码：

```bin
0x61012a56600436101561000d57610120565b600035601c52740100000000000000000000000000000000000000006020526f7fffffffffffffffffffffffffffffff6040527fffffffffffffffffffffffffffffffff8000000000000000000000000000000060605274012a05f1fffffffffffffffffffffffffdabf41c006080527ffffffffffffffffffffffffed5fa0e000000000000000000000000000000000060a0526000156100c3575b610140526001600055600054600052600051610140515650005b6321047fec60005114156100f85734156100dc57600080fd5b600658016100a9565b61014052610140516100f657600080fd5b005b6381853307600051141561011f57341561011157600080fd5b60005460005260206000f350005b5b60006000fd5b61000461012a0361000460003961000461012a036000f3
```

abi：

```json
[
    {
        "name": "lowerFunction",
        "outputs": [],
        "inputs": [],
        "stateMutability": "nonpayable",
        "type": "function",
        "gas": 36604
    },
    {
        "name": "theBool",
        "outputs": [
            {
                "type": "bool",
                "name": ""
            }
        ],
        "inputs": [],
        "stateMutability": "view",
        "type": "function",
        "gas": 1211
    }
]
```

