---
title: Flutter之Dart
date: 2023-10-13 14:42:05
tags: Flutter
---

1. Dart 语言的入口是 `main` 函数，并且必须显式定义。Dart 语言的 `main` 函数没有返回值。传递给 `main` 函数的命令行参数，是通过 `List<String>` 完成的。`List` 是集合类型，每一个 `String` 都表示一个参数。
2. Dart 语言中打印信息使用 `print` 函数。
3. Dart 语言中定义字符串可以使用单引号或双引号。
4. Dart 语言中每行代码必须使用分号结尾。
5. 编写 Dart 语言创建的文件是 `.dart` 文件。

<!-- more -->

Dart源码：安装过 Flutter 后，可以在 `/Applications/flutter/bin/cache/dart-sdk/lib/core/` 找到 Dart 的源码。

## 环境搭建

![01](Flutter之Dart/01.png)

## 基本语法

1. `main` 函数是 Dart 入口；
2. Dart 中打印使用 `print`；

`main` 函数的返回值是空 `void`，在 Dart 中返回值可以省略，由系统自动进行类型判断。

```dart
void main(List<String> args) {
  print("hello dart");
}
```

### 声明变量

1、明确的声明

* 变量类型 变量名 = 赋值;

```dart
String name = "Tom";
```

2、类型推导

`var` 用于定义变量，`final`、`const` 用于定义常量。使用他们定义变量或常量时，虽然没有明确指定变量的类型，但是在编译时变量可以明确自己的类型。

使用 `var` 定义变量：

```dart
var age = 20;
age = 30
// age = "Tom" ❌
```

使用 `final` 定义常量：

```dart
final height = 180;
```

使用 `const` 定义常量：

```dart
const address = "beijing"
```

`final` 和 `const` 的区别：

1. `const` 在赋值时，赋值的内容必须在**编译期间**就确定下来。
2. `final` 在赋值时，可以动态获取，如通过计算、函数返回等方式获取一个值，在**运行期间**来确定一个值。

可以通过函数给 final 赋值：

```dart
final date1 = DateTime.now();
// const date2 = DateTime.now(); ❌
```

`final` 可以定义两个相同类型的常量。

```dart
void main(List<String> args) {
  final p1 = Person("Tom");
  final p2 = Person("Tom");
  print(identical(p1, p2));
  // Prints "false"
}

class Person {
  final String name;
  Person(this.name);
}
```

`const` 可以让一个类型只创建一个常量。

```dart
void main(List<String> args) {
  const p1 = const Person("Tom");
  const p2 = const Person("Tom");
  print(identical(p1, p2));
  // Prints "True"
}

class Person {
  final String name;
  const Person(this.name);
}
```

在日常开发中定义常量多使用 `final`。

可以使用 `runtimeType` 获取变量当前的类型：

```dart
var age = 20;
print(age.runtimeType);
// Prints "Int"
```

### 布尔类型

Dart 中不能判断非零即真，或者非空即真。Bool 类型必须是一个确定的实体。

```dart
void main(List<String> args) {
  var flag = "abc"
  // if (flag) { ❌
  //   print("hello dart");
  // }
  if (flag != null) {
    print("hellp dart");
  }
}
```

### 字符串类型

Dart 字符串是 UTF-16 编码单元的序列，可以使用单引号或双引号创建一个字符串：

```dart
void main(List<String> args) {
  var str1 = 'abc';
  var str2 = "abc";

  print(str1);
  // Prints "abc"
  print(str2);
  // prints "abc"
}
```

可以使用三个单引号或者双引号表示多行字符串

```dart
void main(List<String> args) {
  var str3 = """
abc
cba
nba
  """;
  var str4 = '''
  abc
  cba
  nba
  ''';

  print(str3);
  print(str4);
}
  
```

打印结果：

```js
abc
cba
nba
  
 abc
 cba
 nba
```

* 字符串的拼接

字符串和其他变量或表达式拼接时，使用`${expression}`表达式，如果 expression 是一个标识符，那么 `{}` 可以省略：

```dart
void main(List<String> args) {
  var name = "Tom";
  var age = 18;
  var height = 180;

  var desc1 = "name is ${name}, age is ${age}, height is ${height}。";
  var desc2 = "name is $name, age is $age, height is $height。";
  print(desc1);
  // Prints "name is Tom, age is 18, height is 180。"
  print(desc2);
  // Prints "name is Tom, age is 18, height is 180。"
}
```

