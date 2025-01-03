# 图解LLM推理完整流程---Attention算子

> Written  In 2024_11.16 ___by yxl   
> Last Edited In 2024_12.26___by yxl


在之前的文章中，我们已经了解了Transformer的基本架构和Attention的机制思想：

而本文的目的是为了探究LLM在推理过程中Attention算子的真实细节。事实上由于现在网上大部分人们在介绍LLM的Attention或者LLM时，都会或多或少简化其中的计算过程。并且在描述流程时，对模型的参数名称进行了各式各样的修改。这就导致在我们想要尝试自己计算一个LLM模型大小参数、或者一次前向推理需要多少次计算操作时，会发现根本无从下手。

故本文将会通过具体的矩阵计算过程图来探究在Attention机制究竟是如何在大语言模型LLM的推理过程中发挥作用的。

---

在现在大部分LLM当作，多采用Multi-SelfAttention多头的注意力机制。但如果在不清楚Attention计算流程的情况下直接了解多头注意力会感觉较为复杂。所以本文将先介绍单头的SelfAttention的计算过程。（注意！Self-Attention的计算过程只对理解Attention机制有帮助，其实际的参数和计算过程与多头注意力机制有较大的出入）

## Self-Attention

*注意：Self-Attention的流程图主要用于理解Attention计算过程。更严格地说，只表达了LLM中Perfill的过程，而Decode的流程将会在后续给出。*

![self-attention算子图](../images/04.png)

下面进行一些变量名称解释与约定：
1. 流程图中矩阵块的长和宽严格遵守了矩阵乘法的规则
2. 名称为黄色的都是参数矩阵，名称为紫色的是输入流动的数据。
3. S 为输入文本的序列长度，例如输入：“同学们大家好”，那么S应该为 6。
4. V为预训练时词汇表的长度，通常大小为250,000左右。
5. d_model： 模型隐变量维度，是衡量模型参数大小的重要指标。也是词嵌入层的输出维度。
6. Matmul：矩阵乘操作。
7. Scale ： 缩放操作，就是原公式中那个除以 根号dk 的操作

可以看到，如果仅仅是单头的自注意力机制，那么在整个Attention过程中，计算参数只有词嵌入矩阵和词嵌入位置矩阵。（输出矩阵Wo与词嵌入矩阵共享参数，即转置关系）。

我们会发现整个注意力相关的计算过程没有什么可以学的参数。所以作者借鉴了CNN网络计算过程中“多通道”的思想：即对一个单通道的样本拓展至多通道数，来增加学习到特征的多样性。从而就有了多头注意力机制。

接下来让我们一起看一看多头注意力机制的计算过程：

## Multi-SelfAttention

下面的计算计算过程是严格按照Transformer论文中数据的计算格式和名称进行绘制的。在后续计算LLM的推理计算量和模型参数时，可直接使用该图作为参照标准。

![Multi-SelfAttention](../images/05.png)

注意：变量h为多头注意力机制的头数。并且在改图中h应该为2，虽然图中画了3层，那只是为了说明该矩阵是一个三维矩阵，并不代表其高h为3。并且作者规定：d_model = d_k * h，后续会讲解为什么）

我们可以看到，整个流程被分为了几块：Vocabulary Embedding词嵌入层、Linear线性映射层、Scale-Dot-Product Attention 缩放点乘注意力层。

其中Scale-Dot-Product Attention层的计算过程与之前的SelfAttention中的计算过程完全对等，只不过将原本的二维的Q、K、V向量矩阵变成了三维张量Tensor。但是在具体计算过程中，还是每个二维的向量切片做计算，最后叠加至三维。

此过程与SelfAttention最大区别是，多了一个Linear线性映射层。那么怎样通俗的理解这一层在做什么呢？

![](../images/06.png)

以上便是LLM在Perfill过程中Attention算子计算的全部过程。后续推理的Decode过程、KV cache以及模型参数的计算将会在下一章给出。

> Click here to jump the Next Chapter : (ops!To be done)