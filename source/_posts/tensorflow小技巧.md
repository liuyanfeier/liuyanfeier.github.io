---
title:      tensorflow小技巧
author:     liuyan
catalog:    true
tags:
  - tensorflow
  - deep learning
date: 2017-10-14 16:37:39
urlname:
categories: deep learning
---

### 显卡问题

```sh
import os
import tensorflow as tf

os.environ['CUDA_VISIBLE_DEVICES'] = '2,3' # 指定哪张卡：0, 1, 2, ...

config = tf.ConfigProto()
config.gpu_options.allow_growth = True  # 随着进程逐渐增加显存占用，而不是一下占满
session = tf.Session(config=config, ...)
```

<!-- more -->

那么如果你想要给某个操作指定2号显卡的时候，由于CUDA只能看到2块，2,3号就相当于0,1号卡(而不是'/gpu:2'和'/gpu:3')，则需要用：

```sh
with tf.device('/gpu:0'):
	"your ops here"
```

### 查看ckpt中的网络结构和参数

```sh
python tensorflow/python/tools/inspect_checkpoint.py  
--file_name=model.ckpt-158940 --tensor_name=unit_1_1/conv1/Weights  
```

如果你只用了file_name这个参数，那么看到的就是整体网络结构。如果你2个参数都用了，那看到的就是具体那一层的值。

或者开源项目：MMdnn，先把ckpt转成给定的中间表示，再看网络结构。

### 远程访问Tensorboard

一般来说我们都是在服务器上使用tensorflow的，那么能不能直接在服务器上运行tensorboard呢？

#### 步骤

1. ssh -L 16006:127.0.0.1:6006 account@server.address
2. tensorboard --logdir="tensorboard"
3. 在本地主机访问 <http://127.0.0.1:16006/>

#### 原理

建立ssh隧道，实现远程端口到本地端口的转发。
具体来说就是将远程服务器的6006端口（tensorboard默认将数据放在6006端口）转发到本地的16006端口，在本地对16006端口的访问即是对远程6006端口的访问，当然，转发到本地某一端口不是限定的，可自由选择。