如果不是一个标识符，则不可以省略 `{}`：

```dart
void main(List<String> args) {
  var name = "Tom";
  var age = 18;
  var height = 180;

  var desc3 = "name is $name.runtimeType, age is $age, height is $height。";
  var desc4 = "name is ${name.runtimeType}, age is $age, height is $height。";
  print(desc3);
  // Prints "name is Tom.runtimeType, age is 18, height is 180。"
  print(desc4);
  // Prints "name is String, age is 18, height is 180。"
}
```

数字转字符串：

```dart
void main(List<String> args) {
  int age = 18; // int

  double height = 1.88; // double

  var ageString = age.toString(); // 18 String

  var heightString = height.toString(); // 1.88 String
}
```

字符串转数字：

```dart
void main(List<String> args) {
  int age = 18; // int

  double height = 1.88; // double

  var ageString = age.toString(); // 18 String

  var heightString = height.toString(); // 1.88 String

  print(int.parse(ageString));
  // Prints "18"
  print(double.parse(heightString));
  // Priints "1.88"
}
```

### 集合

1、列表List：`[]`

```dart
void main(List<String> args) {
  var names = ["abc", "cba", "nba"];
  // 添加元素
  names.add("mba");
  print('$names ${names.runtimeType}');
  // Prints "[abc, cba, nba, mba] List<String>"

  // 指定类型
  List<String> stringNames = ["abc", "cba", "nba"];
  print('$stringNames ${stringNames.runtimeType}');
  // Prints "[abc, cba, nba] List<String>"
}
```

2、集合Set：`{}`

```dart
void main(List<String> args) {
  var numbers = {"1", "2", "3", "4", "5", "6", "1"};
  print('$numbers ${numbers.runtimeType}');
  // Prints "{1, 2, 3, 4, 5, 6} _Set<String>"

  // 指定类型
  Set<String> sixNumbers = {"1", "2", "3", "4", "5", "6", "1"};
  print('$sixNumbers ${sixNumbers.runtimeType}');
  // Prints "{1, 2, 3, 4, 5, 6} _Set<String>"
}
```

3、映射Map

```dart
void main(List<String> args) {
  var info = {"name": "Tom", "age": 18};
  print('$info ${info.runtimeType}');
  // Prints "{name: Tom, age: 18} _Map<String, Object>"

  // 指定类型
  Map<String, Object> userInfo = {"name": "Tom", "age": 18};
  print('$userInfo ${userInfo.runtimeType}');
  // Prints "{name: Tom, age: 18} _Map<String, Object>"
}
```

#### 增删改查

1、列表 List

```dart
void main(List<String> args) {
  var numbersList = [1, 2, 3, 4, 5, 6];

  numbersList.add(7);
  print(numbersList);
  // Prints "[1, 2, 3, 4, 5, 6, 7]"

  numbersList.remove(3);
  print(numbersList);
  // Prints "[1, 2, 4, 5, 6, 7]"

  print(numbersList.contains(6));
  // Prints "true"

  print(numbersList.removeAt(1));
  // Prints "2"
  print(numbersList);
  // Prints "[1, 4, 5, 6, 7]"
}
```

2、集合 Set

```dart
void main(List<String> args) {
  var numbersSet = {1, 2, 3, 4, 5, 6};

  numbersSet.add(7);
  print(numbersSet);
  // Prints "{1, 2, 3, 4, 5, 6, 7}"

  numbersSet.remove(3);
  print(numbersSet);
  // Prints "{1, 2, 4, 5, 6, 7}"

  print(numbersSet.contains(6));
  // Prints "true"
}
```

3、映射 Map

```dart
void main(List<String> args) {
  var infoMap = {"name": "Tom", "age": 18};
  print(infoMap['name']);
  // Prints "Tom"

  print('${infoMap.entries}');
  // Prints "(MapEntry(name: Tom), MapEntry(age: 18))"

  print('${infoMap.values} ${infoMap.values.runtimeType}');
  // Prints "(Tom, 18) _CompactIterable<Object>"

  print('${infoMap.keys} ${infoMap.keys.runtimeType}');
  // Prints "(name, age) _CompactIterable<String>"

  print('${infoMap.containsKey('age')}');
  // Prints "true"

  infoMap.remove('age');
  print(infoMap);
  // Prints "{name: Tom}"
}
```

## 函数

### 基本使用

