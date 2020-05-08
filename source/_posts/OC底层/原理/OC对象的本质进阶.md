---
title: OC对象的本质进阶.md
date: 2020-05-08 18:25:55
tags:
---

# Student的本质
定义一个继承 NSObject 的类 Student：
```
@interface Student : NSObject {
    @public
    int _no;
    int _age;
}
@end

@implementation Student

@end
```

将 OC 代码转换为 C\C++ 代码，并在生成的 C/C++ 代码中找到 Student 的实现：
```
struct Student_IMPL {
    struct NSObject_IMPL NSObject_IVARS;
    int _no;
    int _age;
};
```

因为 NSObject_IMPL 内部只有一个成员变量指针 isa，所以上面👆的代码可以写成：
```
struct Student_IMPL {
    Class isa;
    int _no;
    int _age;
};
```
isa（8字节）+ _no（4字节）+ _age（4字节）= Student_IMPL（16字节）。  

根据地址也可以看出成员变量的大小：  
![OC对象的本质进阶01](OC对象的本质进阶/OC对象的本质进阶01.png)  
