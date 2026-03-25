# 024：模拟、模型与实验验证 🔬

在本节课中，我们将探讨计算机模拟、理论模型与物理现实之间的复杂关系。我们将通过具体的例子，学习如何构建和评估模型，并理解统计结论与真实情况之间的区别。

---

上一节我们讨论了模拟结果的统计可信度。本节中，我们来看看一个关键问题：即使统计结果看起来很好，我们如何确保模拟模型本身是正确的？

我以一个“谎言”结束上节课。我声称，当投掷足够多的针并进行足够多的试验后，我们可以通过观察标准差来判断结果的准确性，并以95%的置信度给出答案的范围。但这并不完全正确。

我混淆了统计上可靠的结论与绝对真理。任何统计检验的有效性都依赖于某些假设，例如数据的独立性。但最关键的一个假设是：**我们的模拟确实是现实的一个准确模型**。

回想一下设计模拟的过程：我们研究了布丰投针问题的数学原理，进行了一些代数推导，并据此编写了代码、运行了模拟、分析了统计结果。但假设我在编码时犯了一个错误。例如，在代数推导出的公式中，我错误地输入了数字 `2` 而不是正确的值。

现在，如果我们运行这个有错误的模拟，它可能仍然会快速收敛，给出一个很小的标准差，让我非常自信地认为π的值大约是1.569。然而，我们都知道π的真实值远非如此。问题不在于我的统计方法，而在于我的统计结果是关于**模拟本身**的，而不是关于π的。

这里的教训是：**在相信任何模拟结果之前，我们必须确信我们的概念模型是正确的，并且我们已经正确地实现了这个模型。**

那么，我们该如何做呢？一种方法是根据现实来检验我们的结果。如果我运行模拟后得出π约为1.57，我可以去画一个圆并粗略测量其周长，从而立即知道我的答案远非正确。这正是科学家们应该做的：当他们使用模拟模型推导出某个结果时，总会进行一些实验来验证其推导结果至少是合理的。统计检验有助于确保我们在细节上处理得当，但首先必须进行合理性检查。

这是一个非常重要的教训：**不要被统计检验所迷惑，并将其与真理混为一谈。**

---

现在，我想继续前进，看看更多与我们一直在做的事情类似的例子。实际上，我们将要探讨的是**物理现实、理论模型和计算模型**三者之间的相互作用。这正是现代科学与工程的运作方式。

我们从一个物理情境开始。这里的“物理”不一定指砖瓦或物理学、生物学，它可以是股票市场，即世界上的某种真实情境。我们使用一些理论来获得对该情境的洞察。当理论变得过于复杂或无法直接给出答案时，我们就使用计算。

现在，我想谈谈这三者是如何相互关联的。想象一下，你是一个聪明的高中生，正在学习生物、化学或物理。你可能都经历过这种情况：你尽最大努力完成了一个实验，但通过计算，你发现实验结果与理论并不完全吻合。

你应该怎么做？嗯，我猜你们都遇到过这种情况。你可以直接提交结果，但可能因实验技术不佳而受到批评。更可能的是，你计算出了“正确”的结果并提交了它们，但这又可能因“过于完美”而引人怀疑。但作为聪明人，我猜你们大多数人在高中时都这样做过：计算出正确结果，看看自己的实验结果，然后在两者之间取一个折中值，引入一点误差，但又不显得太愚蠢。

要正确处理这个问题，你需要有一种方法来建模，不仅是现实，还包括实验误差。通常，建模实验误差的最佳方法（即使我们并非有意作弊）是假设实际数据存在某种随机扰动。事实上，高斯的一大贡献就是指出，我们通常可以将实验误差建模为**正态分布**，即高斯分布。

---

让我们看一个例子：考虑一个弹簧。不是指季节或泉水，而是你在物理课上学到的那种弹簧，施加力可以压缩或拉伸它。弹簧非常有用，我们用在汽车、床垫、安全带以及发射抛射物上。事实上，我们稍后会看到，它们在生物学中也经常出现。

1676年，英国物理学家罗伯特·胡克提出了**胡克定律**来解释弹簧的行为。定律非常简单：

**F = -Kx**

