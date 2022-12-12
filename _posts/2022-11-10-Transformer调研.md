---
layout:     post
title:     Transformer调研
subtitle:   论文阅读&代码复现
date:       2022-11-10
author:     ldf
header-img: img/post-bg-transformer.jpg
catalog: true
tags:
    - research
---

# Transformer调研

## 一、介绍 Introduction

2017年Google发表的论文《Attention Is All You Need》中提出了一种新的模型Transformer。它与FaceBook的《Convolutional Sequence to Sequence Learning》都算是Seq2Seq上的创新，本质上来说，都是抛弃了RNN结构来做Seq2Seq任务。

这个模型最初是为了提高机器翻译的效率，它的**Self-Attention机制**和**Position Encoding**可以替代RNN。因为RNN是串行执行的，t时刻没有完成就不能处理t+1时刻，因此很难并行。但是后来发现Self-Attention效果很好，在很多其它的地方也可以使用Transformer模型。

- 这包括著名的OpenAI GPT和**BERT模型**，都是以Transformer为基础的。当然它们只使用了Transformer的Decoder部分，由于没有了Encoder，所以Decoder只有Self-Attention而没有普通的Attention。




## 二、 模型架构 Model Architecture

Transformer 架构是以encoder/decoder架构为基础，整体结构如下图所示，在Encoder和Decoder中都使用了Self-attention, Point-wise和全连接层。Encoder和decoder的大致结构分别如下图的左半部分和右半部分所示。

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20221212202403.png)

 

 

### 2.1 编码组件 Encoder

编码组件部分由一堆编码器（encoder）构成（论文中是将6个编码器叠在一起——数字6没有什么神奇之处，你也可以尝试其他数字）。解码组件部分也是由相同数量（与编码器对应）的解码器（decoder）组成的。

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20221212202355.png)

每个编码器的结构均相同（但它们不共享权重），每层有两个子层：自注意力层（self-attention）（这层帮助编码器在对每个单词编码时关注输入句子的其他单词） 和 全连接的前馈网络层（feed-forward）。

