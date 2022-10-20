---
layout: post
title: Swin Transformer
categories: CV
description:
keywords: transformer,
---

## Swin Transformer: Hierarchical Vision Transformer using Shifted Windows

Swin Transformer

MSRA

ICCV 2021 best paper

### 简介

本文发表在VIT之后，作者提出虽然VIT等工作已经将transformer使用在CV领域中，但VIT主要使用在分类任务上，本文将对transformer在CV的应用做进一步完善，使其成为CV各个任务的backbone

swin transformer主要引入了两个优化：（1） 层次化的底层表示；（2）移动窗口



### 整体结构

<img src="http://pic.inoodles.online/imgimage-20221019235443985.png" alt="image-20221019235443985" style="zoom:50%;" />

### 实验效果

在分类、检测、分割三大任务上显著优于VIT/DeiT/ResNe(X)t

### 后续

