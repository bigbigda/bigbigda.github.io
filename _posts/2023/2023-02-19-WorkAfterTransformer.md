---
layout: post
title: Transformer后续
categories: NLP
description: 
keywords: Transformer,attention，Bert
---

[toc]

### 简介

在之前我们介绍了attention-is-all-you-need这篇文章，事实上自从2017年Transformer结构提出后，截止目前其已经席卷了DeepLearning各个应用方向，不管是Bert还是GPT，还是之前介绍的视觉、多模态等工作，其主干网络都离不开Transformer结构。

本文主要是想介绍Bert和GPT之外，其他一些采用Transformer结构的重要工作。比如RoBERTa、BART、T5等

#### 典型工作一句话总结

| 工作     | 介绍                                                         |
| -------- | ------------------------------------------------------------ |
| RoBERTa  | 充分挖掘BERT模型结构优点，仅通过调整训练目标策略和训练数据集就大幅提升了模型效果 |
| DEBEARTa | 主体为encoder结构，SOTA，广泛应用与kaggle比赛                |
| BART     | encoder-decoder结构                                          |
| T5       | encoder-decoder结构，相对位置编码，多个生成任务的SOTA模型    |



### 分类

- **Pretraining Architecture**

  - **Encoder Pretraining**
  - **Decoder Pretraining**
  - **Transformer (Encoder-Decoder) Pretraining**
- Pretraining Task
  - **Language Modeling (LM):**Predict next token (in the case of unidirectional LM) or previous and next token 
  - **Masked Language Modeling (MLM):** mask out some tokens from the input sentences and then trains the model to predict the masked tokens by the rest of the tokens
  - **Permuted Language Modeling (PLM):** same as LM but on a random permutation of input sequences. A permutation is randomly sampled from all possible permutations. Then some of the tokens are chosen as the target, and the model is trained to predict these targets.
  - **Denoising Autoencoder (DAE):** take a partially corrupted input (e.g. Randomly sampling tokens from the input and replacing them with [MASK] elements. randomly deleting tokens from the input, or shuffling sentences in random order) and aim to recover the original undistorted input.
  - **Contrastive Learning (CTL):** A score function for text pairs is learned by assuming some observed pairs of text that are more semantically similar than randomly sampled text.
#### 主要相关模型

