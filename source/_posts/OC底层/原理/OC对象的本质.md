---
title: OC对象的本质
date: 2020-05-06 14:36:30
tags: OC底层
---

## Objective-C的本质

* Objective-C代码的底层实现其实是C\C++代码
* Objective-C的面向对象是基于C\C++的数据结构(结构体)实现的

![ObjectiveC_C_C++_汇编语言_机器语言](OC对象的本质/ObjectiveC_C_C++_汇编语言_机器语言.png)

<!-- more -->

### 将Objective-C代码转换为C\C++代码

#### 创建一个项目
![OC对象的本质](OC对象的本质/OC对象的本质.png)

在终端打开 main.m 的位置，输入下面的命令生成 main.cpp 文件。因为要生成的代码包括c/c++，所以使用 main.cpp 文件，main.cpp 文件是 c++ 文件，支持 c/c++。

#### 生成 main.cpp
```
$ clang -rewrite-objc main.m -o main.cpp
```
没有指定平台，默认生成的是多个平台的代码，代码量太大。

#### 指定生成 iphoneos 平台、arm64 架构的 main.cpp  
指定平台，不同平台支持的代码不一样，如 Windows、mac、iOS。  
指定框架，不同框架支持的代码也不一样，模拟器(i386)、32bit(armv7)、64bit（arm64）。
```
$ xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m -o main-arm64.cpp
```
xcrun -sdk iphoneos：指定 iphoneos。
-arch arm64：指定 arm64 架构。

报错：xcrun: error: SDK "iphoneos" cannot be located  
解决1：给Xcode命令行工具指定路径↓
```
$ sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer/
```
（如果需要链接其他框架，使用-framework参数。比如-framework UIKit。(未验证)）

#### 取消 Xcode 对 main-arm64.cpp 的文件编译  
删除 Build Phases -> Compile Sources -> main-arm64.cpp
![main-arm64](OC对象的本质/取消编译main_arm64_cpp.png)




## NSObject的底层实现

* 思考：一个OC对象在内存中是如何布局的？

### NSObject 在 OC 中的定义：
```
@interface NSObject <NSObject> {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wobjc-interface-ivars"
    Class isa  OBJC_ISA_AVAILABILITY;
#pragma clang diagnostic pop
}

//简化后：
@interface NSObject {
    Class isa;
}
```
### 在 c++ 中的定义：
```
struct NSObject_IMPL {
    Class isa; // 8个字节
};
```
Class 是指向结构体的指针：typedef struct objc_class *Class。IMPL 是 implementation 的简写。

### 打印 NSObject 实例对象的成员变量所占用的大小 >> 8
```
//1.导入头文件
#import <objc/runtime.h>

//2.打印，结果 8
NSLog(@"%zd", class_getInstanceSize([NSObject class]));
```

### 打印 obj 指针所指向内存的大小 >> 16
```
//1.导入头文件
#import <malloc/malloc.h>

//2.打印，结果 16
NSLog(@"%zd", malloc_size((__bridge const void *)obj));
```
### class_getInstanceSize
下载 runtime 源码 [objc4](https://opensource.apple.com/tarballs/objc4/)。打开源码搜索 class_getInstanceSize，找到 objc-class.mm 文件。
```
size_t class_getInstanceSize(Class cls)
{
    if (!cls) return 0;
    return cls->alignedInstanceSize();
}
```
Jump to Definition -> alignedInstanceSize：
```
// Class's ivar size rounded up to a pointer-size boundary.
uint32_t alignedInstanceSize() const {
    return word_align(unalignedInstanceSize());
}
```
通过翻译过来就是，class_getInstanceSize 内部根据成员变量的大小，四色五入得到 NSObject 实例对象的成员变量所占用的内存大小。

### alloc

alloc 的内部实现是 allocWithZone，在源码中搜索 allocWithZone：
```
id
_objc_rootAllocWithZone(Class cls, malloc_zone_t *zone)
{
    id obj;

    if (fastpath(!zone)) {
        obj = class_createInstance(cls, 0);
    } else {
        obj = class_createInstanceFromZone(cls, 0, zone);
    }

    if (slowpath(!obj)) obj = _objc_callBadAllocHandler(cls);
    return obj;
}
```
Jump to Definition -> class_createInstance：
```
id
class_createInstance(Class cls, size_t extraBytes)
{
    if (!cls) return nil;
    return _class_createInstanceFromZone(cls, extraBytes, nil);
}
```
Jump to Definition -> _class_createInstanceFromZone：
```
//创建 cls 类型的实例对象
static ALWAYS_INLINE id
_class_createInstanceFromZone(Class cls, size_t extraBytes, void *zone,
                              int construct_flags = OBJECT_CONSTRUCT_NONE,
                              bool cxxConstruct = true,
                              size_t *outAllocatedSize = nil)
{
    ASSERT(cls->isRealized());

    // Read class's info bits all at once for performance
    bool hasCxxCtor = cxxConstruct && cls->hasCxxCtor();
    bool hasCxxDtor = cls->hasCxxDtor();
    bool fast = cls->canAllocNonpointer();
    size_t size;

    size = cls->instanceSize(extraBytes);
    if (outAllocatedSize) *outAllocatedSize = size;

    id obj;
    if (zone) {
        obj = (id)malloc_zone_calloc((malloc_zone_t *)zone, 1, size);
    } else {
        obj = (id)calloc(1, size); //c语言分配内存的函数，分配空间：size
    }
    if (slowpath(!obj)) {
        if (construct_flags & OBJECT_CONSTRUCT_CALL_BADALLOC) {
            return _objc_callBadAllocHandler(cls);
        }
        return nil;
    }

    if (!zone && fast) {
        obj->initInstanceIsa(cls, hasCxxDtor);
    } else {
        // Use raw pointer isa on the assumption that they might be
        // doing something weird with the zone or RR.
        obj->initIsa(cls);
    }

    if (fastpath(!hasCxxCtor)) {
        return obj;
    }

    construct_flags |= OBJECT_CONSTRUCT_FREE_ONFAILURE;
    return object_cxxConstructFromClass(obj, cls, construct_flags);
}
```

Jump to Definition -> instanceSize
```
size_t instanceSize(size_t extraBytes) const {
    if (fastpath(cache.hasFastInstanceSize(extraBytes))) {
        return cache.fastInstanceSize(extraBytes);
    }

    size_t size = alignedInstanceSize() + extraBytes;
    // CF requires all objects be at least 16 bytes.
    if (size < 16) size = 16;
    return size;
}
```
可以看到，创建的实例对象的大小至少16个字节。至少16个字节 是 CoreFoundation 框架内部硬性规定的。

#### 总结：  
通过 class_getInstanceSize 获取到的是 NSObject 实例对象的成员变量所占用的内存大小，当前的 NSObject 内部只有一个成员变量 isa，大小8个字节。  
通过 malloc_size 获取到的事 obj 指针所指向的内存的大小。因为 alloc 内部的
```
NSObject *obj = [[NSObject alloc] init];
```
这句代码实际上是在内存中生成了一个结构体，结构体内有一个类型为 Class 的 isa 指针，大小 8 个字节。Class 是一个指向结构体的指针。

