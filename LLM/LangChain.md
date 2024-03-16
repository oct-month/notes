# LangChain

## 介绍

`LangChain`是一个用于开发“由语言模型驱动的应用程序”的框架。

它使得应用程序能：

- **上下文感知**
- **推理**

该框架的组成部分：

- **LangChain Libraries**：Python和JS库，包含大量组件的接口和集成。
- **LangChain Templates**：一组易于部署的参考架构。
- **LangServe**：一个用于将LangChain链部署为REST API的库。
- **LangSmith**：一个开发者平台，用于调试框架上的链。

![Diagram outlining the hierarchical organization of the LangChain framework, displaying the interconnected parts across multiple layers.](.img/langchain_stack.svg)

### LangChain Libraries

价值：

- **Components**：用于处理语言模型的工具和集成，使定制现有链和构建新链更容易。
- **Off-the-shelf chains**：现有链，使其容易上手。

组成部分：

- **langchain-core**：基本抽象和LangChain表达式语言LCEL
- **langchain-community**：第三方集成
- **langchain**：构成应用程序认知架构的链、代理和检索策略。

### LangChain Expression Language (LCEL)

一种组成链的声明式方式。

### Modules

LangChain为以下模块提供接口和集成。

- Model I/O：语言模型接口
- Retrieval：与应用程序特定数据的接口
- Agents：让模型选择使用给定的高级指令的工具

## 安装

基础安装，没有安装各类模型的集成依赖项：

```shell
pip install langchain
```

从源码安装：

```shell
pip install git+https://github.com/langchain-ai/langchain
```

langchain-community包含第三方集成，在langchain的依赖中，也可以独立安装：

```shell
pip install langchain-community
```

langchain-core包含基本抽象和LCEL，在langchain的依赖中，也可以独立安装：

```shell
pip install langchain-core
```

langsmith在langchain的依赖中，也可以独立安装：

```shell
pip install langsmith
```

langsmith注册地址：https://smith.langchain.com/，在settings中可以创建API KEY。使用langsmith时需要设置环境变量：

```shell
export LANGCHAIN_TRACING_V2="true"
export LANGCHAIN_API_KEY="..."      # 创建的KEY
```

langchain-experimental包含实验性的代码，安装方式：

```shell
pip install langchain-experimental
```

langserve帮助开发人员将LangChin runnables和chains部署为REST API，在langchain-cli的依赖中，也可以独立安装：

```shell
pip install "langserve[all]"
```

另外：

```shell
pip install "langserve[client]"  # 用于客户端代码
pip install "langserve[server]"  # 用于服务端代码
```

langchain-cli对于使用LangChain templates和其他LangServe项目非常有用，安装方式：

```shell
pip install langchain-cli
```

## 快速开始

前置条件：已设置并运行本地Ollama实例。并安装相关pip库。

```python
from langchain_community.llms import Ollama
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

llm = Ollama(model = "llama2:13b-chat")

# 简单使用
llm.invoke("How can langsmith help with testing?")

# 提示模板
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are world class technical documentation writer."),
    ("user", "{input}")
])

# 输出解析器
output_parser = StrOutputParser()

# 组装chain（从左到右）（LCEL）
chain = prompt | llm | output_parser

# 使用chain
chain.invoke({
    "input": "How can langsmith help with testi"
})
```

