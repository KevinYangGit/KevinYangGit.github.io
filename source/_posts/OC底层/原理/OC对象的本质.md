---
title: OC对象的本质
date: 2020-05-06 14:36:30
tags: OC底层
---

# Objective-C的本质

* Objective-C代码的底层实现其实是C\C++代码
* Objective-C的面向对象是基于C\C++的数据结构(结构体)实现的

![ObjectiveC_C_C++_汇编语言_机器语言](OC对象的本质/ObjectiveC_C_C++_汇编语言_机器语言.png)

<!-- more -->

## 将Objective-C代码转换为C\C++代码

### 创建一个项目
![OC对象的本质](OC对象的本质/OC对象的本质.png)

在终端打开 main.m 的位置，输入下面的命令生成 main.cpp 文件。因为要生成的代码包括c/c++，所以使用 main.cpp 文件，main.cpp 文件是 c++ 文件，支持 c/c++。

### 生成 main.cpp
```
$ clang -rewrite-objc main.m -o main.cpp
```
没有指定平台，默认生成的是多个平台的代码，代码量太大。

### 指定生成 iphoneos 平台、arm64 架构的 main.cpp  
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

### 取消 Xcode 对 main-arm64.cpp 的文件编译  
生成的 main-arm64.cpp 文件添加到项目后，运行会报错。main-arm64.cpp 是临时生成的，内部有一个 main 函数，没做适配。 

解决：删除 Build Phases -> Compile Sources -> main-arm64.cpp
![main-arm64](OC对象的本质/取消编译main_arm64_cpp.png)




# NSObject的底层实现

* 思考：一个OC对象在内存中是如何布局的？

## NSObject 在 OC 中的定义：
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

## 在 c++ 中的定义：
```
struct NSObject_IMPL {
    Class isa; // 8个字节
};
```
Class 是指向结构体的指针：typedef struct objc_class *Class。IMPL 是 implementation 的简写。结构体中只有一个成员变量，所以这个结构体在内存中占用的大小就是指针 isa 的大小。

![OC对象的本质02](OC对象的本质/OC对象的本质02.png)

## 打印 NSObject 实例对象的成员变量所占用的大小 >> 8
```
//1.导入头文件
#import <objc/runtime.h>

//2.打印，结果 8
NSLog(@"%zd", class_getInstanceSize([NSObject class]));
```

## 打印 obj 指针所指向内存的大小 >> 16
```
//1.导入头文件
#import <malloc/malloc.h>

//2.打印，结果 16
NSLog(@"%zd", malloc_size((__bridge const void *)obj));
```
__bridge 可以实现 Objective-C 与 C 语言变量 和 Objective-C 与 Core Foundation 对象之间的互相转换。  


## 窥视 class_getInstanceSize
下载 runtime 源码 [objc4](https://opensource.apple.com/tarballs/objc4/)。  
打开源码搜索 class_getInstanceSize，找到 objc-class.mm 文件中 class_getInstanceSize 的实现代码。
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
翻译过来就是，class_getInstanceSize 内部根据成员变量的大小，四色五入得到 NSObject 实例对象里成员变量所占用的内存大小。

## 窥视 alloc
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
//创建 cls 的实例对象
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

    size = cls->instanceSize(extraBytes); //分配空间
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

Jump to Definition -> instanceSize：
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
可以看到，创建的实例对象的大小至少16个字节。CoreFoundation 框架内部就是这么硬性规定的。

## 总结：  
```
NSObject *obj = [[NSObject alloc] init];
```
* 上面👆这句代码实际上是在内存中生成了一个 c 语言定义的结构体，结构体内有一个类型为 Class 的 isa 指针，结构体的大小 8 个字节。Class 是一个指向结构体的指针。

* alloc 方法让系统分配了16个字节给 NSObject 对象（可以通过 malloc_size 函数获取）  

* NSObject 对象内部只有一个成员变量，即指针 isa，所以只使用了8个字节的空间（64bit环境下，可以通过 class_getInstanceSize 函数获得）




---




## ps：
### 通过 Xcode 工具查看对象内存。  
打开 Debug -> Debug Workflow -> View Memory，在 Address 输入对象的地址。  
![OC对象的本质03](OC对象的本质/OC对象的本质03.png)


### 常用LLDB指令
#### print、p：打印
```
(lldb) print obj
(NSObject *) $0 = 0x000000010380ef00
(lldb) p obj
(NSObject *) $1 = 0x000000010380ef00
```

#### po：打印对象
```
(lldb) po obj
<NSObject: 0x10380ef00>
```

#### 格式  
x是16进制，f是浮点，d是10进制

#### 字节大小  
b：byte 1字节，h：half word 2字节  
w：word 4字节，g：giant word 8字节

#### 读取内存  
memory read/数量格式字节数  内存地址
```
(lldb) memory read 0x10380ef00
0x10380ef00: 41 81 8b 9a ff ff 1d 00 00 00 00 00 00 00 00 00  A...............
0x10380ef10: e0 ef 80 03 01 00 00 00 20 f2 80 03 01 00 00 00  ........ ....... 
```

x/数量格式字节数  内存地址
```
(lldb) x 0x10380ef00
0x10380ef00: 41 81 8b 9a ff ff 1d 00 00 00 00 00 00 00 00 00  A...............
0x10380ef10: e0 ef 80 03 01 00 00 00 20 f2 80 03 01 00 00 00  ........ .......
(lldb) x/3xg 0x10380ef00
0x10380ef00: 0x001dffff9a8b8141 0x0000000000000000
0x10380ef10: 0x000000010380efe0
(lldb) x/4xg 0x10380ef00
0x10380ef00: 0x001dffff9a8b8141 0x0000000000000000
0x10380ef10: 0x000000010380efe0 0x000000010380f220
(lldb) x/4xw 0x10380ef00
0x10380ef00: 0x9a8b8141 0x001dffff 0x00000000 0x00000000
(lldb) x/4dw 0x10380ef00
0x10380ef00: -1702133439
0x10380ef04: 1966079
0x10380ef08: 0
0x10380ef0c: 0
```

打印结果中， x/3xg 0x10380ef00 打印的 0x001dffff9a8b8141 0x0000000000000000 部分是属于 obj 的内存。x/4xw 0x10380ef00 打印的 0x10380ef00: 0x9a8b8141 0x001dffff 0x00000000 0x00000000 部分属于 obj 的内存。

#### 修改内存中的值  
memory  write  内存地址  数值  
将内存中的第6个字节改成06：
```
(lldb) po obj
<NSObject: 0x10380ef00>

(lldb) memory read 0x10380ef00
0x10380ef00: 41 81 8b 9a ff ff 1d 00 00 00 00 00 00 00 00 00  A...............
0x10380ef10: e0 ef 80 03 01 00 00 00 20 f2 80 03 01 00 00 00  ........ .......
(lldb) memory write 0x10380ef06 6
(lldb) x 0x10380ef00
0x10380ef00: 41 81 8b 9a ff ff 06 00 00 00 00 00 00 00 00 00  A...............
0x10380ef10: e0 ef 80 03 01 00 00 00 20 f2 80 03 01 00 00 00  ........ .......
(lldb) 
```

po obj 获取到对象地址 0x10380ef00，所以第6个字节的地址就是 0x10380ef06。通过 memory write 0x10380ef06 6，将 0x10380ef06 处的字节改为 6。上面👆 x 0x10380ef00 打印出的结果中可以看到，第 6 个字节成功被修改为 06。