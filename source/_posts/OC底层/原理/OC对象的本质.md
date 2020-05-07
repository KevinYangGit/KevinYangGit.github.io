---
title: OC对象的本质
date: 2020-05-06 14:36:30
tags: OC底层
---

## Objective-C的本质

* 我们平时编写的Objective-C代码，底层实现其实都是C\C++代码
* 所以Objective-C的面向对象都是基于C\C++的数据结构(结构体)实现的

![ObjectiveC_C_C++_汇编语言_机器语言](Images/ObjectiveC_C_C++_汇编语言_机器语言.png)


## 将Objective-C代码转换为C\C++代码

终端打开 main.m 位置，输入下面的命令生成 main.cpp 文件。因为要生成的代码包括c/c++，所以使用 main.cpp 文件，main.cpp 文件是 c++ 文件，支持 c。

### 创建一个项目
![OC对象的本质](Images/OC对象的本质.png)

### 生成 main.cpp
```
$ clang -rewrite-objc main.m -o main.cpp
```
### 生成 arm64 架构的 main.cpp  
```
$ xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m -o main-arm64.cpp
```
报错：xcrun: error: SDK "iphoneos" cannot be located  
解决：给Xcode命令行工具指定路径↓
```
$ sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer/
```