换句话说，储存在弹簧中的力 **F** 与弹簧被压缩或拉伸的距离 **x** 成线性关系。这就是胡克定律。该定律适用于多种材料和系统，包括许多生物系统。当然，它不适用于任意大的力。所有弹簧都有一个**弹性极限**，如果拉伸超过这个极限，定律就会失效。

这里的比例常数 **K** 被称为**弹簧常数**。每个弹簧都有一个常数 **K** 来解释其行为。如果弹簧很硬（如汽车悬架），K 就大；如果弹簧不硬（如圆珠笔中的弹簧），K 就小。负号表示弹簧施加的力与位移方向相反。

知道弹簧的弹簧常数具有重要的实际意义，可用于校准秤、原子力显微镜等。事实上，最近人们开始考虑将DNA建模为弹簧，而找到DNA的弹簧常数在一些生物实验中非常有用。

一代又一代的学生通过一个非常简单的实验来估算弹簧常数。你很可能也做过：取一个弹簧，将其悬挂在某种装置上，然后在弹簧底部挂上一个已知质量的砝码，测量弹簧被拉伸的长度。

根据 **F = -Kx**，以及 **F = m * a**（质量乘以加速度），并且我们知道在地球上重力加速度约为 **9.81 米/秒²**，我们就可以通过代数计算 **K**。

如果我们没有实验误差，这一切都很好。但我们确实有误差。因此，人们通常不是只挂一个砝码，而是挂上**不同质量**的砝码，等待弹簧停止运动后进行测量。这样他们就得到了一系列数据点。他们假设误差是正态分布的，有些为正，有些为负。如果进行足够多的实验，误差会相互抵消，我们就能很好地估算出弹簧常数 **K**。

我做了这样一个实验，并将结果存入一个文件。文件的第一行说明了内容（距离以米为单位，质量以千克为单位），然后数据由空格分隔。

以下是读取数据的代码。通常，输入/输出操作应放在单独的函数中，这样如果数据格式改变，只需修改这部分代码。

```python
def get_data(file_name):
    data_file = open(file_name, 'r')
    distances = []
    masses = []
    data_file.readline() # 丢弃标题行
    for line in data_file:
        d, m = line.split()
        distances.append(float(d))
        masses.append(float(m))
    data_file.close()
    return (distances, masses)
```

然后我绘制数据。这里我们使用了一种新类型：**数组**。数组由 `PyLab`（基于 `NumPy`）提供，类似于列表，但支持逐点运算，例如将数组乘以一个数会对每个元素进行乘法。

```python
distances, masses = get_data('spring_data.txt')
distances = pylab.array(distances)
masses = pylab.array(masses)
forces = masses * 9.81
pylab.plot(forces, distances, 'bo')
pylab.xlabel('Force (Newtons)')
pylab.ylabel('Distance (meters)')
```

现在，我可以计算 **K**。但在计算之前，我想看看数据是否合理。我们有一个理论模型：数据应该大致落在一条直线上（考虑到实验误差）。我需要找到那条线，因为知道了线，我就能计算 **K**。**K** 与那条线的斜率成反比。

如何得到那条线？我要找到一条最接近所有数据点的线，即**最佳拟合**线。对于超过两个点的情况，我需要一个**目标函数**来衡量拟合的好坏，以便比较和选择最佳拟合。

一个标准且常用的度量是**最小二乘拟合**。其目标函数是：

**Σ (observed_i - predicted_i)²**

我们希望对所有数据点 i 求和。通过平方差值，我们忽略了点在线上方还是下方，只关心距离线的远近。总和越小，拟合越好。

如何找到最佳拟合？有几种方法，在某些条件下有解析解。好消息是，`PyLab` 内置了此功能。函数是 `polyfit`。

`polyfit` 接受三个参数：所有观测到的 x 值、所有观测到的 y 值，以及多项式的**次数**。它可以拟合任意次数的多项式（直线、抛物线、三次曲线等）。对于直线（一次多项式），它返回系数 **A** 和 **B**，其中直线方程为 **y = Ax + B**。