```dart
void main(List<String> args) {
  print(sum(20, 30));
  // Prints "50"
}

// 参数：int，int
// 返回值：int
int sum(int num1, int num2) {
  return num1 + num2;
}
```

`sum` 函数的参数和返回值都是 `int` 类型，其中返回值类型可以省略，由 Dart 自动推断：

```dart
void main(List<String> args) {
  print(sum(20, 30));
  // Prints "50"
}

sum(int num1, int num2) {
  return num1 + num2;
}
```

* 如果函数中只有一个表达式，可以使用箭头语法（arrow syntax）`=>`

```dart
void main(List<String> args) {
  print(sum(20, 30));
  // Prints "50"
}

sum(int num1, int num2) => num1 + num2;
```

### 可选参数

在 Dart 中没有函数重载。参数类型分为三类：

1. 必选参数；
2. 位置可选参数；
3. 命名可选参数；

#### 必选参数

```dart
void main(List<String> args) {
  sayHello("Tom");
}

void sayHello(String name) {
  print(name);
  // Prints "Tom"
}
```

可选参数有两种：

1. 位置可选参数：[param1, param2, ...]；
2. 命名可选参数：{param1, param2, ...}；

#### 命名可选参数

* 命名可选参数：{param1, param2, ...}；

位置不确定，但是形参必须要写。

```dart
void main(List<String> args) {

  sayHello3("Tom");
  // Prints "name=Tom age=0 height=0.0"
  sayHello3("Tom", age: 18);
  // Prints "name=Tom age=18 height=0.0"
  sayHello3("Tom", height: 180);
  // Prints "name=Tom age=0 height=180.0"
  sayHello3("Tom", age: 18, height: 180);
  // Prints "name=Tom age=18 height=180.0"
}

void sayHello3(String name, {int age = 0, double height = 0}) {
  print('name=$age age=$age height=$height');
}
```

#### 位置可选参数

* 位置可选参数：[param1, param2, ...]；

不需要写名形参，实参和形参在进行匹配时，是根据位置的匹配。

```dart
void main(List<String> args) {
  sayHello2("Tom");
  // Prints "name=Tom age=0 height=0.0"
  sayHello2("Tom", 18);
  // Prints "name=Tom age=18 height=0.0"
  sayHello2("Tom", 18, 180);
  // Prints "name=Tom age=18 height=180.0"
}

void sayHello2(String name, [int age = 0, double height = 0]) {
  print('name=$age age=$age height=$height');
}
```

### 参数默认值

在不传参数的情况下，使用默认值。

```dart
void main(List<String> args) {

  sayHello3("Tom");
  // Prints "name=Tom age=0 height=0.0"
}

void sayHello3(String name, {int age = 0, double height = 0}) {
  print('name=$age age=$age height=$height');
}
```

### 函数是一等公民

函数作为一等公民，即可以赋值给一个变量，也可以作为另一个函数的参数或这返回值。

1、将函数赋值给一个变量：

```dart
void main(List<String> args) {
  var hello = sayHello;
  print(hello);
  // Prints "Closure: (String) => dynamic from Function 'sayHello': static."
}

// 1、定义一个函数
sayHello(String name) {
  print("hello $name");
}
```

可以看到 `hello` 是一个 `Closure: (String) => dynamic` 的类型的函数。

2、将函数作为另一个函数的参数：

```dart
void main(List<String> args) {
  tom(sayHello);
  // Prints "hello Tom"
}

// 1、定义一个函数
sayHello(String name) {
  print("hello $name");
}

// 2、将函数作为另一个函数的参数
tom(Function func) {
  func("Tom");
}
```

3、将函数作为另一个函数的返回值：

```dart
void main(List<String> args) {
  var persion = getPersion();
  persion(sayHello);
  // Prints "hello Tom"
}

// 1、定义一个函数
sayHello(String name) {
  print("hello $name");
}

// 2、将函数作为另一个函数的参数
tom(Function func) {
  func("Tom");
}

// 3、将函数作为另一个函数的返回值
getPersion() {
  return tom;
}
```

4、自定义函数类型

```dart
typedef Calculate = void Function(int num1, int num2);

void main(List<String> args) {
  sum((num1, num2) {
    print('$num1 + $num2 = ${num1 + num2}');
  });
}

void sum(Calculate calc) {
  calc(20, 30);
}
```

### 匿名函数

