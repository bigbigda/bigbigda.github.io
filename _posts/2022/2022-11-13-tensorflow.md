---
layout: post
title: tensorflow相关代码
categories: snip
description:
keywords: tfrecord,tensorflow
---

### 背景

记录tensorflow使用相关的code

#### 解析tfrecord文件

```python
import os
import sys
import tensorflow as tf
from google.protobuf.json_format import MessageToJson

file_name = 'part-r-00127'  #本地tfrecord文件
examples = tf.python_io.tf_record_iterator(file_name)
example = examples.next()
result = tf.train.Example.FromString(example)

# 分支一（这种方式看到的string可能为编码后的byte
js = MessageToJson(result)
js_dict = eval(js)
# 分支二
extra_info=result.features.feature['extra_info']

```

![image-20221113143849630](http://pic.inoodles.online/imgimage-20221113143849630.png)