![img](file:///C:/Users/705lab/AppData/Local/Temp/msohtmlclip1/01/clip_image005.png)

每两个子层中外都套了一个残差连接（residual connections），然后是层归一化（layer normalization）。也就是说，每个子层的输出是 LayerNorm(x+Sublayer(x)) ，其中Sublayer(x)是子层本身实现的功能。为了促进这些残差连接，模型中的所有子层以及嵌入层都会产生维度![img](file:///C:/Users/705lab/AppData/Local/Temp/msohtmlclip1/01/clip_image007.png)=512的输出。

### 2.2 解码组件 Decoder

解码组件也是由6个相同的解码器堆叠而成，也有自注意力（self-attention）层和前馈（feed-forward）层。除此之外，这两个层之间还有一个注意力层（Encoder-Decoder Attention），用来关注输入句子的相关部分（和seq2seq模型的注意力作用相似）。

![img](file:///C:/Users/705lab/AppData/Local/Temp/msohtmlclip1/01/clip_image009.jpg)

我们还修改解码器中的Self-attention子层以防止当前位置Attend到后续位置。这种Masked的Attention是考虑到输出Embedding会偏移一个位置，确保了生成位置i的预测时，仅依赖小于i的位置处的已知输出，相当于把后面不该看到的信息屏蔽掉。

**第一个编码器的输入是一个序列，最后一个编码器的输出是一组注意力向量 Key 和 Value。**这些向量将在每个解码器的 Encoder-Decoder Attention 层被使用，这有助于解码器把注意力集中在输入序列的合适位置。

不同之处是：Encoder-Decoder Attention 层使用前一层的输出构造 Query 矩阵，而 Key 和 Value 矩阵来自于解码器栈的输出：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20221212202348.png)







### 2.3 自注意力机制 Self-Attention

**Attention** **定义：**注意力函数可以描述为将查询和一组键值对映射到输出，其中Query、keys、values和output都是向量vectors。输出计算为值的加权和，其中分配给每个值的权重由查询与相应键的兼容性函数计算。

![img](file:///C:/Users/705lab/AppData/Local/Temp/msohtmlclip1/01/clip_image010.png) 

Google给出的Attention具体公式定义为：

![img](file:///C:/Users/705lab/AppData/Local/Temp/msohtmlclip1/01/clip_image011.png)

#### 2.3.1 举例分析

1. 对编码器的每个输入向量（在本例中，即每个词的词向量）创建三个向量：Query 向量、Key 向量和 Value 向量。它们是通过词向量分别和 3 个矩阵相乘得到的，这 3 个矩阵通过训练获得。

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20221212202341.png)

图 1.12 中，$X_1$  乘以权重矩阵$W_Q$得到$q_1$，即与该单词关联的 Query 向量。最终会为输入句子中的每个词创建一个 Query，一个 Key 和一个 Value 向量。

2. 计算注意力分数。假设我们正在计算这个例子中第一个词 “Thinking” 的自注意力。我们需要根据 “Thinking” 这个词，对句子中的每个词都计算一个分数。这些分数决定了我们在编码 “Thinking” 这个词时，需要对句子中其他位置的每个词放置多少的注意力。

 这些分数，是通过计算 “Thinking” 的 Query 向量和需要评分的词的 Key 向量的点积得到的。如果我们计算句子中第一个位置词的注意力分数，则第一个分数是 $q_1$  和 $k_1$ 的点积，第二个分数是 $q_1$  和 $k_2$ 的点积。

3. 将每个分数除以 $\sqrt{d_{k}}$   （$d_{k}$ 是 Key 向量的维度）。目的是在反向传播时，求梯度更加稳定。

4. 将这些分数进行 Softmax 操作。Softmax 将分数进行归一化处理，使得它们都为正数并且和为 1。(这些 Softmax 分数决定了在编码当前位置的词时，对所有位置的词分别有多少的注意力。)

5. 将每个 Softmax 分数分别与每个 Value 向量相乘。这种做法背后的直觉理解是：对于分数高的位置，相乘后的值就越大，我们把更多的注意力放在它们身上；对于分数低的位置，相乘后的值就越小，这些位置的词可能是相关性不大，我们就可以忽略这些位置的词。
6. 将加权 Value 向量（即上一步求得的向量）求和。这样就得到了自注意力层在这个位置的输出。

整个计算过程如下图：

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20221212202334.png)

#### 2.3.2 多头注意力机制（Multi-head Attention）

在 Transformer 论文中，通过添加一种多头注意力机制，进一步完善了自注意力层。具体做法：首先，通过 h hh 个不同的线性变换对 Query、Key 和 Value 进行映射；然后，将不同的 Attention 拼接起来；最后，再进行一次线性变换。

![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20221212202213.png)

为什么要引入这个机制呢？

- 自注意力机制的缺陷就是：模型在对当前位置的信息进行编码时，会过度的将注意力集中于自身的位置
- 我们希望模型可以基于相同的注意力机制学习到不同的行为，然后将不同的行为作为知识组合起来，例如捕获序列内各种范围的依赖关系（例如，短距离依赖和长距离依赖）。因此，允许注意力机制组合使用查询、键和值的不同的 子空间表示（representation subspaces）可能是有益的。
- 通过独立学习得到 h 组不同的 线性投影（linear projections）来变换Query、key和value，来并行地进行注意力池化。
- 最后，将这 h个注意力池化的输出拼接在一起，并且通过另一个可以学习的线性投影进行变换，以产生最终输出。

### 2.4 引入张量

在Transformer中，各个模块间的信息流动依靠的是向量/张量

和通常的 NLP 任务一样，首先，我们使用词嵌入算法（Embedding）将每个词转换为一个词向量。在 Transformer 论文中，词嵌入向量的维度是 512。



嵌入仅发生在最底层的编码器中。所有编码器都会接收到一个大小为 512 的向量列表——底部编码器接收的是词嵌入向量，其他编码器接收的是上一个编码器的输出。**这个列表大小是我们可以设置的超参数**——基本上这个参数就是训练数据集中最长句子的长度。

 

### 2.5 位置前馈网络（Position-wise Feed-Forward Networks）

 位置前馈网络就是一个全连接前馈网络，每个位置的词都单独经过这个完全相同的前馈神经网络。其由**两个线性变换**组成，即**两个全连接层**组成，**第一个全连接层的激活函数就是简单的 ReLU 激活函数**。可以表示为：

$$
FFN(x)=max(0,xW 
1
​	
 +b 
1
​	
 )W 
2
​	
 +b 
2
​
$$
在每个编码器和解码器中，虽然这个全连接前馈网络结构相同，但是不共享参数。 在Transformer论文中，整个前馈网络的输入和输出维度都是 $d_{model}=512$，第一个全连接层的输出和第二个全连接层的输入维度为 $d_{ff}=2048$。

###  2.6 残差连接和层归一化(层间操作)——ADD&Normalization

 编码器结构中有一个需要注意的细节：每个编码器的每个子层（Self-Attention 层和 FFN 层）都有一个残差连接，再执行一个层标准化操作，整个计算过程可以表示为：$sub\_layer\_output=LayerNorm(x+SubLayer(x))$

 ![](https://raw.githubusercontent.com/BBQldf/PicGotest/master/20221212202259.png)

**为什么用残差网络：**

- Add代表了Residual Connection，是为了解决多层神经网络训练困难的问题。
- 因为对于有些层，我们并不确定其效果是不是正向的。加了残差连接之后，我们相当于将上一层的信息兵分两路，一部分通过我们的层进行变换，另一部分直接传入下一层，再将这两部分的结果进行相加作为下一层的输出。
- 我们通过残差连接之后，就算再不济也至少可以保留上一层的信息，这是一个非常巧妙的思路。(在ResNet中被广泛使用)

**为什么使用LayerNorm？（其实就是问，为什么不用BatchNorm？）：**

- Layer Normalization，通过对层的激活值的归一化，可以加速模型的训练过程，使其更快的收敛
- batchNorm是在batch上进行处理，对每个batch的数据进行归一化，对小batchsize效果不好；（本质上，就是对同一个batch下不同样本的同一个特征做归一化），它有一些缺点：
  - 由于每次计算均值和方差是在一个batch上，所以如果batchsize太小，则计算的均值、方差不足以代表整个数据分布；
  - BN实际使用时需要计算并且保存某一层神经网络batch的均值和方差等统计信息，对于对一个固定深度的前向神经网络（DNN，CNN）使用BN，很方便；**但对于RNN来说，sequence的长度是不一致的，**换句话说RNN的深度不是固定的，不同的time-step需要保存不同的特征，可能**存在一个特殊sequence比其他sequence长很多，**（那它的均值等的计算就会出现波动）这样训练时，计算很麻烦。
- layerNorm在通道方向上做归一化，主要对RNN作用明显；（对同一个样本的不同特征做归一化（针对所有样本，并且是样本独立的））,他也有一些缺点：
  -  layer normalization 对所有的特征进行缩放，这显得很没道理。我们算出一行这【身高、体重、年龄】三个特征的均值方差并对其进行缩放，事实上会因为特征的量纲不同而产生很大的影响。但是BN则没有这个影响，因为BN是对一列进行缩放，一列的量纲单位都是相同的。
- 但是在NLP领域，LN是更合适的，这是因为：
  -  如果我们将一批文本组成一个batch，那么BN的操作方向是，对每句话的第一个词进行操作。但语言文本的复杂性是很高的，任何一个词都有可能放在初始位置，且词序可能并不影响我们对句子的理解。而BN是针对每个位置进行缩放，这不符合NLP的规律。

<font color='red'>注意：</font>

- 在BN和LN都能使用的场景中，BN的效果一般优于LN，原因是基于不同数据，同一特征得到的归一化特征更不容易损失信息。

### 2.7 位置编码

相对于RNN（上一个时刻的输出作为下一个时刻的输入来传递历史信息，天然的时序操作），Transformer是一次性把数据进行计算，除了需要知道词之间的距离以外，我们还应了解词之间的顺序，为此，引入了**位置编码**。

其具体的数学公式如下：
$$
PE_{(pos, 2i)} = \sin(pos / 10000^{2i/d_{model}})\\
PE_{(pos, 2i+1)} = \cos (pos / 10000^{2i/d_{model}})
$$
这里的意思是将id为p的位置映射为一个dpos维的位置向量，这个向量的第i个元素的数值就是PEi(p)。

> 注意：这是Transformer原始论文使用的位置编码方法，而在BERT模型里，使用的是简单的可以学习的Embedding，和Word Embedding一样，只不过输入是位置而不是词而已。

这个公式其实就是对位置的一种Map。可以参考计算机中的进制表示，比如15，就是1111，6就是0110。

### 2.8 Mask（掩码）——层内，层间两种

#### 2.8.1 Padding Mask

因为每个批次输入序列的长度是不一样的，所以我们要对输入序列进行对齐。具体来说，就是在较短的序列后面填充 0（但是如果输入的序列太长，则是截断，把多余的直接舍弃）。

由于后面还要经过softmax层，具体的做法：把这些位置的值加上一个非常大的负数（负无穷），这样的话，经过 Softmax 后，这些位置的概率就会接近 0。

#### 2.8.2 Sequence Mask

Sequence Mask 是为了使得 Decoder 不能看见未来的信息。也就是对于一个序列，在 t 时刻，我们的解码输出应该只能依赖于 t 时刻之前的输出，而不能依赖 t之后的输出。因为我们需要想一个办法，把 t之后的信息给隐藏起来。

具体的做法：产生一个上三角矩阵，上三角的值全为 0。把这个矩阵作用在每个序列上，就可以达到我们的目的。

### 2.9 最后的线性层和 Softmax 层

线性层是一个简单的全连接神经网络，其将解码器栈的输出向量映射到一个更长的向量，这个向量被称为 logits 向量。

现在假设我们的模型有 10000 个英文单词（模型的输出词汇表）。因此 logits 向量有 10000 个数字，每个数表示一个单词的分数。

然后，Softmax 层会把这些分数转换为概率（把所有的分数转换为正数，并且加起来等于 1）。最后选择最高概率所对应的单词，作为这个时间步的输出。

在 Transformer 论文，提到一个细节：编码组件和解码组件中的嵌入层，以及最后的线性层共享权重矩阵。不过，在嵌入层中，会将这个共享权重矩阵乘以$\sqrt{d_{model}}$。

### 2.10 正则化操作

为了提高 Transformer 模型的性能，在训练过程中，使用了以下的正则化操作：

1. Dropout。对编码器和解码器的每个子层的输出使用 Dropout 操作，是在进行残差连接和层归一化之前。词嵌入向量和位置编码向量执行相加操作后，执行 Dropout 操作。Transformer 论文中提供的参数$P_{drop} = 0.1P$。
2. Label Smoothing（标签平滑）。Transformer 论文中提供的参数 $\epsilon_{ls} = 0.1$

## 三、实验复现

针对两篇论文，目前已经给出了相应的代码：

1. [HeteroFL: Computation and Communication Efficient Federated Learning for Heterogeneous Clients](https://github.com/dem123456789/HeteroFL-Computation-and-Communication-Efficient-Federated-Learning-for-Heterogeneous-Clients)
2. [No One Left Behind: Inclusive Federated Learning over Heterogeneous Devices](https://github.com/Rachelxuan11/InclusiveFL)







