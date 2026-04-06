# 004：线性代数（第二部分）🚀

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_1.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_3.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_5.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_7.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_9.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_11.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_13.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_15.png)

在本节课中，我们将深入学习线性代数的更多核心概念，包括张量切片、广播、线性方程组求解以及矩阵微积分的入门知识。这些内容是理解后续自动微分和神经网络训练的基础。

## 张量切片操作回顾与深化 🔪

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_17.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_19.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_21.png)

上一节我们介绍了矩阵的基本操作。本节中，我们来看看更复杂的张量切片操作。切片是选择和操作张量特定部分的重要技术。

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_23.png)

假设我们有以下张量：
```python
tensor = torch.tensor([[[1, 2, 3, 4, 5, 6, 7, 8],
                        [9, 10, 11, 12, 13, 14, 15, 16]],
                       [[17, 18, 19, 20, 21, 22, 23, 24],
                        [25, 26, 27, 28, 29, 30, 31, 32]]])
```
该张量的形状是 `(2, 2, 8)`。

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_25.png)

以下是几个切片操作的例子：

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_27.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_29.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_31.png)

*   **获取第一行中的奇数索引元素**：`tensor[0, 0, ::2]` 或 `tensor[0, 0, :8:2]`。结果形状为 `(4,)`。
*   **获取每个矩阵的第二行**：`tensor[:, 1, :]`。结果形状为 `(2, 8)`。注意，当使用标量索引时（如 `1`），该维度会被压缩（消除）。
*   **获取每个矩阵中每行的第三个元素**：`tensor[:, :, 2]`。结果形状为 `(2, 2)`。

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_33.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_35.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_37.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_39.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_41.png)

理解切片时维度的压缩规则至关重要：使用冒号 `:` 切片会保留该维度，而使用标量索引则会消除该维度。

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_43.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_45.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_46.png)

## 广播机制详解 📡

切片让我们能操作数据的子集，而广播则允许我们对形状不同的张量进行运算。广播是NumPy和PyTorch中的一项机制，它会在执行按元素操作时，自动扩展较小张量的维度以匹配较大张量的形状。

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_48.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_50.png)

广播遵循以下规则：从最右边的维度开始，逐元素比较两个张量的形状。如果两个维度相等，或其中一个为1，则它们是“兼容的”。兼容的维度中，大小为1的维度会被“拉伸”（复制）以匹配另一个张量的大小。如果所有维度都兼容，则可以进行运算。

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_52.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_54.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_56.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_58.png)

以下是广播的例子：

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_60.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_62.png)

*   **标量与向量相加**：`torch.tensor([1, 2, 3]) + 1`。标量 `1` 被广播为 `[1, 1, 1]`。
*   **矩阵与行向量相加**：一个形状为 `(4, 3)` 的矩阵与一个形状为 `(3,)` 的行向量相加。行向量被广播为 `(4, 3)` 的形状。
*   **不兼容的形状**：尝试将形状为 `(2, 1)` 的张量与形状为 `(4, 3)` 的张量相加会引发错误，因为第二维（1 vs 3）不兼容。

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_64.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_66.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_68.png)

广播功能强大，可以简化代码，例如高效地生成多维坐标网格。但也需谨慎使用，因为意外的广播可能导致难以调试的错误。

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_70.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_72.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_74.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_76.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_78.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_79.png)

## 求解线性方程组 ➗

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_81.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_82.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_84.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_85.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_87.png)

在机器学习中，我们经常需要求解线性方程组。一个包含 *m* 个方程和 *n* 个未知数的系统可以简洁地表示为矩阵方程：
**A X = B**
其中 **A** 是 *m × n* 的系数矩阵，**X** 是 *n × 1* 的未知数向量，**B** 是 *m × 1* 的结果向量。

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_89.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_91.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_93.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_95.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_97.png)

如果 **A** 是方阵且可逆，则解为：
**X = A⁻¹ B**

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_99.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_101.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_103.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_105.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_106.png)

在PyTorch中，我们可以直接计算：
```python
import torch
A = torch.tensor([[2., 1., 1.], [0., 3., 2.], [0., 0., 1.]])
B = torch.tensor([4., 5., 6.])
X = torch.inverse(A) @ B  # 或者 torch.matmul(torch.inverse(A), B)
```
更优的方法是使用专用求解器 `torch.linalg.solve`，它在数值稳定性和计算性能上通常优于直接求逆：
```python
X = torch.linalg.solve(A, B)
```
专用求解器使用了如LU分解等更高效、更稳定的算法。

## 矩阵微积分入门 📐

为了对神经网络进行梯度下降训练，我们需要计算损失函数对众多参数的导数。这涉及到多变量函数的微分，即矩阵微积分。

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_108.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_110.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_112.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_114.png)

对于一个多变量函数 *f(x₁, x₂, ..., xₙ)*，其梯度是所有偏导数组成的向量：
**∇f = [∂f/∂x₁, ∂f/∂x₂, ..., ∂f/∂xₙ]**

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_116.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_117.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_119.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_120.png)

当有多个函数 *f₁, f₂, ..., fₘ* 时，所有偏导数可以组织成一个 *m × n* 的矩阵，称为**雅可比矩阵 (Jacobian Matrix)**：
**J = [ [∂f₁/∂x₁, ∂f₁/∂x₂, ..., ∂f₁/∂xₙ],
        [∂f₂/∂x₁, ∂f₂/∂x₂, ..., ∂f₂/∂xₙ],
        ...,
        [∂fₘ/∂x₁, ∂fₘ/∂x₂, ..., ∂fₘ/∂xₙ] ]**

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_122.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_123.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_125.png)

在PyTorch中，我们可以使用自动微分来计算雅可比矩阵在特定输入点的值：
```python
def F(x):
    # 假设有两个函数
    f1 = 3 * x[0]**2 * x[1]
    f2 = 2 * x[0] + x[1]**8
    return torch.stack([f1, f2])

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_127.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_129.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_131.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_133.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_135.png)

x = torch.tensor([1., 1.], requires_grad=True)
y = F(x)
jacobian = torch.autograd.functional.jacobian(F, x)
```

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_136.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_138.png)

最后，记住两个重要的矩阵导数公式（采用分子布局）：
1.  对于 **Ax**，关于 **x** 的导数是 **A**。
    **∂(Ax)/∂x = A**
2.  对于二次型 **xᵀx**，关于 **x** 的导数是 **2x**。
    **∂(xᵀx)/∂x = 2x**

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_140.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_142.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_144.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_146.png)

## 总结 🎯

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_148.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_149.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_150.png)

![](img/2ba0ec5db4d39a31adeeb56dd3be867a_151.png)

本节课中我们一起学习了线性代数的几个关键进阶主题。我们深化了对张量切片和广播机制的理解，这是高效操作数据的基础。我们掌握了使用PyTorch求解线性方程组的方法，并了解了专用求解器的优势。最后，我们开启了矩阵微积分的大门，引入了梯度和雅可比矩阵的概念，这是下一周学习自动微分和反向传播的核心预备知识。请务必理解这些基础概念，它们将贯穿我们整个机器学习之旅。