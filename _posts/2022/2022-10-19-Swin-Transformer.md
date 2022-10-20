---
layout: post
title: Swin Transformer
categories: CV
description:
keywords: transformer, object detection
---

## Swin Transformer: Hierarchical Vision Transformer using Shifted Windows

Swin Transformer

MSRA

ICCV 2021 best paper

### 简介

本文发表在VIT之后，作者提出虽然VIT等工作已经将transformer使用在CV领域中，但VIT主要使用在分类任务上，本文将对transformer在CV的应用做进一步完善，使其成为CV各个任务的backbone

VIT等cv transformer工作之所以取得成功，一个主要原因是利用了transformer建模全局信息的能力，但是其也存在一个问题，即未能学习到更细粒度以及层次化的图像表达，而这类信息往往对检测、分割等任务非常重要。一个例子就是VIT在小物体检测上表现并不好。

针对VIT无法很好建模细粒度信息的问题，一个直接原因是VIT的patch大小为16x16，即每个patch过大。直接的方法是减少patch，但这会导致encoder结构输入序列过长，计算复杂度过大，并不可行。

swin transformer主要引入了两个优化：

- 层次化的图像表示（通过相邻Swin Transformer Block之间的Patch Merging）
- 跨Patch的信息交互（通过Shifted Window Attention机制）



### 整体结构

<img src="http://pic.inoodles.online/imgimage-20221019235443985.png" alt="image-20221019235443985" style="zoom:50%;" />

### 实验效果

在分类、检测、分割三大任务上显著优于VIT/DeiT/ResNe(X)t

### 