匿名闭包（anonymous function），也叫作 lambda 或 闭包（closure）。定义一个没有名字的函数，作为函数的参数或返回值，提高编码效率。

```dart
void main(List<String> args) {
  say((String text) {
    print(text);
  });
  // Prints "hello"
}

void say(Function func) {
  func("hello");
}
```

`say` 函数的返回值为 `void`，参数是一个函数，该函数的参数是 `String` 类型，函数体是一个打印方法：

```dart
(String text) {
  print(text);
}
```

示例二：使用 `forEach` 遍历

```dart
void main(List<String> args) {
  var array = [1, 2, 3, 4, 5, 6];
  // 1、匿名函数
  array.forEach((element) {
    print(element);
  });
  // array.forEach((element) => print(element));

  // 2、有名字的函数
  printElement(element) {
    print(element);
  }
  array.forEach(printElement);
}
```

打印结果：

```js
1
2
3
4
5
6
1
2
3
4
5
6
```

### 词法作用域

优先使用自己作用域中的变量，如果没有找到，则一层层向外查找。

```dart
var name = 'Tom';
void main(List<String> args) {
  void say() {
    print(name);
    // Prints "Tom"
  }

  say();
}
```

```dart
var name = 'Tom';
void main(List<String> args) {
  var name = 'Tom 1';
  void say() {
    print(name);
    // Prints "Tom 1"
  }

  say();
}
```

```dart
var name = 'Tom';
void main(List<String> args) {
  var name = 'Tom 1';
  void say() {
    var name = 'Tom 2';
    print(name);
    // Prints "Tom 2"
  }

  say();
}
```

### 词法闭包

```dart
void main(List<String> args) {
  createMaker(num addBy) {
    // 返回一个闭包
    return (num i) {
      return i + addBy;
    };
  }

  var adder = createMaker(1);
  print(adder(1));
  // Prints "2"
  print(adder(2));
  // Prints "3"
}
```

创建出来的 `adder` 记住了创建时传入的 2。

### 隐式返回

所有函数都有一个返回值。如果没有指定返回值，则默认返回 `null`。

```dart
void main(List<String> args) {
  print(method());
  // Prints "null"
}

method() {
  print('function');
}
```

## 特殊运算符

* `??` 空值合并运算符

```dart
void main(List<String> args) {
  var name = null;
  name = name ?? "Tom";
  print(name);
  // Prints "Tom"
}
```

* `??=` 空值赋值运算符

```dart
void main(List<String> args) {
  var name = null;
  name ??= 'Tom';
  print(name);
  // Prints "Tom"
}
```

* `..` 级联运算符

```dart
void main(List<String> args) {
  var p = Persion()
    ..name = 'Tom'
    ..eat()
    ..run();
}

class Persion {
  String name;

  Persion({this.name = ''});

  void run() {
    print('$name is running');
  }

  void eat() {
    print('$name is eating');
  }
}
```

打印信息：

```js
Tom is eating
Tom is running
```

* `for` 循环

```dart
void main(List<String> args) {
  for (var i = 0; i < 6; i++) {
    print(i);
  }

  var numbers = [1, 2, 3, 4, 5, 6];
  for (var i = 0; i < numbers.length; i++) {
    print(numbers[i]);
  }
}
```

* `for-in` 循环

```dart
void main(List<String> args) {
  var numbers = [1, 2, 3, 4, 5, 6];
  for (var number in numbers) {
    print(number);
  }
}
```

## Dart之面向对象

### 类的定义

```dart
void main(List<String> args) {
  // var p = new Person("Tom", 18);
  var p = Person("Tom", 18);
  print(p);
  // Prints "name is Tom, age is 18"
}

class Person {
  String name;
  int age;

  Person(this.name, this.age);

  @override
  String toString() {
    return 'name is $name, age is $age';
  }
}
```

1、初始化类时，类名前的 `new` 可以省略：

```dart
// var p = new Person("Tom", 18);
var p = Person("Tom", 18);
```

2、Dart 语言要求类的成员变量，在类初始化前要有确定的值。如果没有在初始化时给成员变量赋值，就会报错：

![02](Flutter之Dart/02.png)

### 类的构造函数

