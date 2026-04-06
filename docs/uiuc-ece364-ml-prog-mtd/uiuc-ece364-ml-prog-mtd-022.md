# 022：Transformer 模型（第一部分）

![](img/41aef183c5a0f59aac90280da97cb26b_1.png)

![](img/41aef183c5a0f59aac90280da97cb26b_3.png)

在本节课中，我们将学习 Transformer 模型的基础知识。我们将从理解为何需要超越循环神经网络（RNN）开始，然后深入探讨 Transformer 的核心概念，包括**分词**、**推理过程**以及模型的基本架构。我们将通过简单的代码示例来巩固理解，确保初学者也能跟上。

![](img/41aef183c5a0f59aac90280da97cb26b_5.png)

---

## 从 RNN 到 Transformer

上一节我们介绍了序列模型的基础。本节中，我们来看看为什么需要 Transformer。

![](img/41aef183c5a0f59aac90280da97cb26b_7.png)

RNN 存在几个主要问题：
1.  **速度慢**：RNN 必须**顺序处理数据**，无法充分利用并行计算。
2.  **梯度问题**：在长序列中容易出现**梯度消失**或**梯度爆炸**。
3.  **缺乏注意力机制**：RNN 在处理当前词时，对所有历史信息的关注是均等的，无法有效聚焦于最相关的上下文。

为了解决这些问题，Vaswani 等人在 Google Research 提出了 **Transformer 模型**。最初的 Transformer 是一个编码器-解码器模型，它是后续所有变体（如仅编码器、仅解码器模型）的基础。本节课我们将以此为基础进行探索。

![](img/41aef183c5a0f59aac90280da97cb26b_9.png)

![](img/41aef183c5a0f59aac90280da97cb26b_11.png)

---

![](img/41aef183c5a0f59aac90280da97cb26b_13.png)

## 分词：将文本转化为数字

模型只能处理数字，因此我们需要将文本（符号）转化为数字表示。在 RNN 中我们介绍过 **独热编码**，但当词汇表规模达到数十万甚至百万时，这种方法效率极低。

因此，我们需要 **分词**，即将文本拆分为有意义的子单元。以下是几种主要的分词方法：

![](img/41aef183c5a0f59aac90280da97cb26b_15.png)

*   **基于字符的分词**：将文本拆分为单个字符（包括空格）。每个字符分配一个数字。优点是词汇空间小，缺点是序列长，语义信息弱。
*   **基于词的分词**：将每个完整的词分配一个数字。优点是语义信息强，缺点是词汇空间巨大，且无法处理未登录词（如专有名词）。
*   **基于子词的分词**：一种折中方案。将词拆分为常见的子单元（如前缀、后缀、词根）。例如，“studying” 可能被拆分为 “stud” 和 “ying”。这能在压缩词汇空间和保留语义之间取得平衡。

让我们通过代码来看一个真实的分词示例。以下代码加载一个预训练的小型模型及其分词器。

![](img/41aef183c5a0f59aac90280da97cb26b_17.png)

```python
from transformers import AutoTokenizer, AutoModelForCausalLM

# 加载预训练模型和对应的分词器
model_name = "distilgpt2"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name)
model.eval()  # 设置为评估模式

![](img/41aef183c5a0f59aac90280da97cb26b_19.png)

![](img/41aef183c5a0f59aac90280da97cb26b_20.png)

# 分词示例
sample_text = "The quick brown fox"
tokens = tokenizer.tokenize(sample_text)
input_ids = tokenizer.encode(sample_text)
decoded_text = tokenizer.decode(input_ids)

![](img/41aef183c5a0f59aac90280da97cb26b_22.png)

print("Tokens:", tokens)
print("Input IDs:", input_ids)
print("Decoded Text:", decoded_text)
```

![](img/41aef183c5a0f59aac90280da97cb26b_24.png)

![](img/41aef183c5a0f59aac90280da97cb26b_26.png)

运行结果可能类似：
*   Tokens: `['The', 'Ġquick', 'Ġbrown', 'Ġfox']`
*   Input IDs: `[464, 2068, 7586, 21831]`
*   Decoded Text: `The quick brown fox`

![](img/41aef183c5a0f59aac90280da97cb26b_28.png)

这里需要注意：
*   分词结果不等于单词。例如，`Ġ` 是这个分词器用来表示前导空格的特殊符号。
*   每个模型都有其特定的分词方案和词汇表。

![](img/41aef183c5a0f59aac90280da97cb26b_30.png)

![](img/41aef183c5a0f59aac90280da97cb26b_31.png)

---

![](img/41aef183c5a0f59aac90280da97cb26b_33.png)

![](img/41aef183c5a0f59aac90280da97cb26b_35.png)

![](img/41aef183c5a0f59aac90280da97cb26b_36.png)

## 模型推理：文本是如何生成的

![](img/41aef183c5a0f59aac90280da97cb26b_38.png)

![](img/41aef183c5a0f59aac90280da97cb26b_40.png)

![](img/41aef183c5a0f59aac90280da97cb26b_42.png)

![](img/41aef183c5a0f59aac90280da97cb26b_43.png)

理解了如何将文本输入模型后，我们来看看模型如何生成新的文本，即**推理过程**。

当我们把一串输入 ID（如 `[464, 2068, 7586, 21831]`）输入模型后，模型会输出一个形状为 `(batch_size, sequence_length, vocab_size)` 的张量，称为 **logits**。

![](img/41aef183c5a0f59aac90280da97cb26b_45.png)

以我们的例子为例，输出形状可能是 `(1, 4, 50257)`。
*   `1`：批大小。
*   `4`：输入序列的长度。
*   `50257`：该模型的词汇表大小。

