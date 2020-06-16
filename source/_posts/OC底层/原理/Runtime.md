---
title: Runtime
date: 2020-06-12 18:27:54
tags: OC底层原理
---

思考：
* 讲一下 OC 的消息机制
* 消息转发机制流程
* 什么是 Runtime？平时项目中有用过么？
* Runtime 的具体应用
* 打印结果分别是什么？
```
@interface Person : NSObject
@end

@implementation Person
@end

@interface Student : Person
@end

@implementation Student
- (instancetype)init
{
    self = [super init];
    if (self) {
        NSLog(@"[self class] = %@", [self class]);
        NSLog(@"[super class] = %@", [super class]);
        NSLog(@"[self superclass] = %@", [self superclass]);
        NSLog(@"[super superclass] = %@", [super superclass]);
    }
    return self;
}
@end
```

```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        BOOL res1 = [[NSObject class] isKindOfClass:[NSObject class]];
        BOOL res2 = [[NSObject class] isMemberOfClass:[NSObject class]];
        BOOL res3 = [[Person class] isKindOfClass:[Person class]];
        BOOL res4 = [[Person class] isMemberOfClass:[Person class]];
        NSLog(@"%d %d %d %d", res1, res2, res3, res4);
    }
    return 0;
}
```

* 以下代码能不能执行成功？如果可以，打印结果是什么？
```
@interface Person : NSObject
@property (nonatomic, copy) NSString *name;
- (void)print;
@end

@implementation Person
- (void)print {
    NSLog(@"my name's %@", self.name);
}
@end

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    id cls = [Person class];
    void *obj = &cls;
    [(__bridge id)obj print];
}
@end
```

<!-- more -->

Objective-C 是一门动态性比较强的编程语言，跟 C、C++ 等语言有着很大的不同，Objective-C 的动态性是由 Runtime API 来支撑的，Runtime API 提供的接口基本都是 C 语言的，源码由 C\C++\ 汇编语言编写。

# isa 详解
学习 Runtime，首先要了解它底层的一些常用数据结构，比如 isa 指针。在 arm64 架构之前，isa 就是一个普通的指针，存储着 Class、Meta-Class 对象的内存地址。从 arm64 架构开始，对 isa 进行了优化，变成了一个共用体（union）结构，还使用位域来存储更多的信息。

## 位运算
```
@interface Person : NSObject
@property (nonatomic, assign, getter=isTall) BOOL tall;
@property (nonatomic, assign, getter=isRich) BOOL rich;
@property (nonatomic, assign, getter=isHandsome) BOOL handsome;
@end

@implementation Person
@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Person *person = [[Person alloc] init];
        person.tall = YES;
        person.rich = YES;
        person.handsome = YES;
        
        NSLog(@"tall：%d, rich：%d, handsome：%d", person.isTall, person.isRich, person.isHandsome);
        NSLog(@"person的大小：%zd", class_getInstanceSize([person class]));
    }
    return 0;
}
```

打印结果：
```
tall：1, rich：1, handsome：1
person的大小：16
```

tall（1个字节）+ rich（1个字节）+ handsome（1个字节）+ isa（8个字节）= 11个字节。根据内存对齐原则，person 的内存大小是16个字节。

因为 tall、rich 和 handsome 都是 BOOL 类型，它们的值只有0和1，所以可以用3个二进制位来存储他们的值。

### 设计  
定义一个 char 类型的变量 _tallRichHandsome 用来存储3个 BOOL 类型变量的值：
```
@interface Person()
{
    char _tallRichHandsome; //ob 0000 0000
}
@end
```

_tallRichHandsome 占1个字节（8位：0b 0000 0000），让它最右边的3位（0b00000111）分别存储 tall、rich 和 handsome：
```
0b 0000 0111 //_tallRichHandsome（tall：YES, rich：YES, handsome：YES）

ob 0000 0001 //tall
ob 0000 0010 //rich
ob 0000 0100 //handsome
```

### 取值
* 与运算（&）  
运算规则：0&0=0，0&1=0，1&0=0，1&1=1。即：两位同时为1时，结果为1，否则为0。  

因为与运算（&）可以获取到特定位的值，所以可以通过与运算（&）分别获取三个变量的值：  
初始化 _tallRichHandsome
```
_tallRichHandsome = 0b00000101; //（tall：YES, rich：NO, handsome：YES）
```

获取 tall
```
  0b00000101
& 0b00000001
-------------
  0b00000001
```

获取 rich
```
  0b00000101
& 0b00000010
-------------
  0b00000000
```

获取 handsome
```
  0b00000101
& 0b00000100
-------------
  0b00000100
```

