# 思维链

2022 **Chain-of-Thought Prompting Elicits Reasoning in Large Language Models**

通过生成一连串的思维链，即一系列中间推理步骤，显著提高LLM执行复杂推理的能力。

这种推理能力在规模足够大的语言模型中是自然出现的，只需通过在提示中提供几个思维链的示例。

2022 **Emergent Abilities of Large Language Models**

如果某一能力在规模较小的模型中不存在，但在较大的模型中存在，则该能力是新兴的。

这是一种量变引发的质变，是一种不可预测现象。

2023 **Large Language Models are Zero-Shot Reasoners**

在提示中提供几个CoT示例，可以认为是Few-Short形式的思维链。

而不添加示例，只是简答的添加“Let’s think step by step”在提示的最后，也能促进LLM在回答每个问题之前进行一步一步的思考过程，这是Zero-Shot形式的思维链。

2023 **Synthetic Prompting: Generating Chain-of-Thought Demonstrations for Large  Language Models**

CoT的提示质量取决于提供给LLM的演示，而手动创建大量演示的成本高昂。

因此可以利用一些手动制作的示例来促使LLM自己生成更多的示例，并选择有效的演示以引发更好的推理。

先是一个后向过程，促使LLM从种子示例中生成推理链，然后基于推理链生成复杂问题。

接着是前向过程，为后向过程中生成的问题合成最终的推理链，其不包含最终答案，因为答案可以通过执行生成的代码获得。

2023 **Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought  Reasoning by Large Language Models**

现有的CoT有三个缺陷：计算错误、漏步错误和语义误解。

为解决漏步错误，本文提出了PS提示。首先指定一个计划将整个任务分解为较小的子任务，然后根据计划执行这些子任务。

```txt
Let’s first understand the problem and devise a plan to solve the problem. Then, let’s carry out the plan and solve the problem step by step
```

为解决计算错误并提高生成推理步骤的质量，提出PS+提示的扩展变体。

```txt
extract relevant variables and their corresponding numerals
calculate intermediate results (pay attention to calculation and commonsense)
```

提取答案的提示则如下：

```txt
Therefore, the answer (arabic numerals) is
```

完整的提示参考：

```txt
Let’s first understand the problem, extract relevant variables and their corresponding numerals, and make a plan. Then, let’s carry out the plan, calculate intermediate variables (pay attention to correct numerical calculation and commonsense), solve the problem step by step, and show the answer.
```

2023 **Tree of Thoughts: Deliberate Problem Solving with Large Language Models**

LLM在推理过程中受限于token级别的、从左到右的决策过程。因此在需要探索、战略前瞻等的任务中表现不佳。

ToT对流行的CoT方法进行泛化，使得语言模型可以对其“想法”（解决问题的中间步骤）进行探索。

ToT将每个问题构建为在树上的搜索，其中每个节点是一个状态 $s=[x,z_{1...i}]$ ，包含输入和迄今为止思维序列的部分解。

2024 **Graph of Thoughts: Solving Elaborate Problems with Large Language Models**