```python
def fit_data(file_name):
    distances, masses = get_data(file_name)
    distances = pylab.array(distances)
    masses = pylab.array(masses)
    forces = masses * 9.81
    pylab.plot(forces, distances, 'ko', label='Measured points')
    # 拟合直线 (degree=1)
    A, B = pylab.polyfit(forces, distances, 1)
    predicted_distances = A * forces + B
    pylab.plot(forces, predicted_distances, label='Linear fit')
    # 计算弹簧常数 K (K = 1/A)
    K = 1.0 / A
    pylab.title('K = ' + str(round(K, 5)) + ' N/m')
    pylab.legend(loc='best')
```

运行后，我们得到一条拟合直线，并计算出 **K** 约为 21。但我们应该满意吗？从图上来看，这些点离直线相当远。如果拟合不好，我就必须怀疑从这个拟合模型推导出的 **K** 值。

让我们尝试拟合一个三次曲线（`degree=3`）。

```python
    # 拟合三次曲线 (degree=3)
    coeffs = pylab.polyfit(forces, distances, 3)
    # coeffs 包含 [a, b, c, d]，对应 ax^3 + bx^2 + cx + d
    predicted_cubic = pylab.polyval(coeffs, forces)
    pylab.plot(forces, predicted_cubic, 'r:', label='Cubic fit')
```

视觉上，三次曲线似乎是比直线好得多的数据描述。但是，我们应该为此高兴吗？我们构建模型的目的是为了更好地理解弹簧。我们经常用模型来**预测**我们无法在实验中测量的值。

假设我使用这个三次模型来预测挂上一个1.5公斤重物时弹簧的拉伸。模型预测弹簧不仅会停止拉伸，甚至会收缩到比原始位置还短。这在物理世界中极不可能发生。

这表明，虽然我可以轻松地用一条曲线拟合数据，并且拟合得很好，但它可能具有很差的**预测价值**。我之所以开始这项研究，是基于一个关于弹簧的理论（胡克定律），它应该是一个**线性模型**。仅仅因为我的数据可能不符合该理论，并不意味着我应该随意拟合一条曲线。

如果我们愿意使用足够高次数的多项式，几乎可以完美拟合任何数据，但这证明不了什么，也不实用。因此，让我们暂时忽略曲线，看看原始数据。数据在末端似乎变平了。这违反了胡克定律（线性关系）。我是否违反了胡克定律？还是我做错了什么？

胡克定律只适用于**弹性极限内**。很可能我在实验中超过了弹簧的弹性极限。如果这是真的，我就有理论依据可以**丢弃最后那几个变平的数据点**。

让我们回到代码，丢弃最后六个点（假设它们超出了弹性极限），然后重新进行线性拟合。

```python
    # 丢弃最后六个数据点（假设超出弹性极限）
    forces_trimmed = forces[:-6]
    distances_trimmed = distances[:-6]
    A_trimmed, B_trimmed = pylab.polyfit(forces_trimmed, distances_trimmed, 1)
    predicted_trimmed = A_trimmed * forces_trimmed + B_trimmed
    pylab.plot(forces_trimmed, predicted_trimmed, 'g--', label='Trimmed linear fit')
    K_trimmed = 1.0 / A_trimmed
```

现在，我们得到了视觉上更好的拟合，并且 **K** 值也大不相同。我们对此更满意。如果我对修剪后的数据拟合三次曲线，会发现三次曲线和直线看起来非常相似。

但问题又来了：我们怎么知道哪条线能更好地代表物理现实？毕竟，我可以只保留任意两个点，然后得到一条完美拟合的直线（均方误差为0）。所以，我们再次看到，这个问题不能仅靠统计学来回答。我必须回到理论。我的理论告诉我，关系应该是线性的，并且我有理论依据来丢弃最后那几个点（超过了弹性极限）。但我没有理论依据去随意删除中间任何我不喜欢的点。

---

让我们继续沿着这条路径，看另一个实验，同样涉及弹簧（但这是不同的“弹簧”）。这次是**弓和箭**。弓的臂本质上是一个弹簧。当你拉回弓弦时，你在臂上施加了力。当你释放时，弹簧（臂）回到其自然位置，将箭射出，使其沿一定轨迹飞行。

我对观察抛射物（箭）遵循的轨迹感兴趣。历史上，很多数学正是源于对抛射轨迹的研究（如火炮弹道）。我进行了一些实验，将数据存入文件，格式类似。