代码实现：
```
@interface Person : NSObject
- (BOOL)isTall;
- (BOOL)isRich;
- (BOOL)isHandsome;
@end

@interface Person()
{
    char _tallRichHandsome;
}
@end

@implementation Person

- (instancetype)init
{
    self = [super init];
    if (self) {
        _tallRichHandsome = 0b00000111;
    }
    return self;
}

- (BOOL)isTall {
    return !!(_tallRichHandsome & 1); //ob 0000 0001（二进制） == 1（十进制）
}

- (BOOL)isRich {
    return !!(_tallRichHandsome & 2); //ob 0000 0010（二进制） == 2（十进制）
}

- (BOOL)isHandsome {
    return !!(_tallRichHandsome & 4); //ob 0000 0100（二进制） == 4（十进制）
}
@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Person *person = [[Person alloc] init];
        NSLog(@"tall：%d, rich：%d, handsome：%d", person.isTall, person.isRich, person.isHandsome);
    }
    return 0;
}
```

打印结果：
```
tall：1, rich：1, handsome：1
```

因为返回的是 BOOL 类型，与运算取出的是有值（ob00000001、ob00000010、ob00000100）和0（ob00000000），所以可以在与运算的结果前加“!!”取反两次：
```
!(ob 0000 0000)  YES
!!(ob 0000 0000)  NO

!(ob 0000 0001)  NO
!!(ob 0000 0001)  YES
```

### 掩码
上面的实现，在取 tall 时 &1，取 rich 时 &2，取 handsome 时 &4，这样太抽象，可以使用掩码增加可读性：
```
#define TallMask 1
#define RichMask 2
#define HandsomeMask 4
```

直接使用二进制定义掩码会更直观：
```
#define TallMask ob00000001
#define RichMask ob00000010
#define HandsomeMask ob00000100
```

使用位移运算符，简化代码：
```
#define TallMask (1<<0)     //左移0位：ob00000001（二进制），1（十进制）
#define RichMask (1<<1)     //左移1位：ob00000010（二进制），2（十进制）
#define HandsomeMask (1<<2) //左移2位：ob00000100（二进制），4（十进制）
```

最终实现：
```
#define TallMask (1<<0)
#define RichMask (1<<1)
#define HandsomeMask (1<<2)

@implementation Person
- (BOOL)isTall {
    return !!(_tallRichHandsome & TallMask);
}

- (BOOL)isRich {
    return !!(_tallRichHandsome & RichMask);
}

- (BOOL)isHandsome {
    return !!(_tallRichHandsome & HandsomeMask);
}
@end
```

### 设置
* 或运算（|）  
运算规则：0|0=0，0|1=1，1|0=1，1|1=1。即：参加运算的两个对象只要有一个为1，其结果就为1。

因为或运算（|）可以设置特定位的值，所以可以通过或运算（|）分别设置三个变量的值：   
初始化 _tallRichHandsome
```
_tallRichHandsome = 0b00000010; //（tall：NO, rich：YES, handsome：NO）
```

设置 tall 为 YES
```
  0b00000010
| 0b00000001
-------------
  0b00000011
```

设置 rich 为 NO
```
  0b00000010
& 0b11111101  //~0b00000010
-------------
  0b00000000
```

设置 handsome 为 YES
```
  0b00000010
| 0b00000100
-------------
  0b00000110
```

设置 YES 时，和 _tallRichHandsome 进行按位或运算，修改特定位置的值。  
设置 NO 操时，先对 rich 的二进制数按位取反，再和 _tallRichHandsome 进行按位与运算。


```
@interface Person : NSObject
- (void)setTall:(BOOL)tall;
- (void)setRich:(BOOL)rich;
- (void)setHandsome:(BOOL)handsome;
- (BOOL)isTall;
- (BOOL)isRich;
- (BOOL)isHandsome;
@end

#define TallMask (1<<0)
#define RichMask (1<<1)
#define HandsomeMask (1<<2)

@interface Person()
{
    char _tallRichHandsome;
}
@end

@implementation Person

- (void)setTall:(BOOL)tall {
    if (tall) {
        _tallRichHandsome |= TallMask;
    } else {
        _tallRichHandsome &= ~TallMask;
    }
}

- (void)setRich:(BOOL)rich {
    if (rich) {
        _tallRichHandsome |= RichMask;
    } else {
        _tallRichHandsome &= ~RichMask;
    }
}

- (void)setHandsome:(BOOL)handsome {
    if (handsome) {
        _tallRichHandsome |= HandsomeMask;
    } else {
        _tallRichHandsome &= ~HandsomeMask;
    }
}

- (BOOL)isTall {
    return !!(_tallRichHandsome & TallMask);
}

- (BOOL)isRich {
    return !!(_tallRichHandsome & RichMask);
}

- (BOOL)isHandsome {
    return !!(_tallRichHandsome & HandsomeMask);
}
@end
```

