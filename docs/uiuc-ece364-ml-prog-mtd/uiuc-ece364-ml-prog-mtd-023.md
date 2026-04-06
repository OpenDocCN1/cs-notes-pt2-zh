# 023：Transformer 模型（第二部分）

![](img/60641c8cb790d0849145bcdfabb3f8e5_0.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_1.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_2.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_4.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_6.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_8.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_10.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_12.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_14.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_15.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_16.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_18.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_20.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_22.png)

在本节课中，我们将继续深入探讨 Transformer 模型。上一节我们介绍了 Transformer 的基本架构，本节中我们将看看一些更高级的使用方法，包括多层结构、特殊令牌以及不同的训练策略。

![](img/60641c8cb790d0849145bcdfabb3f8e5_24.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_26.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_28.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_29.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_31.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_33.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_34.png)

## 文本生成策略

![](img/60641c8cb790d0849145bcdfabb3f8e5_36.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_38.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_40.png)

上一节我们讨论了在推理阶段如何生成文本。我们从一个前缀（例如 “The meaning of life is”）开始，然后依赖模型来填充后续内容。模型会为词汇表中的每个可能的词元（token）输出一个概率分布。核心问题在于：我们如何根据这些概率选择下一个词元？

![](img/60641c8cb790d0849145bcdfabb3f8e5_42.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_44.png)

### 贪婪解码

![](img/60641c8cb790d0849145bcdfabb3f8e5_46.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_48.png)

最基本的方法是**贪婪解码**。在每个步骤中，我们只选择概率最高的词元。

![](img/60641c8cb790d0849145bcdfabb3f8e5_50.png)

**代码示例：贪婪解码逻辑**
```python
next_token = argmax(probability_distribution)
```

![](img/60641c8cb790d0849145bcdfabb3f8e5_52.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_54.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_56.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_58.png)

然而，贪婪解码存在一个问题：它只关注当前步骤的最优选择，可能导致最终生成的整个句子并不是全局概率最高的。因为一个在早期看起来概率稍低的词元，可能会引导模型走向一个整体概率更高的句子序列。

![](img/60641c8cb790d0849145bcdfabb3f8e5_60.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_62.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_64.png)

### 集束搜索

![](img/60641c8cb790d0849145bcdfabb3f8e5_66.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_68.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_69.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_70.png)

为了解决贪婪解码的局限性，我们引入**集束搜索**。其核心思想是，在每一步，我们保留概率最高的 `K` 条候选序列（称为“集束”），而不是只保留一条。

![](img/60641c8cb790d0849145bcdfabb3f8e5_72.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_73.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_75.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_77.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_78.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_79.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_80.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_81.png)

以下是集束搜索的步骤：
1.  **初始化**：从初始前缀开始，计算下一个词元的概率，保留概率最高的 `K` 个候选序列。
2.  **扩展**：对于当前的 `K` 个候选序列，分别计算各自下一个词元的概率，每个序列会生成 `K` 个新的扩展序列，总共得到 `K * K` 个候选。
3.  **筛选**：从这 `K * K` 个候选序列中，再次选择概率总和（累积对数概率）最高的 `K` 个序列，作为新的集束。
4.  **重复**：重复步骤 2 和 3，直到达到最大生成长度或遇到结束符。

![](img/60641c8cb790d0849145bcdfabb3f8e5_83.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_85.png)

**公式：序列概率**
序列 `S = (w1, w2, ..., wn)` 的概率通常用其对数概率的累积和来表示：
`log P(S) = log P(w1) + log P(w2|w1) + ... + log P(wn|w1, w2, ..., wn-1)`

![](img/60641c8cb790d0849145bcdfabb3f8e5_87.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_89.png)

集束搜索通过同时探索多条路径，比贪婪解码更有可能找到全局更优的句子。在 `transformers` 库中，可以通过设置 `num_beams` 参数大于 1 来启用集束搜索。

![](img/60641c8cb790d0849145bcdfabb3f8e5_91.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_93.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_94.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_95.png)

## 训练 Transformer 模型

现在，我们来看看如何训练 Transformer 模型。训练过程与我们之前训练其他模型类似。

![](img/60641c8cb790d0849145bcdfabb3f8e5_97.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_99.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_101.png)

### 标准训练流程

![](img/60641c8cb790d0849145bcdfabb3f8e5_103.png)

以编码器-解码器模型为例，假设我们的任务是问答。输入（编码器）是问题：“Mary had three apples, but ate one. How many does she have now?”，期望的输出（解码器）是答案：“Mary has 2 apples.”。

