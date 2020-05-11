---
title: isa和superclass
date: 2020-05-11 14:35:58
tags: OC底层
---

![isa和superclass](isa和superclass/isa和superclass01.png)

<!-- more -->

* instance 的 isa 指向 class
* class 的 isa 指向 meta-class
* meta-class 的 isa 指向基类的 meta-class
* class 的 superclass 指向父类的 class，如果没有父类，superclass 指针为nil
* meta-class 的 superclass 指向父类的 meta-class，基类的 meta-class 的 superclass 指向基类的 class
* instance 调用对象方法的轨迹：isa 找到 class，方法不存在，就通过 superclass 找父类
* class 调用类方法的轨迹：isa 找 meta-class，方法不存在，就通过 superclass 找父类

# isa指针