以下是读取和绘制轨迹数据的代码：

```python
def get_trajectory_data(file_name):
    data_file = open(file_name, 'r')
    distances = []
    heights1, heights2, heights3, heights4 = [], [], [], []
    data_file.readline()
    for line in data_file:
        d, h1, h2, h3, h4 = line.split()
        distances.append(float(d))
        heights1.append(float(h1))
        heights2.append(float(h2))
        heights3.append(float(h3))
        heights4.append(float(h4))
    data_file.close()
    return (distances, [heights1, heights2, heights3, heights4])

def process_trajectory(file_name):
    distances, heights = get_trajectory_data(file_name)
    # 计算平均高度
    num_heights = len(heights)
    total_heights = pylab.array([0]*len(distances))
    for h in heights:
        total_heights = total_heights + pylab.array(h)
    mean_heights = total_heights / num_heights

    distances = pylab.array(distances)
    mean_heights = pylab.array(mean_heights)
    pylab.plot(distances, mean_heights, 'bo', label='Measured')
    pylab.xlabel('Distance from Launch Point (inches)')
    pylab.ylabel('Height Above Launch Point (inches)')

    # 尝试拟合直线
    A_lin, B_lin = pylab.polyfit(distances, mean_heights, 1)
    heights_pred_lin = A_lin * distances + B_lin
    pylab.plot(distances, heights_pred_lin, 'r-', label='Linear Fit')
    pylab.legend()
```

拟合直线后，我们发现拟合效果很差。这表明“箭沿直线飞行”的理论不是一个好模型。

现在，让我们尝试拟合一个二次曲线（抛物线，`degree=2`）。

```python
    # 尝试拟合二次曲线（抛物线）
    coeffs_quad = pylab.polyfit(distances, mean_heights, 2)
    heights_pred_quad = pylab.polyval(coeffs_quad, distances)
    pylab.plot(distances, heights_pred_quad, 'g--', label='Quadratic Fit')
```

视觉上，抛物线拟合比直线好得多。这暗示箭可能沿抛物线而非直线飞行。

下一个问题是：我们的眼睛告诉我们它更好，但好多少？我们如何量化比较两种拟合的优劣？回想一下，`polyfit` 通过最小化**均方误差**来工作。因此，一种比较方法是计算直线的均方误差和抛物线的均方误差。显然，抛物线的误差会更小。

然而，均方误差虽然可以用于比较两个模型，但在**绝对意义上**衡量拟合优度并不理想。因为它有下界0，但没有上界，可以任意大。

因此，我们通常使用另一个指标：**决定系数**，通常写作 **R²**。

**R² = 1 - (估计误差 / 测量数据的方差)**

我们比较的是估计误差与数据原始变异性之间的比率。可以证明，R² 的值总是在 0 到 1 之间，这为我们提供了一个在绝对意义上思考拟合优度的好方法。

以下是计算 R² 的代码：

```python
def r_squared(measured, predicted):
    """计算决定系数 R²。
        measured: 测量值的列表或数组。
        predicted: 模型预测值的列表或数组。"""
    estimated_error = ((predicted - measured)**2).sum()
    measured_mean = measured.sum() / float(len(measured))
    measured_variance = ((measured - measured_mean)**2).sum()
    return 1 - estimated_error / measured_variance
```

我们可以用这个函数来量化直线拟合和抛物线拟合的优劣。R² 越接近 1，说明模型解释数据变异性的能力越强。

---

本节课中我们一起学习了：
1.  统计上可靠的模拟结果并不等同于物理真理，必须对概念模型和代码实现进行验证。
2.  科学探索是物理现实、理论模型和计算模型三者不断交互、迭代验证的过程。
3.  实验误差通常可以建模为正态分布（高斯分布）。
4.  可以使用 `polyfit` 进行多项式拟合（如线性回归），并通过最小二乘法找到最佳拟合。
5.  高次多项式可能过度拟合数据，导致模型预测能力变差。必须用理论指导模型选择。
6.  决定系数 **R²** 是一个在 0 到 1 范围内的指标，用于在绝对意义上评估模型的拟合优度，优于原始的均方误差。

理解模型、数据和理论之间的相互作用，是进行有意义的计算科学和工程分析的核心。