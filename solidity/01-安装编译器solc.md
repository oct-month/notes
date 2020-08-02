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

## 3、第一个 solidity 合约

Faucet.sol：

```sol
// Our first contract is a faucet
pragma solidity ^0.4.22;


contract Owned
{
    address owner;

    constructor () public
    {
        owner = msg.sender;     // 记录创建交易的账户
    }

    modifier onlyOwner          // 自定义修饰符
    {
        require(msg.sender == owner, "Only the contract owner can call this function!");
        _;                      // 被修饰函数代码的执行位置
    }

    function destroy() public onlyOwner
    {
        selfdestruct(owner);    // 合约自我销毁，并把余额转移到 owner 中
    }
}


contract Faucet is Owned        // 继承 Owned
{
    event Withdrawal(address indexed to, uint amount);  // 定义事件：记录所有提币操作
    event Deposit(address indexed from, uint amout);    // 定义事件：记录所有存入操作

    function with_draw(uint withdraw_amount) public // 用于提币的函数
    {
        require(withdraw_amount <= 0.1 ether, "you are too greedy!");
        require(address(this).balance >= withdraw_amount, "Faucet balance is insufficient!");
        msg.sender.transfer(withdraw_amount);       // 向合约调用者转账
        emit Withdrawal(msg.sender, withdraw_amount);   // 记录提币事件
    }

    function () public payable  // 接收转账的函数
    {
        emit Deposit(msg.sender, msg.value);        // 记录存入事件
    }
}
```

编译：

```sh
cnpm install solc@^0.4.22 -S
ls -s node_modules/.bin/solcjs ./solc
./solc --bin Faucet.sol
./solc --abi Faucet.sol
```

.vscode/settings.json

```json
{
    "solidity.compileUsingLocalVersion": "./solc"
}
```

Faucet_sol_Faucet.bin：

```bin
6080604052336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff1602179055506103d7806100536000396000f30060806040526004361061004c576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806383197ef01461009c5780639aefc5b9146100b3575b3373ffffffffffffffffffffffffffffffffffffffff167fe1fffcc4923d04b559f4d29a8bfc6cda04eb5b0d3c460751c2402c5c5cc9109c346040518082815260200191505060405180910390a2005b3480156100a857600080fd5b506100b16100e0565b005b3480156100bf57600080fd5b506100de60048036038101908080359060200190929190505050610204565b005b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff161415156101ca576040517f08c379a000000000000000000000000000000000000000000000000000000000815260040180806020018281038252602f8152602001807f4f6e6c792074686520636f6e7472616374206f776e65722063616e2063616c6c81526020017f20746869732066756e6374696f6e21000000000000000000000000000000000081525060400191505060405180910390fd5b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16ff5b67016345785d8a00008111151515610284576040517f08c379a00000000000000000000000000000000000000000000000000000000081526004018080602001828103825260138152602001807f796f752061726520746f6f20677265656479210000000000000000000000000081525060200191505060405180910390fd5b803073ffffffffffffffffffffffffffffffffffffffff163110151515610313576040517f08c379a000000000000000000000000000000000000000000000000000000000815260040180806020018281038252601f8152602001807f4661756365742062616c616e636520697320696e73756666696369656e74210081525060200191505060405180910390fd5b3373ffffffffffffffffffffffffffffffffffffffff166108fc829081150290604051600060405180830381858888f19350505050158015610359573d6000803e3d6000fd5b503373ffffffffffffffffffffffffffffffffffffffff167f7fcf532c15f0a6db0bd6d0e038bea71d30d808c7d98cb3bf7268a95bf5081b65826040518082815260200191505060405180910390a2505600a165627a7a723058203f5b2da1f741af287f7d5178775e4981ee19c5d1818fae01f9dff5dc496064220029
```

Faucet_sol_Faucet.abi：

```json
[
    {
        "constant": false,
        "inputs": [],
        "name": "destroy",
        "outputs": [],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
    },
    {
        "constant": false,
        "inputs": [
            {
                "name": "withdraw_amount",
                "type": "uint256"
            }
        ],
        "name": "with_draw",
        "outputs": [],
        "payable": false,
        "stateMutability": "nonpayable",
        "type": "function"
    },
    {
        "payable": true,
        "stateMutability": "payable",
        "type": "fallback"
    },
    {
        "anonymous": false,
        "inputs": [
            {
                "indexed": true,
                "name": "to",
                "type": "address"
            },
            {
                "indexed": false,
                "name": "amount",
                "type": "uint256"
            }
        ],
        "name": "Withdrawal",
        "type": "event"
    },
    {
        "anonymous": false,
        "inputs": [
            {
                "indexed": true,
                "name": "from",
                "type": "address"
            },
            {
                "indexed": false,
                "name": "amout",
                "type": "uint256"
            }
        ],
        "name": "Deposit",
        "type": "event"
    }
]
```

