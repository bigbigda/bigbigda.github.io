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
##### 主要相关模型

![image-20230219224946848](http://pic.inoodles.online/imgimage-20230219224946848.png)

##### 主要相关模型（y轴表示参数量）

![image-20230219225226575](http://pic.inoodles.online/imgimage-20230219225226575.png)

### RoBERTa

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



#### DeBERTa

#### BART

#### T5



### 参考资料

https://amatriain.net/blog/transformer-models-an-introduction-and-catalog-2d1e9039f376/ （介绍基于Transformer的50多篇相关工作）

https://docs.google.com/spreadsheets/d/1ltyrAB6BL29cOv2fSpNQnnq2vbX8UrHl47d7FkIf6t4/edit#gid=0 （上文中整理的相关工作code链接及简介）