```dart
void main(List<String> args) {
  var p1 = Person('Tom', 18);
  var p2 = Person.custom('Jack', 18, 180);
  print(p1);
  // Prints "name is Tom, age is 18, height is 0.0"
  print(p2);
  // Prints "name is Jack, age is 18, height is 180.0"
}

class Person {
  String name;
  int age;
  double height = 0;

  Person(this.name, this.age);
  // 等同
  //Person(String name, int age) {
  //  this.name = name;
  //  this.age = age;
  //}

  Person.custom(this.name, this.age, this.height);

  @override
  String toString() {
    return 'name is $name, age is $age, height is $height';
  }
}
```

使用 json 初始化类：

```dart
void main(List<String> args) {
  var p3 = Person.fromMap({'name': 'Tom', 'age': 18, 'height': 180.0});
  print(p3);
  // Prints "name is Tom, age is 18, height is 180.0"
}

class Person {
  late String name;
  late int age;
  double height = 0;

  Person.fromMap(Map<String, dynamic> map) {
    this.name = map['name'];
    this.age = map['age'];
    this.height = map['height'];
  }

  @override
  String toString() {
    return 'name is $name, age is $age, height is $height';
  }
}
```

### 初始化列表

* 初始化列表会在构造函数前执行，常用于设置 `final` 变量的值，使用逗号`,`分隔初始化表达式。

```dart
void main(List<String> args) {
  var p = Person('Tom', age: 18);
  print(p);
  // Prints "name is Tom, age is 18"
}

class Person {
  final String name;
  final int age;

  Person(this.name, {int? age}) : this.age = age ?? 0 {}

  @override
  String toString() {
    return 'name is $name, age is $age';
  }
}
```

在创建对象时，可以选择传或不传 `age`，不传使用默认值：

```dart
void main(List<String> args) {
  var p = Person('Tom');
  print(p);
  // Prints "name is Tom, age is 0"
}
```

Dart 3.0 后，使用 `?` 表示 `age` 可能为 `null`，在可选参数列表中使用 `??` 判断 `age` 是否有值。

![05](Flutter之Dart/05.png)

`widget` 中 `container` 的相关源码（路径`/Applications/flutter/packages/flutter/lib/src/widgets/container.dart`）：

```dart
Container({
    super.key,
    this.alignment,
    this.padding,
    this.color,
    this.decoration,
    this.foregroundDecoration,
    double? width,
    double? height,
    BoxConstraints? constraints,
    this.margin,
    this.transform,
    this.transformAlignment,
    this.child,
    this.clipBehavior = Clip.none,
  }) : assert(margin == null || margin.isNonNegative),
       assert(padding == null || padding.isNonNegative),
       assert(decoration == null || decoration.debugAssertIsValid()),
       assert(constraints == null || constraints.debugAssertIsValid()),
       assert(decoration != null || clipBehavior == Clip.none),
       assert(color == null || decoration == null,
         'Cannot provide both a color and a decoration\n'
         'To provide both, use "decoration: BoxDecoration(color: color)".',
       ),
       constraints =
        (width != null || height != null)
          ? constraints?.tighten(width: width, height: height)
            ?? BoxConstraints.tightFor(width: width, height: height)
          : constraints;
```

### 重定向构造方法

因为 Dart 不支持方法重载，所以对于有多个成员变量的类，不能写多个不同的构造方法：

![06](Flutter之Dart/06.png)

可以使用**构造函数重定向**来实现这种场景：

```dart
void main(List<String> args) {
  var p = Person('name');
  print(p.age);
  // Prints "0"
}

class Person {
  String name;
  int age;

  Person(String name) : this._internal(name, 0);

  Person._internal(this.name, this.age);
}
```

### 工厂构造方法

普通的构造函数会自动返回创建出来的对象，不能手动返回：

```dart
void main(List<String> args) {
  final p3 = Person('Jack', 'brown');
  final p4 = Person('Jack', 'brown');
  print(identical(p3, p4));
  // Pritns "false"
}

class Person {
  String name;
  String color;

  Person(this.name, this.color);
}
```

工厂构造函数最大的特点是**可以手动返回一个对象**：

```dart
void main(List<String> args) {
  final p1 = Person.withName('Tom');
  final p2 = Person.withName('Tom');
  print(identical(p1, p2));
  // Prints "true"
}

class Person {
  String name;
  String color;

  Person(this.name, this.color);

  static final Map<String, Person> _nameCache = {};
  static final Map<String, Person> _colorCache = {};

  factory Person.withName(String name) {
    if (_nameCache.containsKey(name)) {
      return _nameCache[name]!;
    } else {
      final p = Person(name, 'default');
      _nameCache[name] = p;
      return p;
    }
  }

  factory Person.withColor(String color) {
    if (_colorCache.containsKey(color)) {
      return _colorCache[color]!;
    } else {
      final p = Person('default', color);
      _colorCache[color] = p;
      return p;
    }
  }
}
```

