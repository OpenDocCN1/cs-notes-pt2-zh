# 010：逻辑回归与分类 📊

![](img/129dc44a9f4ea78dcd103d34f0f6b798_0.png)

在本节课中，我们将要学习分类问题，特别是逻辑回归和支持向量机。我们将从线性分类器开始，探讨其局限性，然后介绍更强大的逻辑回归和支持向量机模型。课程内容将涵盖核心概念、数学公式以及代码实现，确保初学者能够理解。

---

![](img/129dc44a9f4ea78dcd103d34f0f6b798_2.png)

## 线性分类器 🧮

上一节我们介绍了线性回归，本节中我们来看看如何将其用于分类。分类的目标是预测一个离散的目标值。在本课程中，我们主要关注二分类问题。

我们定义一个线性函数：
\[
z = w^T x
\]
其中，\( x \) 是输入向量（包含一个恒为1的偏置项 \( x_0 \)），\( w \) 是权重向量。然后，我们在这个线性模型之上添加一个分类层：
\[
y = \begin{cases}
1 & \text{if } z \ge 0 \\
0 & \text{if } z < 0
\end{cases}
\]
这本质上是一个符号函数（sign function）。

### 线性分类器的局限性

![](img/129dc44a9f4ea78dcd103d34f0f6b798_4.png)

并非所有函数都可以用线性分类器完美分离。例如，异或（XOR）函数就不是线性可分的。这意味着我们无法找到一条直线将两类点完全分开。

然而，我们可以使用特征映射（feature mapping）的技巧。例如，对于XOR问题，我们可以将特征从 \( [x_1, x_2] \) 映射到 \( [1, x_1, x_2, x_1 x_2] \)，从而在新的特征空间中使其线性可分。

---

![](img/129dc44a9f4ea78dcd103d34f0f6b798_6.png)

## 支持向量机（SVM）⚔️

线性分类器的一个主要问题是，所有数据点对决策边界的影响是均等的，这可能导致边界被远离的异常点过度影响。支持向量机通过引入“间隔”（margin）的概念来解决这个问题。

![](img/129dc44a9f4ea78dcd103d34f0f6b798_8.png)

SVM的目标是找到一个决策边界，使得两类数据点之间的间隔最大化。我们不仅希望数据点被正确分类，还希望它们尽可能远离决策边界。

### SVM的数学表述

对于正样本（标签 \( t = 1 \)），我们希望 \( w^T x \ge 1 \)。
对于负样本（标签 \( t = -1 \)），我们希望 \( w^T x \le -1 \)。
我们可以将这两个条件合并为一个不等式：
\[
t \cdot (w^T x) \ge 1
\]
或者等价地：
\[
1 - t \cdot (w^T x) \le 0
\]

SVM的损失函数旨在惩罚那些违反上述不等式的点（即被误分类或落在间隔内的点），同时鼓励间隔最大化。一个常见的损失函数形式是合页损失（Hinge Loss）：
\[
L = \frac{1}{N} \sum_{i=1}^{N} \max(0, 1 - t_i \cdot (w^T x_i)) + \lambda ||w||^2
\]
其中，\( \lambda \) 是一个控制正则化强度的超参数。

以下是使用PyTorch实现SVM的一个简化示例框架：
```python
import torch
import torch.nn as nn

class LinearSVM(nn.Module):
    def __init__(self, input_dim):
        super(LinearSVM, self).__init__()
        self.linear = nn.Linear(input_dim, 1)

    def forward(self, x):
        return self.linear(x)

# 定义合页损失
def hinge_loss(output, target):
    loss = torch.mean(torch.clamp(1 - target * output, min=0))
    return loss
```

---

![](img/129dc44a9f4ea78dcd103d34f0f6b798_10.png)

## 逻辑回归 🔮

当数据不是线性可分，或者我们想要得到属于某个类别的概率时，逻辑回归是一个更好的选择。逻辑回归在线性模型的基础上使用了Sigmoid函数。

