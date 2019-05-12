# Dart语法

重要的概念：
1. 所有的变量都是对象，所有的对象都是一个类。数字，函数和null也是对象。所有的对象都继承自Object类
2. Dart是强类型语言，但是Dart可以自动推断类型，所有声明变量时类型是可选的。当你确定没有预计的类型的，可以使用关键字dynamic
3. Dart支持通用类型，也就是泛型
4. Dart支持顶级函数(main())以及绑定到类或对象的函数(静态或者实例方法)。可以在函数内创建函数
5. 同样的，Dart支持顶级变量，以及绑定到类或对象的变量(静态或非静态)。变量又被称为字段或者属性
6. 和java不同，Dart没有关键字public,protected和private，通过变量或者函数名字前边加下划线("_")，来表明该变量或函数为私有变量或私有函数
7. 变量或者函数的命名规则为：以字母或者_为开头，后跟字母或者数字的组合

 



## 变量
### 基本类型
```
bool done true;
int num = 2;
double x = 3.14;
final bool visible = false;
final double y = 2.7;
final int amount = 100;

const bool debug = true;
const int sum = 42;
const double z = 1.2;
```
1. Dart中没有byte、char和float
2. int、double都是64位
3. final表示一个运行时常量(在程序运行的时候赋值，赋值后值不再改变)
4. const表示一个编译时常量，在程序编译时它的值就确定了
5. Dart里所有的东西都是对象，包括int、函数

