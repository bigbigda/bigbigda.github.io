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

example = examples.next()

result = tf.train.Example.FromString(example)

result.features.feature['x_255']

result.features.feature['x_73']



#### tfrecord和tf.train.Example

##### 什么是TFRecord

tfrecord是tensorflow training和inference中标准的数据存储格式之一。对于工业界来说，数据的高效存储和读取性能对模型落地非常重要，二进制的存储格式具有占用空间少，拷贝和读取（from disk）更加高效的特点。tfrecord便是为了解决这个问题而产生的一种record-oriented的标准数据表示形式。其本质是一行一行字节串构成的样本数据。

![image-20230817192551631](http://pic.inoodles.online/imgimage-20230817192551631.png)



它实质上是由protobuf定义的一种数据协议，一条tfrecord数据代表一条样本数据，也可以叫做一个Example，其中tensorflow提供了两种Example表示形式 Example和SequenceExample。它的定义代码位于[tensroflow/core/example/example.proto & feature.proto]。

##### 样本生产

```python
import tensorflow as tf

# 回忆上一小节介绍的，每个Example内部实际有若干种Feature表达，下面
# 的四个工具方法方便我们进行Feature的构造
def _bytes_feature(value):
    return tf.train.Feature(bytes_list=tf.train.BytesList(value=[value]))

def _float_feature(value):
    return tf.train.Feature(float_list=tf.train.FloatList(value=[value]))

def _int64_feature(value):
    return tf.train.Feature(int64_list=tf.train.Int64List(value=[value]))

def _int64list_feature(value_list):
    return tf.train.Feature(int64_list=tf.train.Int64List(value=value_list))

# 单条样本的生产逻辑
def serialize_example(user_id, city_id, app_type, viewd_pois, avg_paid, comment):
		# 注意我们需要按照格式来进行数据的组装，这里的dict便按照指定Schema构造了一条Example
    feature = {
      'user_id': _int64_feature(user_id),
      'city_id': _int64_feature(city_id),
      'app_type': _int64_feature(app_type),
      'viewd_pois': _int64list_feature(viewd_pois),
      'avg_paid': _float_feature(avg_paid),
      'comment': _bytes_feature(comment),
    }
		# 调用相关api将Example数据序列化为byte string
    example_proto = tf.train.Example(features=tf.train.Features(feature=feature))
    return example_proto.SerializeToString()

# 样本的生产，这里展示了将2条样本数据写入到了文件中
def write_demo(filepath):
    with tf.python_io.TFRecordWriter(filepath) as writer:
        writer.write(serialize_example(1, 10, 1, [658, 325], 36.3, "yummy food."))
        writer.write(serialize_example(2, 20, 2, [897, 568, 126], 89.6, "nice place to have dinner."))
    print "write demo data done."

filepath = "testdata.tfrecord"
write_demo(filepath)
```
##### 样本解析

```python
def read_demo(filepath):
		# 定义要如何读取数据，定义schema
    schema = {
        'user_id': tf.FixedLenFeature([], tf.int64),
        'city_id': tf.FixedLenFeature([], tf.int64),
        'app_type': tf.FixedLenFeature([], tf.int64),
        'viewed_pois': tf.VarLenFeature(tf.int64),
        'avg_paid': tf.FixedLenFeature([], tf.float32, default_value=0.0),
        'comment': tf.FixedLenFeature([], tf.string, default_value=''),
    }
		
    # 使用相关api，按照schema解析样本数据
    def _parse_function(example_proto):
        return tf.parse_single_example(example_proto, schema)
		
    # 使用TFRecordDataset加载tfrecord格式的文件
    dataset = tf.data.TFRecordDataset(filepath)
    parsed_dataset = dataset.map(_parse_function)
    next = parsed_dataset.make_one_shot_iterator().get_next()
    
    # 这里直接利用session，简单打印数据
    with tf.Session() as sess:
        while True:
            try:
                print sess.run(next)
            except:
                print "out of data"
                break
```



- 官网资料：https://www.tensorflow.org/tutorials/load_data/tfrecord

#### FixedLenFeature和VarLenFeature

| API名              | 功能                                                         | 参数                                                         |
| :----------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| tf.FixedLenFeature | 将数据以Dense的形式读取出来                                  | shape：定义样本数据该字段的shape，注意我们demo中的scalar shape都定义为[]dtype：数据类型defaule_value：默认值 |
| tf.VarLenFeature   | 读取变长特征数据，数据被读取为[SparseTensor](https://www.tensorflow.org/api_docs/python/tf/sparse/SparseTensor)的格式。例如view_pois是一个array，因此采用VarLenFeature进行读取 | dtype：数据类型                                              |

 