![image-20230219224946848](http://pic.inoodles.online/imgimage-20230219224946848.png)

#### 主要相关模型（y轴表示参数量）

![image-20230219225226575](http://pic.inoodles.online/imgimage-20230219225226575.png)

### RoBERTa

RoBERTa: A Robustly Optimized BERT Pretraining Approach

#### 背景

本文发表于2019年，作者发现BERT被低估了，他们指出通过超参的调节、训练数据补充、训练目标的微调，原始BERT的效果可以得到明显的提升，超过当时所有BERT改进工作（例如XLNet）

#### 优化点

- 增大训练时间、增加batchSize大小、更多训练数据
  - 原始BERT使用BOOKCORPUS和English WIKIPEDIA，总共16G未压缩文本
  - 本文使用了五个训练文本，数据总量达160G
- 移除next-sentence预测任务
  - 原始BERT包括两个预训练任务：MLM（预测被mask的词）、NSP（预测输入中第二部分是不是第一部分的下一句）
- 使用更长的训练序列
- 使用动态的mask
- 输入文本编码
  - 使用GPT中提出的BPE
    - BPE是一种sub-character的编码方式，最早用于压缩，主要做法是对所有训练文本中出现频率最高的子串（例如"app")进行编码，最终未编码的用字母直接表示



### DeBERTa

DEBERTA: DECODING-ENHANCED BERT WITH DISENTANGLED ATTENTION

#### 简介

本文在BERT和RoBERTa的基础上进行改进提出了DeBERTa模型，共有15亿参数。基于此模型的大量下游实验都取得了优于RoBERTa的效果。尤其是在SuperGLUE上取得了优于人类的效果。

- disentangled attention mechanism：使用相对位置的位置编码层，attention计算也基于这两部分分别进行
- enhanced mask decoder: 在最终softmax之前考虑绝对位置
- 利用一个对抗式训练方法来增强模型泛化性



### BART

BART: Denoising Sequence-to-Sequence Pre-training for Natural Language Generation, Translation, and Comprehension

Facebook

##### 简介

BART是一个使用sequence-to-sequence结构的denoising autoencoder。

其预训练任务为恢复一段被加噪的文本。作者尝试了许多增加噪音的方法，发现最有效的方法是将输入随机shuffle以及in-filling（将输入中任意长度、包括0长度的片段用**一个**掩码代替）

在NLU任务（GLUE和SQuAD）上与RoBERTa效果类似，在NLG任务（abstractive、question answering、summarization、translation）取得当时SOTA效果。

BART也提出了一个新的机器翻译任务方案：在BART encoder输入端接入一些transformer层，这些层负责将原文本转化为带目标的噪音文本，然后利用BART将这个文本去噪。

##### 模型结构

- 基于标准Transformer结构改造
- ReLU激活函数修改为GeLUs
- decoder每一个层新增了对encoder最后一个隐含层的cross-attention（原始Transformer也是这么处理的）

##### 预训练

- 预训练任务为恢复corrupted文本，loss为decoder输出和原始文本的交叉熵
- Text Infilling：将输入中$\lambda$长度的文本替换为一个[mask]，$\lambda$取自泊松分布，为0时相当于在原文中直接插入一个[mask]
- 针对翻译任务时，首先将BART原结构中encoder的embedding层改为一个随机初始化的encoder，然后进行端到端训练。训练分两步，第一步固定BART大部分参数，主要更新随机初始化encoder、positional embedding，第二步训练全部参数，两步训练均基于BART最终输出对应的交叉熵loss。

##### 实验结果

![image-20230221013500803](http://pic.inoodles.online/imgimage-20230221013500803.png)

### T5

Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer

google，原文67页

作者将所有任务都转化为text-to-text格式，例如英语翻译到德语的输入转化为 "translate English to German：sentence"，一个分类任务输入转为"mnli premise: sentence"。注意这里的前缀本质是一个超参，作者发现修改前缀的文本对最终效果影响不大

<img src="http://pic.inoodles.online/imgimage-20230222004524747.png" alt="image-20230222004524747" style="zoom: 33%;" />

#### 模型结构

- 基于原始Transformer
- 移除Layer Norm的bias；
- 残差连接之后做LayerNorm；
- position encoding（不同频率的正余弦函数）变为相对位置编码，attention函数计算时，Q和V相乘计算attention数值时加入相对位置的编码，编码是可训练的embedding。

#### C4数据集

作者选取了Common Crawl数据集，该数据集每月自动从web上爬取20TB左右的数据，该数据集可能包含各种英语、代码以及内容重复，本文对这些数据做了进一步清理，最终大小约750G

#### 实验

本文的实验非常非常多，下图所示为其中4组

- 预训练任务方面，类似BERT的masked language modeling最好
- Mask策略中，replace spans最好
- mask率为15%时最佳

<img src="/Users/mtdp/Library/Application Support/typora-user-images/image-20230224003929772.png" alt="image-20230224003929772" style="zoom:50%;" />

#### 评估任务

- 文本分类：GLUE & SuperGLUE，是测试通用语言理解能力的文本分类任务的集合，包括句子可接受性判断、情绪分析、释义/句子相似度、自然语言推断、指代消解、完成句子、词义消歧和问题回答。
  - GLUE进一步介绍：https://zhuanlan.zhihu.com/p/135283598
- 机器翻译：WMT English to German, French, and Romanian translation
- 文本摘要：CNN/Daily Mail abstractive summarization
- 智能问答：SQuAD question answering



### 参考资料

https://amatriain.net/blog/transformer-models-an-introduction-and-catalog-2d1e9039f376/ （介绍基于Transformer的50多篇相关工作）

https://docs.google.com/spreadsheets/d/1ltyrAB6BL29cOv2fSpNQnnq2vbX8UrHl47d7FkIf6t4/edit#gid=0 （上文中整理的相关工作code链接及简介）