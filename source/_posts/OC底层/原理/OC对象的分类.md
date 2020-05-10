---
title: OC对象的分类
date: 2020-05-09 16:37:02
tags: OC底层
---

Objective-C中的对象，简称OC对象，主要可以分为3种：  
* instance对象（实例对象）
* class对象（类对象）
* meta-class对象（元类对象）  

<!-- more -->

# instance 对象
instance 对象就是通过类 alloc 出来的对象，每次调用 alloc 都会产生新的 instance 对象。
```
NSObject *object1 = [[NSObject alloc] init];
NSObject *object2 = [[NSObject alloc] init];
```

object1、object2 是 NSObject 的 instance 对象，它们是不同的两个对象，分别占据着两块不同的内存。  

## instance 对象在内存中存储的信息

* isa指针
* 其他成员变量

定义 Person
```
@interface Person : NSObject
{
    @public
    int _age;
}
@end

@implementation Person
@end
```

创建 Person 的实例对象
```
Person *p1 = [[Person alloc] init];
Person *p2 = [[Person alloc] init];
```

p1、p2 对象在内存中存储的信息
![OC对象的分类](OC对象的分类/OC对象的分类01.png)


# class 对象
每个类在内存中有且只有一个class对象，同一个类 alloc 出来的实例对象共同拥有唯一的 class 对象。

获取 class 对象：
```
Class objectClass1 = [object1 class];
Class objectClass2 = [object2 class];
Class objectClass3 = object_getClass(object1);
Class objectClass4 = object_getClass(object2);
Class objectClass5 = [NSObject class];
```

objectClass1 ~ objectClass5 都是 NSObject 的 class 对象，它们是同一个对象。

## class 对象在内存中存储的信息主要包括
* isa指针
* superclass指针
* 类的属性信息（@property）、类的对象方法信息（instance method）
* 类的协议信息（protocol）、类的成员变量信息（ivar）  
......  

![OC对象的分类](OC对象的分类/OC对象的分类02.png)

不同的 instance 对象却拥有相同的属性、对象方法、协议和成员变量等等，这些信息都存放在 class 对象的内存中，保证了同样的信息只存储一份。

## 注意
以下代码获取的 objectClass 是 class 对象，并不是 meta-class 对象
```
Class objectClass = [[NSObject class] class];
```

# meta-class 对象

每个类在内存中有且只有一个meta-class对象。  

将类对象当做参数传入，获得元类对象：
```
Class objectMetaClass = object_getClass(objectClass5);
```

## meta-class 对象在内存中存储的信息主要包括
* isa指针
* superclass指针
* 类的类方法信息（class method）  
......

![OC对象的分类](OC对象的分类/OC对象的分类03.png)

## 查看 objecClass 是否为 meta-class
```
#import <objc/runtime.h>

BOOL result = class_isMetaClass(objecClass)
```