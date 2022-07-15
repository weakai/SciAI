# 维特比算法

### [HMM+Viterbi(维特比算法)+最短路径分析](https://zhuanlan.zhihu.com/p/59889195)

**维特比算法** (Viterbi algorithm) 是机器学习中应用非常广泛的动态规划算法，在求解隐马尔科夫、条件随机场的预测以及 seq2seq 模型概率计算等问题中均用到了该算法。实际上，维特比算法不仅是很多自然语言处理的解码算法，也是现代数字通信中使用最频繁的算法。

### [如何通俗地讲解 viterbi 算法？](https://www.zhihu.com/question/20136144)

如下图，假如你从S和E之间找一条最短的路径，除了遍历完所有路径，还有什么更好的方法？

![](https://pic1.zhimg.com/80/v2-8a0e2bc64b7305a04c3aef847fa62bdb_720w.jpg?source=1940ef5c)

viterbi 维特比算法解决的是篱笆型的图的最短路径问题，图的节点按列组织，每列的节点数量可以不一样，每一列的节点只能和相邻列的节点相连，不能跨列相连，节点之间有着不同的距离，距离的值就不在图上一一标注出来了，大家自行脑补