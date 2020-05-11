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

# isa

## instance 对象、class 对象 和 meta-class 对象之间的 isa 关系
![isa和superclass](isa和superclass/isa和superclass02.png)

## 定义 Person
```
@interface Person : NSObject <NSCopying>
{
    @public
    int _age;
}
@property (nonatomic, assign) int no;
- (void)personInstanceMethod;
+ (void)personClassMethod;
@end

@implementation Person
@end
```

创建 Person 的实例对象
```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        
        Person *person = [[Person alloc] init];
        
        person->_age = 10;

        [person personInstanceMethod];
        
        [Person personClassMethod];
        
    }
    return 0;
}
```

## 将 OC 代码转换为 C\C++ 代码
找到 main.m 所在文件，在终端输入：
```
$ xcrun -sdk iphoneos clang -arch arm64  -rewrite-objc main.m
```

没有通过 ‘-o’ 生成指定文件时，默认生成 main.cpp 文件，打开 main.cpp 文件。找到 main 函数：
```
int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 

        Person *person = ((Person *(*)(id, SEL))(void *)objc_msgSend)((id)((Person *(*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("Person"), sel_registerName("alloc")), sel_registerName("init"));

        (*(int *)((char *)person + OBJC_IVAR_$_Person$_age)) = 10;

        ((void (*)(id, SEL))(void *)objc_msgSend)((id)person, sel_registerName("personInstanceMethod"));

        ((void (*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("Person"), sel_registerName("personClassMethod"));

    }
    return 0;
}
```

找到 objc_msgSend：
```
((void (*)(id, SEL))(void *)objc_msgSend)((id)person, sel_registerName("personInstanceMethod"));

((void (*)(id, SEL))(void *)objc_msgSend)((id)objc_getClass("Person"), sel_registerName("personClassMethod"));

//简化后：
objc_msgSend(person, sel_registerName("personInstanceMethod"));

objc_msgSend(objc_getClass("Person"), sel_registerName("personClassMethod"));
```

[person personInstanceMethod] 的具体实现是 objc_msgSend(person, sel_registerName("personInstanceMethod"))。  
即在实例对象 person 调用 -(void)personInstanceMethod 对象方法的时候，向实例对象 person 发送一条 "personInstanceMethod" 消息。  

[Person personClassMethod] 的具体实现是 objc_msgSend(objc_getClass("Person"), sel_registerName("personClassMethod"))。  
即在类对象 Person 调用 +(void)personClassMethod 类方法的时候，向类对象 Person 发送一条 "personClassMethod" 消息。  

## 方法调用与对象的关系
```
[person personInstanceMethod];
[Person personClassMethod];
```

上面👆两个方法调用表现出来的是，实例对象 person 可以调用存在 Person 类对象里的对象方法， Person 类对象可以调用存储在 Person 元类对象里的类方法。

## 小结
* instance 对象的 isa 指针指向 class 对象。当调用对象方法时，通过 instance 对象的 isa 指针找到 class 对象，最后找到对象方法的实现进行调用。

* class 对象的 isa 指针指向 meta-class 对象。当调用类方法时，通过 class 对象的 isa 指针找到 meta-class对象，最后找到类方法的实现进行调用。


# superclass

## 定义 Studen 继承自 Person
```
@interface Student : Person <NSCoding>
{
    @public
    int _weight;
}
@property (nonatomic, assign) int height;
- (void)studentInstanceMethod;
+ (void)studentClassMethod;
@end

@implementation Student
- (void)test
{
    
}
- (void)studentInstanceMethod
{
    
}
+ (void)studentClassMethod
{
    
}
- (id)initWithCoder:(NSCoder *)aDecoder
{
    return nil;
}
- (void)encodeWithCoder:(NSCoder *)aCoder
{
    
}
@end
```

创建 Student 的实例对象
```
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        
         Student *student = [[Student alloc] init];
         
         [student test];
         
         [student personInstanceMethod];
         
         [student init];
         
         [Student studentClassMethod];
         
         [Student personClassMethod];
         
         [Student load];
    }
    return 0;
}
```

## class 对象的 superclass 指针

### Student 类对象、Person 类对象 和 NSObject 类对象之间的 superclass 关系：
![isa和superclass](isa和superclass/isa和superclass03.png)

### Student 的实例对象调用父类 Person 里的对象方法：
```
[student personInstanceMethod];
```

对象方法 -(void)personInstanceMethod 方法保存在 Person 的类对象里，[student personInstanceMethod] 首先通过 student 的 isa 指针找到 Student 的类对象，再通过 Student 类对象里的 superclass 找到 Person 的类对象，最后在 Person 类对象里找到了对象方法 -(void)personInstanceMethod。

### Student 的实例对象调用父类 NSObject 里的对象方法：
```
[student init];
```

对象方法 -(void)init 方法保存在 NSObject 的类对象里，[student init] 首先通过 student 的 isa 指针找到 Student 的类对象，再通过 Student 类对象里的 superclass 找到 Person 的类对象，再通过 Person 类对象里的 superclass 找到 NSObject 的类对象，最后在 NSObject 类对象里找到了对象方法 -(void)init。

### 小结
* 具有继承关系的不同的类之间，是通过 superlass 指针连接的。有了 superlass 指针的连接，子类就实现了调用父类方法的逻辑。

* 当 Student 的 instance 对象要调用 Person 的对象方法时，会先通过 isa 找到 Student 的 class，然后通过 superclass 找到 Person 的 class，最后找到对象方法的实现进行调用。


## meta-class 对象的 superclass 指针

### Student 元类对象、Person 元类对象 和 NSObject 元类对象之间的 superclass 关系：
![isa和superclass](isa和superclass/isa和superclass04.png)

### Student 类对象调用 Student 元类对象里的类方法：
```
[Student studentClassMethod];
```

首先通过 Student 类对象里的 isa 指针找到 Student 元类对象，最终在 Student 元类对象里找到类方法 +(void)studentClassMethod。

### Student 类对象调用父类 Person 元类对象里的类方法：
```
[Student personClassMethod];
```

首先通过 Student 类对象里的 isa 指针找到 Student 元类对象，再通过 Student 元类对象里的 superclass 找到 Person 元类对象，最终在 Person 元类对象里找到类方法 +(void)personClassMethod。

### Student 类对象调用父类 NSObject 元类对象里的类方法：
```
[Student load];
```

首先通过 Student 类对象里的 isa 指针找到 Student 元类对象，再通过 Student 元类对象里的 superclass 找到 Person 元类对象，再通过 Person 元类对象里的 superclass 找到 NSObject 元类对象，最终在 NSObject 元类对象里找到类方法 +(void)load。