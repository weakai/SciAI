---
title: DeBERTa
data: 2022-7-18
---

微软提出了一种新的预训练语言模型 DeBERTa。它从两方面改进 BERT 和 RoBEARTa，实验表明，DeBERTa 在许多下游 NLP 任务上表现都优于 RoBERTa 和 BERT。

本文提出了一种新的基于 Transformer 的神经语言模型 DeBERTa（Decoding enhanced BERT with  discentanged attention），它被证明比 RoBERTa 和 BERT 作为 PLM 更有效，并且经过微调后，在一系列 NLP 任务中取得了更好的效果。

## 修改

### 分离的注意机制

在 BERT 中，输入层中的每个单词都是用一个向量表示的，这个向量是单词（内容）嵌入和位置嵌入的总和，而 DeBERTa  中的每个单词都是用两个向量表示的，这两个向量分别对其内容和位置进行编码，并且单词之间的注意力权重是根据单词的位置和内容来计算的内容和相对位置。这是因为观察到一对词的注意力权重不仅取决于它们的内容，而且取决于它们的相对位置。例如，当单词“deep”和“learning”相邻出现时，它们之间的依赖性要比出现在不同句子中时强得多。

### 增强型掩码解码器

第二，DeBERTa 在预训练时增强了 BERT 的输出层。在模型预训练过程中，将 BERT 的输出 Softmax  层替换为一个增强的掩码解码器（EMD）来预测被屏蔽的令牌。这是为了缓解训练前和微调之间的不匹配。在微调时，我们使用一个任务特定的解码器，它将  BERT 输出作为输入并生成任务标签。然而，在预训练时，我们不使用任何特定任务的解码器，而只是通过 Softmax 归一化 BERT  输出（logits）。因此，我们将掩码语言模型（MLM）视为任何微调任务，并添加一个任务特定解码器，该解码器被实现为两层 Transformer 解码器和 Softmax 输出层，用于预训练。

在标准的 BERT 预训练期间，我们通过词汇表将与掩码令牌对应的最终隐藏向量输送给输出 Softmax 层。在微调过程中，我们将 BERT 输出输入到一个特定于任务的解码器中，该解码器具有一个或多个特定任务的解码层，如果输出是概率，则再加上一个 Softmax 层。为了减少预训练和微调之间的不匹配，我们将 MLM 和其他下游任务一样对待，并用包含一个或多个 Transformer 层和一个 Softmax 输出层的增强型掩码解码器（EMD）替换 BERT 的输出 Softmax。这使得结合 BERT 和 EMD 进行预训练的 DeBERTa 成为一个编码器-解码器模型。