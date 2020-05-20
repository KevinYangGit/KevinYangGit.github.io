---
title: +load 和 +initialize
date: 2020-05-20 10:41:21
tags: OC底层原理
---

# +load 方法
* +load 方法会在 runtime 加载类、分类时调用；

* 每个类、分类的 +load 方法，在程序运行过程中只调用一次；