Dart具有类型推断功能：
```
var done = true;
var num = 2;
var x = 3.14;

final visible = false;
final amount = 100;
final y = 2.7;

const debug = true;
const sum = 42;
const z = 1.2;
```
### 字符串String
```
var str = 'foo';
var str2 = str.toUpperCase();
var str3 = str.trim();
assert (str == str2);
assert (!identical(str2, str3));
```
1. String是不可变对象
2. 使用==比较String的内容是否一样
3. 使用identical判断是否位同一个对象
4. 字符编码为UTF-16，使用单引号或者双引号都可以
5. 使用 $s在一个字符串之中添加变量，使用${}添加表达式
6. 使用+拼接字符串
7. 使用'''或者"""创建多行字符串
8. 在'或者"前边添加r来创建原始字符串


### List、Map和Set

#### List
创建对象时推荐省略new关键字
```
  //使用构造函数创建对象
   //跟var list = new List<int>();
   var list = List<int>();
   list.add(0);
   list.add(1);

   //通过字面量创建对象，list创建的泛型参数可以从变量定义推断出来
    //推荐使用字面量方式创建对象
   var list1 =[1,2];
   //没有元素，显示指定泛型参数为int
   var list2 = <int>[];
   list2.add(2);
   list2.add(3);

   //指向的是一个常量，不能添加元素
   var list3 = const[1,2];
   //但是list3本身不是一个常量，可以对其赋值
   list3 =[4,5];
   const list4 = [5.6];//等同于 const list4 = const[5.6];

   //遍历集合数组的方式
   var list5 =[1,2,3,4,5,6,7,8];
   for(var i in list5){
     print("i = $i");
   }
```
#### Set
```
 //创建set集合
    var set = Set<String>();
    set.add("123");
    set.add("2342");

    assert(set.contains("123"));
```

#### Map
```
dynamic obd = 'string';
Object o = 'string';
//dynamic 关键字声明的对象不会做类型检查
//Object 关键字声明的对象 会做类型检查以确保是类型安全的
//使用关键字is判断是哪种类型
if (obd is String) {}

//使用as 关键字 进行强制类型转换
String s = o as String;
```

### 语句
java中常见的语句 if else,do while,while和switch在Dart里面都支持


### 函数
支持可选参数和默认参数：
```
void main(){
    print(foo(2));
    print(foo(1,2));
}

//可选参数和默认参数
int foo(int x,[int y =1]){
  if(y !=null){
    return x + y;
  }
  return x;
}
//输出结果都为3
```
### ..操作符
使用它可以调用当前对象内部的方法或者字段
```
var button = querySelector('#confirmg
button.text = 'Confirm';
button.classes.add('important');
button.onClick.listen((e) => window.alert('Confirmed!'));
可以写成如下方式：

querySelector('#confirm') // Get an object.
  ..text = 'Confirm' // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));

```

### 类
1. 先执行子类 initializer list，但只初始化自己的成员变量
2. 初始化父类的成员变量
3. 执行父类构造函数的函数体
4. 执行子类构造函数的函数体

#### 构造函数
类似Java的构造函数，通常用来创建一个类的实例,dart的一个构造函数语法糖，可以直接给属性赋值
```
class Point {
  num x, y;

  // Syntactic sugar for setting x and y
  // before the constructor body runs.
  Point(this.x, this.y);
}
```
##### 默认构造函数
如果不声明构造韩式，系统将提供一个默认的构造函数。默认的构造函数没有参数，而且将调用父类的无参数构造函数
##### 构造函数不能继承 
子类不能从父类继承构造函数。声明无参数的构造函数的子类只有默认的构造函数，即没有参数、没有名字
##### 命名构造函数
通过命名构造函数实现一个类可以有多个构造函数，或者提供更有正对性的构造函数
```
class Point {
  num x, y;

  Point(this.x, this.y);

  // Named constructor 命名构造
  Point.origin() {
    x = 0;
    y = 0;
  }
}
```
注意：构造函数是不能继承测，所以子类是不能继承父类的命名构造函数。如果希望使用父类中的构造函数创建子类的实例，必须在子类中实现父类中的构造函数

##### 调用父类非默认构造函数
默认，子类的构造函数调用父类非命名、无参构造函数。父类的构造函数在构造函数体之前调用。如果有初始化列表，初始化在父类构造函数之前自行，总之，执行顺序如下：
1. 初始化列表
2. 父类的无参构造函数
3. 当前类的无参构造函数

如果父类没有未命名、无参构造函数，那么你必须手动调用父类中的一个构造函数。注意：父类的构造函数调用在：之后，构造函数体之前
例如：Employee类的构造函数调用他父类Person的命名构造函数
```
class Person {
  String firstName;

  Person.fromJson(Map data) {
    print('in Person');
  }
}

class Employee extends Person {
  // Person does not have a default constructor;
  // you must call super.fromJson(data).
  Employee.fromJson(Map data) : super.fromJson(data) {
    print('in Employee');
  }
}

main() {
  var emp = new Employee.fromJson({});

  // Prints:
  // in Person
  // in Employee
  if (emp is Person) {
    // Type check
    emp.firstName = 'Bob';
  }
  (emp as Person).firstName = 'Bob';
}
```
构造函数的参数可以为表达式，父类构造函数不能使用this.例如，参数可以调用静态方法，但是不能调用实例方法


##### 初始化列表
在执行构造函数体之前初始化实例变量。用逗号分隔每个初始化
```
import 'dart:math';

class Point {
  final num x;
  final num y;
  final num distanceFromOrigin;

  Point(x, y)
      : x = x,
        y = y,
        distanceFromOrigin = sqrt(x * x + y * y);
}

main() {
  var p = new Point(2, 3);
  print(p.distanceFromOrigin);
}
```

##### 可重定向的构造函数
又是一个构造函数的目的只是重定向到同类的另一个构造函数。一个可重定向函数的函数体是空的，同时构造函数的调用是在冒号之后的
```
class Point {
  num x, y;

  // The main constructor for this class.
  Point(this.x, this.y);

  // Delegates to the main constructor.
  Point.alongXAxis(num x) : this(x, 0);
}

```

##### 常量构造函数
如果一个对象是不回改变的，可以将这个对象创建为编译时常量，定义cost构造函数而且要确保所有的常量都是final的
```
class ImmutablePoint {
  static final ImmutablePoint origin =
      const ImmutablePoint(0, 0);

  final num x, y;

  const ImmutablePoint(this.x, this.y);
}
```

##### 工厂构造函数
当你需要构造函数不是每次都创建一个新的对象时，使用factory关键字。例如工程构造函数返回一个在缓存中的实例或者返回一个子类的实例
```
class Logger {
  final String name;
  bool mute = false;

  // _cache is library-private, thanks to
  // the _ in front of its name.
  static final Map<String, Logger> _cache =
      <String, Logger>{};

  factory Logger(String name) {
    if (_cache.containsKey(name)) {
      return _cache[name];
    } else {
      final logger = Logger._internal(name);
      _cache[name] = logger;
      return logger;
    }
  }

  Logger._internal(this.name);

  void log(String msg) {
    if (!mute) print(msg);
  }
}
```
注意：工厂构造函数不能使用this

调用工厂构造函数，可以使用new关键字
```
var logger = Logger('UI');
logger.log('Button clicked');
```


这里只写了部分Dart基础,剩下的部分需要通过官网来学习

[Dart语法学习](https://www.dartlang.org/guides/language/language-tour)