![](img/129dc44a9f4ea78dcd103d34f0f6b798_12.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_14.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_16.png)

Sigmoid函数定义为：
\[
\sigma(z) = \frac{1}{1 + e^{-z}}
\]
其中，\( z = w^T x \)。Sigmoid函数将任何实数 \( z \) 映射到 \( (0, 1) \) 区间，可以解释为概率。

![](img/129dc44a9f4ea78dcd103d34f0f6b798_17.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_18.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_19.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_20.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_22.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_23.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_25.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_27.png)

### 逻辑回归的推导

![](img/129dc44a9f4ea78dcd103d34f0f6b798_29.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_30.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_32.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_33.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_35.png)

对于单个数据点 \( (x, t) \)，其中 \( t \in \{0, 1\} \)：
*   如果 \( t = 1 \)，我们希望 \( \sigma(w^T x) \) 尽可能大（接近1）。
*   如果 \( t = 0 \)，我们希望 \( \sigma(w^T x) \) 尽可能小（接近0），即 \( 1 - \sigma(w^T x) \) 尽可能大。

![](img/129dc44a9f4ea78dcd103d34f0f6b798_37.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_39.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_40.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_42.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_44.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_45.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_46.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_48.png)

这可以统一表示为最大化 \( \sigma(w^T x)^t \cdot (1 - \sigma(w^T x))^{1-t} \)。对于整个数据集，我们最大化所有数据点概率的乘积（似然函数）。为了便于计算，我们通常取负对数似然（Negative Log-Likelihood）作为损失函数进行最小化：
\[
L(w) = -\sum_{i=1}^{N} \left[ t_i \log(\sigma(w^T x_i)) + (1 - t_i) \log(1 - \sigma(w^T x_i)) \right]
\]
这也就是二元交叉熵损失（Binary Cross-Entropy Loss）。

![](img/129dc44a9f4ea78dcd103d34f0f6b798_50.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_52.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_54.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_55.png)

以下是逻辑回归的PyTorch实现示例：
```python
import torch
import torch.nn as nn

![](img/129dc44a9f4ea78dcd103d34f0f6b798_57.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_59.png)

class LogisticRegression(nn.Module):
    def __init__(self, input_dim):
        super(LogisticRegression, self).__init__()
        self.linear = nn.Linear(input_dim, 1)

    def forward(self, x):
        z = self.linear(x)
        # 使用sigmoid激活函数
        return torch.sigmoid(z)

# 模型、损失函数和优化器
model = LogisticRegression(input_dim=1)
criterion = nn.BCELoss()  # 二元交叉熵损失
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)
```

---

![](img/129dc44a9f4ea78dcd103d34f0f6b798_61.png)

## 总结 🎯

本节课中我们一起学习了分类问题的基本方法。

![](img/129dc44a9f4ea78dcd103d34f0f6b798_63.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_65.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_66.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_67.png)

1.  **线性分类器**：我们在线性模型上添加了一个阈值函数来进行二分类，但它只能处理线性可分的数据。
2.  **支持向量机（SVM）**：通过引入“间隔”的概念，SVM专注于那些靠近决策边界的数据点，从而得到一个更鲁棒的分类器，并能通过核技巧处理非线性问题。
3.  **逻辑回归**：它使用Sigmoid函数将线性输出转化为概率，适用于需要概率输出或数据非线性程度不高的情况。其损失函数是交叉熵损失。

![](img/129dc44a9f4ea78dcd103d34f0f6b798_69.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_71.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_73.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_75.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_77.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_79.png)

![](img/129dc44a9f4ea78dcd103d34f0f6b798_80.png)

每种方法都有其适用场景：线性分类器简单快速；SVM对于寻找清晰间隔的边界非常有效；而逻辑回归则是输出概率和作为更复杂神经网络基础组件的理想选择。在接下来的课程中，我们将继续深入探讨分类的更多高级主题。