**这个 logits 张量的含义是关键**：对于输入序列中的**每一个位置**，模型都输出一个长度为 `vocab_size` 的向量。这个向量中的每个值，代表了**词汇表中每个词作为下一个词出现**的未归一化的可能性分数（logit）。

具体来说：
*   位置 0 的向量：预测在第一个词 “The” **之后**最可能出现的词。
*   位置 1 的向量：预测在 “The quick” **之后**最可能出现的词。
*   ...
*   位置 3 的向量（最后一个）：预测在 “The quick brown fox” **之后**最可能出现的词。**这正是我们生成下一个词所需要的**。

![](img/41aef183c5a0f59aac90280da97cb26b_47.png)

![](img/41aef183c5a0f59aac90280da97cb26b_49.png)

![](img/41aef183c5a0f59aac90280da97cb26b_51.png)

因此，生成文本的基本步骤（**贪婪解码**）如下：
1.  给定初始文本（如 “The meaning of life is”），将其分词并转化为输入 IDs。
2.  将输入 IDs 送入模型，获取最后一个位置的 logits 向量。
3.  对该 logits 向量应用 **softmax** 函数，得到概率分布。
4.  选择概率最高的那个词对应的 ID，作为下一个词。
5.  将这个新词添加到输入序列的末尾。
6.  重复步骤 2-5，直到生成长度达到预设值或遇到停止符。

以下是实现单步生成的代码示例：

![](img/41aef183c5a0f59aac90280da97cb26b_53.png)

![](img/41aef183c5a0f59aac90280da97cb26b_55.png)

![](img/41aef183c5a0f59aac90280da97cb26b_57.png)

```python
import torch

![](img/41aef183c5a0f59aac90280da97cb26b_59.png)

def generate_step(model, tokenizer, input_text, device='cpu'):
    # 将文本编码为输入 IDs
    input_ids = tokenizer.encode(input_text, return_tensors='pt').to(device)

    # 模型前向传播，获取 logits
    with torch.no_grad(): # 推理时不计算梯度
        outputs = model(input_ids)
        logits = outputs.logits

    # 获取最后一个位置的 logits
    next_token_logits = logits[0, -1, :] # 形状: (vocab_size,)

    # 应用 softmax 获取概率
    probs = torch.softmax(next_token_logits, dim=-1)

    # 获取概率最高的前 k 个 token
    top_k_probs, top_k_indices = torch.topk(probs, 10)

    # 解码并打印候选词
    print("Top candidates for next word:")
    for i, (idx, prob) in enumerate(zip(top_k_indices, top_k_probs)):
        token = tokenizer.decode(idx)
        print(f"  {i+1}. '{token}' (prob: {prob:.4f})")

    # 贪婪解码：选择概率最高的 token
    next_token_id = top_k_indices[0]
    next_token = tokenizer.decode(next_token_id)

    # 将新词添加到文本中
    new_text = input_text + next_token
    print(f"\nChosen next word: '{next_token}'")
    print(f"New text: '{new_text}'")
    return new_text

# 使用示例
current_text = "The meaning of life is"
for _ in range(3): # 生成3个词
    current_text = generate_step(model, tokenizer, current_text, device='cpu')
    print("-" * 30)
```

运行后，你可能会看到类似这样的输出：
```
Top candidates for next word:
  1. ' to' (prob: 0.3)
  2. ' not' (prob: 0.2)
  3. ' a' (prob: 0.15)
  ...
Chosen next word: ' to'
New text: 'The meaning of life is to'
------------------------------
Top candidates for next word:
  1. ' be' (prob: 0.4)
  2. ' live' (prob: 0.25)
  3. ' say' (prob: 0.1)
  ...
Chosen next word: ' be'
New text: 'The meaning of life is to be'
------------------------------
...
```

通过这种方式，模型就能逐步生成连贯的文本。早期的 GPT-2 模型可能因为没有明确的句子结束符而陷入循环。现代模型（如 GPT-3/4）会使用特殊的 **`<|endoftext|>`** 等标记来标识生成结束。

![](img/41aef183c5a0f59aac90280da97cb26b_61.png)

![](img/41aef183c5a0f59aac90280da97cb26b_62.png)

---

![](img/41aef183c5a0f59aac90280da97cb26b_63.png)

![](img/41aef183c5a0f59aac90280da97cb26b_64.png)

![](img/41aef183c5a0f59aac90280da97cb26b_65.png)

![](img/41aef183c5a0f59aac90280da97cb26b_66.png)

![](img/41aef183c5a0f59aac90280da97cb26b_67.png)

![](img/41aef183c5a0f59aac90280da97cb26b_68.png)

![](img/41aef183c5a0f59aac90280da97cb26b_69.png)

![](img/41aef183c5a0f59aac90280da97cb26b_70.png)

![](img/41aef183c5a0f59aac90280da97cb26b_71.png)

![](img/41aef183c5a0f59aac90280da97cb26b_72.png)

![](img/41aef183c5a0f59aac90280da97cb26b_73.png)

![](img/41aef183c5a0f59aac90280da97cb26b_74.png)

![](img/41aef183c5a0f59aac90280da97cb26b_75.png)

![](img/41aef183c5a0f59aac90280da97cb26b_76.png)

![](img/41aef183c5a0f59aac90280da97cb26b_77.png)

![](img/41aef183c5a0f59aac90280da97cb26b_78.png)

本节课中我们一起学习了 Transformer 的动机、文本的**分词**方法以及模型**推理**（文本生成）的基本原理。我们了解到，模型通过预测序列中每个位置的下一个词的概率分布，并通过**贪婪解码**等策略选择新词，从而自回归地生成文本。下一节课，我们将深入 Transformer 模型的核心——**注意力机制**的内部结构。