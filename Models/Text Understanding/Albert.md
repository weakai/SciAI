# Albert

最近各种大体量的预训练模型层出不穷，经常是一个出来刷榜没几天，另外一个又出现了。
BERT、RoBERTa、XLNET等等都是代表人物。
这些“BERT”们虽然一个比一个效果好，但是他们的体量都是非常大的，懂不懂就几千万几个亿的参数量，而且训练也非常困难。
新出的ALBERT就是为了解决模型参数量大以及训练时间过长的问题。

ALBERT 最小的参数只有十几 M,  效果要比 BERT 低 1-2 个点，最大的 xxlarge 也就 200 多M。可以看到在模型参数量上减少的还是非常明显的，但是在速度上似乎没有那么明显。最大的问题就是这种方式其实并没有减少计算量，也就是受推理时间并没有减少，训练时间的减少也有待商榷。

总体来看这个模型并没有那么的让人惊喜，虽然在各个数据及上有更好的效果而且参数量下降了。但是 XXlarge 版本的计算量还是非常大的，这就意味着其在训练和推理上要花更多的时间，作者也说了未来的工作就是提升模型的推理速度。

## 模型详解

整个模型的结构还是依照了 BERT 的骨架，采用了 Transformer 以及 GELU 激活函数。具体的创新部分应该有三个：一个是将 embedding 的参数进行了因式分解，然后就是跨层的参数共享，最后是抛弃了原来的 NSP 任务，现在使用 SOP 任务。这几个更新的细节后面会仔细再说。这三个更新前两个的主要任务就是来进行参数减少的，第三个更新这已经算不上什么更新了，之前已经有很多工作发现原来 BERT 中的下一句话预测这个任务并没有什么积极地影响。根据文章的实验结果来看似乎参数共享对参数降低的影响比较大，同时也会影响模型的整体效果。后面就来详细的说一下这三个改动。

### Factorized embedding parameterization

原始的 BERT 模型以及各种依据 transformer 来搞的预训练语言模型在输入的地方我们会发现它的E是等于H的，其中E就是 embedding  size，H就是 hidden size，也就是 transformer 的输入输出维度。这就会导致一个问题，当我们的 hidden  size 提升的时候，embedding size也需要提升，这就会导致我们的 embedding  matrix 维度的提升。所以这里作者将E和H进行了解绑，具体的操作其实就是在 embedding 后面加入一个矩阵进行维度变换。E 是永远不变的，后面 H 提高了后，我们在E的后面进行一个升维操作，让 E 达到 H 的维度。这使得 embedding 参数的维度从O(V×H)到了O(V×E + E×H), 当E远远小于H的时候更加明显。

### Cross-layer parameter sharing

之前 transformer 的每一层参数都是独立的，包括 self-attention 和全连接，这样的话当层数增加的时候，参数就会很明显的上升。之前有工作试过单独的将 self-attention 或者全连接进行共享，都取得了一些效果。这里作者尝试将所有的参数进行共享，这其实就导致多层的 attention 其实就是一层 attention 的叠加。同时作者通过实验还发现了，使用参数共享可以有效地提升模型的稳定程度。实验结果如下图：

![](https://pic3.zhimg.com/80/v2-85533260658fa30a0b527003208ba2f2_720w.jpg)

### Inter-sentence coherence loss

这里作者使用了一个新的 loss，其实就是更改了原来 BERT 的一个子任务 NSP,  原来 NSP 就是来预测下一个句子的，也就是一个句子是不是另一个句子的下一个句子。这个任务的问题出在训练数据上面，正例就是用的一个文档里面连续的两句话，但是负例使用的是不同文档里面的两句话。这就导致这个任务包含了主题预测在里面，而主题预测又要比两句话连续性的预测简单太多。新的方法使用了 sentence-order prediction(SOP),  正例的构建和 NSP 是一样的，不过负例则是将两句话反过来。**实验的结果也证明这种方式要比之前好很多**。但是这个这里应该不是首创了，百度的 ERNI E貌似也采用了一个这种的。

## 参考

- [ALBERT 论文解读](https://zhuanlan.zhihu.com/p/88099919)