![](img/60641c8cb790d0849145bcdfabb3f8e5_105.png)

在训练时：
1.  我们将问题和答案（以起始符开头，如 “start a sentence”）分别输入编码器和解码器。
2.  解码器使用**掩码多头注意力**，确保在预测第 `t` 个词元时，只能看到前 `t-1` 个词元，防止信息泄露。
3.  模型为解码器输出的每个位置生成一个概率分布。
4.  我们使用**交叉熵损失**，将模型在每个位置输出的概率分布与真实的词元（例如 “Mary”, “has”, “2”, “apples”）进行比较。
5.  通过反向传播更新模型权重。

![](img/60641c8cb790d0849145bcdfabb3f8e5_107.png)

**重要说明**：这种训练方式要求模型输出与数据集中答案**完全一致**的文本。即使模型生成了语义正确但表述不同的答案（例如，“After calculation, she has two apples left.”），也会被标记为错误。这是当前基于精确匹配的评估方式的局限性。

## 特殊令牌

![](img/60641c8cb790d0849145bcdfabb3f8e5_109.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_110.png)

许多 Transformer 模型使用**特殊令牌**来赋予模型特定的能力或信息。

### CLS 令牌

![](img/60641c8cb790d0849145bcdfabb3f8e5_112.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_113.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_114.png)

**CLS** 令牌通常被添加在输入序列的开头。在像 BERT 这样的编码器模型中，这个令牌的最终隐藏状态被用作整个序列的聚合表示。

**用途**：对于句子分类任务（如情感分析），我们通常将 CLS 令牌对应的输出向量连接到一个线性分类层，而不是使用所有词元的输出。模型在训练中会学习让 CLS 令牌“吸收”整个句子的信息。

### SEP 令牌

**SEP** 令牌用作分隔符，用于分隔输入中的不同部分，例如两个句子、问题和上下文等。

不同的模型定义了不同的特殊令牌集（如 `[PAD]`, `[MASK]`, `[UNK]`）。在使用特定模型时，需要了解其分词器使用了哪些特殊令牌。

![](img/60641c8cb790d0849145bcdfabb3f8e5_116.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_118.png)

## 模型架构变体

Transformer 的编码器和解码器可以独立使用，形成不同的架构变体，适用于不同任务。

![](img/60641c8cb790d0849145bcdfabb3f8e5_120.png)

### 编码器-仅模型

![](img/60641c8cb790d0849145bcdfabb3f8e5_122.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_124.png)

例如 **BERT**。它只使用 Transformer 的编码器部分。
*   **特点**：**双向**。编码器的自注意力机制允许每个词元关注序列中的所有其他词元。
*   **用途**：擅长理解任务，如文本分类、命名实体识别、情感分析。它不直接用于生成连贯的文本序列。
*   **训练**：常使用**掩码语言建模**（随机遮盖输入中的一些词元，让模型预测它们）和**下一句预测**（判断两个句子是否连续）进行预训练。

![](img/60641c8cb790d0849145bcdfabb3f8e5_126.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_128.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_130.png)

### 解码器-仅模型

例如 **GPT 系列**。它只使用 Transformer 的解码器部分，并移除了编码器-解码器注意力层。
*   **特点**：**单向**。由于解码器使用了掩码自注意力，每个词元只能关注它之前的词元。
*   **用途**：擅长文本生成任务，如故事创作、代码生成、对话。
*   **训练**：通常使用**自回归语言建模**（给定前文，预测下一个词元）进行预训练。

### 编码器-解码器模型

![](img/60641c8cb790d0849145bcdfabb3f8e5_132.png)

例如 **T5**。它同时使用编码器和解码器。
*   **特点**：编码器处理输入，解码器生成输出，两者通过交叉注意力连接。
*   **用途**：适用于序列到序列的任务，如翻译、摘要、问答。

## 总结

![](img/60641c8cb790d0849145bcdfabb3f8e5_134.png)

![](img/60641c8cb790d0849145bcdfabb3f8e5_135.png)

本节课中我们一起学习了 Transformer 模型更深入的内容。我们探讨了改进文本生成的集束搜索算法，回顾了 Transformer 的标准训练流程及其在评估上的特点。我们还介绍了特殊令牌（如 CLS 和 SEP）的作用，以及 Transformer 的不同架构变体：适用于理解任务的编码器-仅模型（如 BERT）、适用于生成任务的解码器-仅模型（如 GPT）、和适用于序列转换的编码器-解码器模型。理解这些差异有助于为特定任务选择合适的模型。