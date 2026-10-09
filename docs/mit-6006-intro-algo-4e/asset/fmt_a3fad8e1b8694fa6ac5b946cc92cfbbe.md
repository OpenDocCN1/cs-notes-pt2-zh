
给出一个需要应用级别算法内容的应用示例，并讨论所涉及算法的功能。

### 1.2-2

假设在特定计算机上，对于大小为`n`的输入，插入排序需要`8n²`步，而归并排序需要`64n lg n`步。对于哪些`n`值，插入排序胜过归并排序？

### 1.2-3

什么是`n`的最小值，使得运行时间为`100n²`的算法在同一台机器上比运行时间为`2^n`的算法更快？

**问题**

### 1-1 运行时间的比较

对于每个函数`f(n)`和时间`t`在下表中，确定在时间`t`内可以解决的问题的最大规模`n`，假设解决问题的算法需要`f(n)`微秒。

![艺术](img/Art_P5.jpg)

**章节注释**

有许多关于算法这一一般主题的优秀文本，包括 Aho，Hopcroft 和 Ullman 的文本[5, 6]，Dasgupta，Papadimitriou 和 Vazirani 的文本[107]，Edmonds 的文本[133]，Erickson 的文本[135]，Goodrich 和 Tamassia 的文本[195, 196]，Kleinberg 和 Tardos 的文本[257]，Knuth 的文本[259, 260, 261, 262, 263]，Levitin 的文本[298]，Louridas 的文本[305]，Mehlhorn 和 Sanders 的文本[325]，Mitzenmacher 和 Upfal 的文本[331]，Neapolitan 的文本[342]，Roughgarden 的文本[385, 386, 387, 388]，Sanders，Mehlhorn，Dietzfelbinger 和 Dementiev 的文本[393]，Sedgewick 和 Wayne 的文本[402]，Skiena 的文本[414]，Soltys-Kulinicz 的文本[419]，Wilf 的文本[455]，以及 Williamson 和 Shmoys 的文本[459]。有关算法设计更实际方面的讨论可以参考 Bentley 的文本[49, 50, 51]，Bhargava 的文本[54]，Kochenderfer 和 Wheeler 的文本[268]，以及 McGeoch 的文本[321]。算法领域的概述也可以在 Atallah 和 Blanton 的书中找到[27, 28]，以及 Mehta 和 Sahhi 的书中[326]。对于较少技术性的材料，请参阅 Christian 和 Griffiths 的书籍[92]，Cormen 的书籍[104]，Erwig 的书籍[136]，MacCormick 的书籍[307]，以及 Vöcking 等人的书籍[448]。计算生物学中使用的算法概述可以在 Jones 和 Pevzner 的书籍中找到[240]，Elloumi 和 Zomaya 的书籍中找到[134]，以及 Marchisio 的书籍中找到[315]。

¹ 有时，当问题背景已知时，问题实例本身简称为“问题”。

² 更确切地说，只有决策问题——即具有“是/否”答案的问题——才能是 NP 完全的。旅行推销员问题的决策版本询问是否存在一种顺序的停靠点，其距离总和最多为给定数量。
