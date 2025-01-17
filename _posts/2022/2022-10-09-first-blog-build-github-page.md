---
layout: post
title: 第一篇博客：github pages搭建
categories: Engineer
description: 记录搭建github pages
keywords: 工程、博客搭建
---

### github pages搭建

- 2022-10-09 终于搭建完成了基于github的个人博客
- 2022-10-11 buy Typora
- 2022-10-11 解决图片问题



#### 解决图片问题记录

- 方案一：参考原始代码，将图片放在blog同路径下
  - 最初尝试在one-blog.md同路径下创建one-blog-images目录，将图片放在其中，但实际网站上无法展示图片
  - 最终放在/images/one-blog-images下可以解决此问题，但是这种方式**图片解析很慢**，且由于图片比较多，以后有超过github-pages 1G 空间限制风险，遂放弃
- 方案二：使用typora+ipic+微博图床，实现复制图片时自动update到云端，同时md文件中保存的也是url。可以完美解决方案一的问题。
  - 方案二引入的新问题是如果以后微博图床失效，那整个博客有挂掉的风险。最初希望通过配置typora实现上传的同时保存到本地，但没找到此功能。**后续计划**自己写python实现此功能
  - 目前优化为**typora+PicGo+七牛云**，考虑到七牛云也在提供收费服务，稳定性有明显提升
  - 七牛云国内存储需要备案域名，而域名备案不仅需要有国内主机（例如阿里云主机），还需要在工信部各种登记审批，所以最终采用的是北美区域存储