## 位域


## 共用体（union）

在源码 [objc4-781](https://opensource.apple.com/tarballs/objc4/) 中查找 isa 的定义。  

找到 OC 对象的结构体 objc_object：
```
struct objc_object {
private:
    isa_t isa;
    
    ··· //省略一堆方法
}
```

可以看到 isa 是一个 isa_t 类型的变量，Jump To Definition -> isa_t：
```
union isa_t {
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }

    Class cls;
    uintptr_t bits;
#if defined(ISA_BITFIELD)
    //位域
    struct { 
        ISA_BITFIELD;  // defined in isa.h
    };
#endif
};
```

位域中是一个宏 ISA_BITFIELD：
```
# if __arm64__ //真机上市 arm64
#   define ISA_MASK        0x0000000ffffffff8ULL
#   define ISA_MAGIC_MASK  0x000003f000000001ULL
#   define ISA_MAGIC_VALUE 0x000001a000000001ULL
#   define ISA_BITFIELD                                                      \
      uintptr_t nonpointer        : 1;                                       \
      uintptr_t has_assoc         : 1;                                       \
      uintptr_t has_cxx_dtor      : 1;                                       \
      uintptr_t shiftcls          : 33; /*MACH_VM_MAX_ADDRESS 0x1000000000*/ \
      uintptr_t magic             : 6;                                       \
      uintptr_t weakly_referenced : 1;                                       \
      uintptr_t deallocating      : 1;                                       \
      uintptr_t has_sidetable_rc  : 1;                                       \
      uintptr_t extra_rc          : 19
#   define RC_ONE   (1ULL<<45)
#   define RC_HALF  (1ULL<<18)

# elif __x86_64__ //模拟器是 x86 架构
#   define ISA_MASK        0x00007ffffffffff8ULL
#   define ISA_MAGIC_MASK  0x001f800000000001ULL
#   define ISA_MAGIC_VALUE 0x001d800000000001ULL
#   define ISA_BITFIELD                                                        \
      uintptr_t nonpointer        : 1;                                         \
      uintptr_t has_assoc         : 1;                                         \
      uintptr_t has_cxx_dtor      : 1;                                         \
      uintptr_t shiftcls          : 44; /*MACH_VM_MAX_ADDRESS 0x7fffffe00000*/ \
      uintptr_t magic             : 6;                                         \
      uintptr_t weakly_referenced : 1;                                         \
      uintptr_t deallocating      : 1;                                         \
      uintptr_t has_sidetable_rc  : 1;                                         \
      uintptr_t extra_rc          : 8
#   define RC_ONE   (1ULL<<56)
#   define RC_HALF  (1ULL<<7)

# else
#   error unknown architecture for packed isa
# endif
```

可以看到 ISA_BITFIELD 在 `__arm64__`（真机） 和 `__x86_64__`（mac电脑/模拟器） 架构有不同的定义。

将宏 ISA_BITFIELD 替换掉，保留真机（arm64）代码，可以看到一个比较完整的 isa_t：
```
union isa_t {
    isa_t() { }
    isa_t(uintptr_t value) : bits(value) { }
    Class cls;
    uintptr_t bits;
    struct { 
        uintptr_t nonpointer        : 1;                                       \
        uintptr_t has_assoc         : 1;                                       \
        uintptr_t has_cxx_dtor      : 1;                                       \
        uintptr_t shiftcls          : 33; /*MACH_VM_MAX_ADDRESS 0x1000000000*/ \
        uintptr_t magic             : 6;                                       \
        uintptr_t weakly_referenced : 1;                                       \
        uintptr_t deallocating      : 1;                                       \
        uintptr_t has_sidetable_rc  : 1;                                       \
        uintptr_t extra_rc          : 19
    };
```

* nonpointer  
0，代表普通的指针，存储着Class、Meta-Class对象的内存地址  
1，代表优化过，使用位域存储更多的信息

* has_assoc  
是否有设置过关联对象，如果没有，释放时会更快

* has_cxx_dtor  
是否有C++的析构函数（.cxx_destruct），如果没有，释放时会更快

* shiftcls  
存储着Class、Meta-Class对象的内存地址信息

* magic  
用于在调试时分辨对象是否未完成初始化

* weakly_referenced  
是否有被弱引用指向过，如果没有，释放时会更快

* deallocating  
对象是否正在释放

* extra_rc  
里面存储的值是引用计数器减1

* has_sidetable_rc  
引用计数器是否过大无法存储在 isa 中，如果为1，那么引用计数会存储在一个叫 SideTable 的类的属性中