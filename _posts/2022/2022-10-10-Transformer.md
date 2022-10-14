---

layout: post
title: Attention-Is-All-You-Need
categories: Transformer
description:
keywords: attention,transformer

---

# Attention Is All You Need

### 整体结构

<img src="http://pic.inoodles.online/imgimgimage-20221012011539932.png" alt="image-20221012011539932" style="zoom:50%;" />

<img src="http://pic.inoodles.online/imgimgimage-20221012002512478.png" alt="image-20221012002512478" style="zoom:50%;" />

#### Attention计算

$Attention(Q, K, V) = softmax(\frac{QK^T}{\sqrt{d_k}})V$

### 实现细节

| 参数名            | 参数含义                | base模型取值 |
| ----------------- | ----------------------- | ------------ |
| N                 | encoder/decoder层数     | 6            |
| $d_{model}$       | embedding维度           | 512          |
| h                 | head数量                | 8            |
| $d_{k}$ / $d_{v}$ | key、value、query的维度 | 64           |

#### 输入层embedding

- 对于英文，进行token embedding时可以以单词的粒度token化，也可以以词根的粒度token化；对于中文，一般以字为单位
- 本文实现中embedding同encoder/decoder部分联合训练，且input embedding和output embedding共享参数
- decoder的embedding层和FC层权重共享：embedding层可以说是通过one-hot获取对应的embedding向量，FC层则是通过向量x去得到它为各个词的softmax概率（理论上也是one-hot），所以这两个计算近似为互逆过程。

### 其他参考资料

http://jalammar.github.io/illustrated-transformer/

https://nlp.seas.harvard.edu/2018/04/03/attention.html 

https://zhuanlan.zhihu.com/p/132554155