📢注意：这里的 `_nameCache[name]!` 加了 `!`，表明一定不为 `null`。这是 Dart 3.0 增加的能力-[空安全类型系统](https://dart.cn/codelabs/null-safety)。

### setter 和 getter

```dart
void main(List<String> args) {
  final p = Person();

  p.name = 'Tom';
  print(p.name);
  // Prints "Tom"

  p.setName = 'Jack';
  print(p.name);
  // Prints "Jack"
  print(p.getName);
  // Prints "Jack"
}

class Person {
  String name;

  Person({String? name}) : this.name = name ?? "";

  // Setter
  set setName(String name) => this.name = name;
  // Getter
  String get getName => name;
}
```

### 类的继承

1. 使用 `extends` 表示继承关系；
2. 子类的构造方法在执行前，将隐含调用父类的无参默认构造方法（没有参数且与类同名的构造方法）。
3. 如果父类没有无参默认构造方法，则子类的构造方法必须在初始化列表中通过`super`显式调用父类的某个构造方法。

```dart
void main(List<String> args) {
  var t = Teacher(18, 'Tom');
  print('name is ${t.name}, age is ${t.age}');
  // Prints "name is Tom, age is 18"
}

class Person {
  String name;
  Person(this.name);
}

class Teacher extends Person {
  int age;

  Teacher(this.age, String name) : super(name);
}
```



### Object 和 dynamic 的区别

Dart 中所有类型都是 Object 的子类，包括 Function 和 Null。

* dynamic 与 Object 声明的变量都可以赋值任意对象，且后期可以改变赋值的类型。

![03](Flutter之Dart/03.png)

* Object 对象使用的属性和方法若不存在，**编译时会报错**。

* dynamic 对象使用的属性和方法若不存在，**编译时不会报错，运行时会报错**。

![04](Flutter之Dart/04.png)

### 抽象类的使用

📢注意一：继承自抽象类后，必须实现抽象类中没有实现的方法。

![07](Flutter之Dart/07.png)

📢注意二：抽象类不能实例化。

![08](Flutter之Dart/08.png)

必须实现抽象类中没有实现的方法：

```dart
void main(List<String> args) {
  final t = Teacher();
  print('name is ${t.getName()}, age is ${t.getAge()}');
  // Prints "name is 名字, age is 18"
}

abstract class Person {
  int getAge();
  String getName() {
    return '名字';
  }
}

class Teacher extends Person {
  @override
  int getAge() {
    return 18;
  }
}
```

可以重写抽象类中有实现的方法：

```dart
void main(List<String> args) {
  final t = Teacher();
  print('name is ${t.getName()}, age is ${t.getAge()}');
  // Prints "name is Tom, age is 18"
}

abstract class Person {
  int getAge();
  String getName() {
    return '名字';
  }
}

class Teacher extends Person {
  @override
  int getAge() {
    return 18;
  }

  @override
  String getName() {
    return 'Tom';
  }
}
```

### 隐式接口

Dart 默认情况下所有的类都是隐式接口。在将一个类作为接口使用时，实现接口的类，必须实现这个接口中所有方法。通常将用于给别人实现的类声明为抽象类。

定义一个 `Tom` 类，继承自 `Person`，使用 `Run`、`Swim` 类作为接口：

```dart
void main(List<String> args) {
  var t = Tom();
  t.running();
}

abstract class Run {
  void running() {}
}

abstract class Swim {
  void swimming() {}
}

class Person {
  String name;
  int age;

  Person(this.name, this.age);
}

class Tom extends Person implements Run, Swim {
  Tom() : super('Tom', 18);

  @override
  void running() {
    print('Tom is good at running');
  }

  @override
  void swimming() {
    print('Tom is good at swimming');
  }
}
```

![09](Flutter之Dart/09.png)

通过 `implements` 实现某个类时，必须重新实现类定义的所有方法(无论这个类是否已经对方法实现)。如果想要直接复用之前类的原有实现方案，则可以使用 `mixin`混入。

### mixin混入的使用

想要直接复用之前类的原有实现方案，一般采取继承。但是因为 Dart 只支持单继承，所以无法直接复用多个类原有的实现方案。

想要**直接复用多个类原有的实现方案**，可以采用 Mixin 混入的方式：

1. 使用 `mixin` 关键字定义一个类；
2. 使用 `with` 关键字来实现混入；

```dart
void main(List<String> args) {
  var t = Tom();
  t.running();
  // Prints "running"
  t.swimming();
  // Prints "swimming"
}

/**
1、使用 mixin 关键字定义类
*/
mixin Run {
  void running() {
    print('running');
  }
}

mixin Swim {
  void swimming() {
    print('swimming');
  }
}

class Person {
  String name;
  int age;

  Person(this.name, this.age);

  // 父类实现隐式接口
  void running() {
    print('running');
  }
}

/**
2、使用 with 实现混入，不需要重写方法
*/
class Tom extends Person with Run, Swim {
  Tom() : super('Tom', 18);
}
```

如果要类比的话，`mixin` 混入和 Swift 的协议有异曲同工之妙。

`implements` 和 `with` 关键字一起使用时，`implements` 要放最后：

```dart
void main(List<String> args) {
  var t = Tom();
  t.running();
}

abstract class Run {
  void running() {
    print('running');
  }
}

mixin Swim {
  void swimming() {
    print('swimming');
  }
}

class Person {
  String name;
  int age;

  Person(this.name, this.age);

  // 父类实现隐式接口
  void running() {
    print('running');
  }
}

class Tom extends Person with Swim implements Run {
  Tom() : super('Tom', 18);
}
```

### 类属性

```dart
void main(List<String> args) {
  var p = Person('Tom');
  Person.age = 18;
}

class Person {
  String name;

  static int age = 0;

  Person(this.name);
}
```

### 类方法

```dart
void main(List<String> args) {
  var p = Person('Tom');
  p.talkName();
  // Prints "Tom"

  Person.age = 18;
  Person.tallAge();
  // Prints "18"
}

class Person {
  String name;

  static int age = 0;

  Person(this.name);

  void talkName() {
    print(name);
  }

  static void tallAge() {
    print(age);
  }
}
```

### 枚举

选择器必须覆盖枚举类型的全部情况，否则会报错：

![10](Flutter之Dart/10.png)

```dart
void main(List<String> args) {
  final color = Colors.red;

  switch (color) {
    case Colors.red:
      print('红色');
      break;
    case Colors.blue:
      print('蓝色');
      break;
    default:
      print('绿色');
      break;
  }
  // Prints "红色"
}

enum Colors { red, blue, green }
```

## Dart中库的使用

### 一、使用系统库

使用 `import '库的url'` 的形式导入一个库。

```dart
import 'dart:math';

void main(List<String> args) {
  final num1 = 20;
  final num2 = 30;
  print(min(num1, num2));
  // Prints "50"
}
```

库url有三种方式：

1、使用 `dart:` 表示 dart 标准库，

```dart
import 'dart:io';
import 'dart:math';
import 'dart:html';
```

2、使用相对路径，导入项目中自定义的 .dart 文件：

```dart
import 'utils/math_utils.dart'
```

3、Pub包管理系统中的库，使用前缀 `package:`：

```dart
import 'package:http/http.dart'
```

### 二、使用自定义库

![11](Flutter之Dart/11.png)

1、`as` 关键字给库起别名。

当各个库有命名冲突的时候，可以使用 `as` 关键字来使用命名空间：

```dart
import 'utils/math_utils.dart' as mUtils;
```

2、默认情况下，导入一个库时就是导入这个库中所有的内容。

`show`：执行要导入的内容。

```dart
import "utils/math_utils.dart" show sum, mul;
```

`hide`：隐藏某个要导入的内容，导入其它内容。

```dart
import "utils/math_utils.dart" hide mul;
```

3、将需要导入的库，使用 `export` 抽取到公共的 dart 文件

![12](Flutter之Dart/12.png)

```dart
import "utils/utils.dart";

void main(List<String> args) {
  print(sum(20, 30));
  // Prints "50"
  print(min(10, 11));
  // Prints "10"
}
```

### 使用第三方库

```dart
import 'package:http/http.dart' as http;

main(List<String> args) async {
  var url = 'http://123.207.32.32:8000/home/multidata';
  var response = await http.get(url);
  print('Response status: ${response.statusCode}');
  print('Response body: ${response.body}');
}
```

参考 [http 1.1.0 ](https://pub.dev/packages/http/install)