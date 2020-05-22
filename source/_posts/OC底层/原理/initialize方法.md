---
title: +initialize方法
date: 2020-05-22 15:20:48
tags: OC底层原理
---

* +initialize 方法会在类第一次接收到消息时调用

<!-- more -->

# 定义 Persion、Student 及其分类
```
@interface Person : NSObject
@end

@implementation Person
+ (void)initialize
{
    NSLog(@"Person +initialize");
}
@end

@interface Person (Test1)
@end

@implementation Person (Test1)
+ (void)initialize
{
    NSLog(@"Person (Test1) +initialize");
}
@end

@interface Person (Test2)
@end

@implementation Person (Test2)
+ (void)initialize
{
    NSLog(@"Person (Test2) +initialize");
}
@end

@interface Student : Persion
@end

@implementation Student
+ (void)initialize
{
    NSLog(@"Student +initialize");
}
@end

@interface Student (Test1)
@end

@implementation Student (Test1)
+ (void)initialize
{
    NSLog(@"Student (Test1) +initialize");
}
@end

@interface Student (Test2)
@end

@implementation Student (Test2)
+ (void)initialize
{
    NSLog(@"Student (Test2) +initialize");
}
@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSLog(@"main");
        [Persion alloc];
        [Student alloc];
    }
    return 0;
}
```

编译顺序：  
![load的实现原理02](load的实现原理/load的实现原理02.png)

打印结果：
```
Persion (Test1) +initialize
Student (Test2) +initialize
```

# objc4源码解读过程
* objc-msg-arm64.s  
objc_msgSend  

* objc-runtime-new.mm  
class_getInstanceMethod  
lookUpImpOrNil  
lookUpImpOrForward  
_class_initialize  
callInitialize  
objc_msgSend(cls, SEL_initialize)

[Persion alloc] 的本质是：
```
objc_msgSend(objc_getClass("Persion"), sel_registerName("alloc"))
```

[Student alloc] 的本质是：  
```
 objc_msgSend(objc_getClass("Student"), sel_registerName("alloc"))
```

# objc_msgSend()
打开 runtime 源码 [objc4-781](https://opensource.apple.com/tarballs/objc4/)，搜索 objc_msgSend()，会发现 objc_msgSend() 方法的源码是汇编语言。
 
objc_msgSend() - 类方法调用流程：  
isa -> 类对象/元类对象，寻找方法，调用  
superclass -> 类对象/元类对象，寻找方法，调用  
superclass -> 类对象/元类对象，寻找方法，调用

从“寻找方法”入手👇

# class_getClassMethod

class_getClassMethod 是获取类方法的函数，内部调用的是 class_getInstanceMethod 获取对象方法的函数。
```
Method class_getClassMethod(Class cls, SEL sel)
{
    if (!cls  ||  !sel) return nil;

    return class_getInstanceMethod(cls->getMeta(), sel);
}
```

## class_getInstanceMethod
Jump To Definition -> class_getInstanceMethod
```
Method class_getInstanceMethod(Class cls, SEL sel)
{
    if (!cls  ||  !sel) return nil;

    // This deliberately avoids +initialize because it historically did so.

    // This implementation is a bit weird because it's the only place that 
    // wants a Method instead of an IMP.

#warning fixme build and search caches
        
    // Search method lists, try method resolver, etc.
    lookUpImpOrForward(nil, sel, cls, LOOKUP_RESOLVER);

#warning fixme build and search caches

    return _class_getMethod(cls, sel);
}
```

## lookUpImpOrForward

在 class_getInstanceMethod 方法中，调用 lookUpImpOrForward(nil, sel, cls, LOOKUP_RESOLVER) 方法。所以 behavior == LOOKUP_RESOLVER

Jump To Definition -> lookUpImpOrForward
```
IMP lookUpImpOrForward(id inst, SEL sel, Class cls, int behavior)
{
    ...
    ...
    ...//一堆方法

    // Check for +initialize
    if ((behavior & LOOKUP_INITIALIZE)  &&  !cls->isInitialized()) { //如果类还没有实现 +initialized 方法
        initializeNonMetaClass (_class_getNonMetaClass(cls, inst));
        // If sel == initialize, initializeNonMetaClass will send +initialize 
        // and then the messenger will send +initialize again after this 
        // procedure finishes. Of course, if this is not being called 
        // from the messenger then it won't happen. 2778172
    }

    ...
    ...
    ...//一堆方法
}
```

## initializeNonMetaClass

initializeNonMetaClass 方法使用递归的方式优先将传入的类 cls 的父类调用 initializeNonMetaClass 方法进行处理。

Jump To Definition -> initializeNonMetaClass
```
void initializeNonMetaClass(Class cls)
{
    ASSERT(!cls->isMetaClass());

    Class supercls;
    bool reallyInitialize = NO;

    // Make sure super is done initializing BEFORE beginning to initialize cls.
    // See note about deadlock above.
    supercls = cls->superclass; //获取传入的类 cls 的父类
    if (supercls  &&  !supercls->isInitialized()) { //父类存在 && 父类没有 +initialize
        initializeNonMetaClass(supercls); //递归，父类优先 +initialize
    }
    
    ...
    ...
    ...

#if __OBJC2__
        @try
#endif
        {
            callInitialize(cls); //

            if (PrintInitializing) {
                _objc_inform("INITIALIZE: thread %p: finished +[%s initialize]",
                             objc_thread_self(), cls->nameForLogging());
            }
        }
        ...
        ...
        ...
}
```

## callInitialize

向传入的类 cls 发送“initialize”消息。至此实现了 +initialize 方法的调用。

Jump To Definition -> callInitialize
```
void callInitialize(Class cls)
{
    ((void(*)(Class, SEL))objc_msgSend)(cls, @selector(initialize));
    asm("");
}
```