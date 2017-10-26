---
layout: page
title: 深刻理解c++11基本类型
category: 
    - blogs
---
以前通过菜鸟教程自学了c++，自以为掌握c++所有基础知识和基本语法。今天偶尔翻下传说中的经典c++入门书籍《primer c++第五版 》。看到第二章才发现菜鸟还真是菜鸟，自己的基础是多么的脆弱不堪。
不说别的了，现在让我总结一下我之前没能彻底理解的c++基本类型，内置基本类型，类型转换，变量等基本概念这里不会详细讨论。

**一、引用**
----
引用的本质是为对象另外起一个名字。通过将声明符写成&d的形式来定义引用类型。
如下
~~~cs
int a = 102;
int &b = a;//b是a的引用

b = 2;//实际上b指向的对象被赋值，也就是 a = 2;
int c = b;//实际上是 c = a;
int &d = b;//实际上是 int &d = a;
~~~
定义引用时，程序会将**引用和它的初始值绑定**，而不是通过拷贝赋值。程序初始化后，无法令引用绑定别的对象，只能和之前的对象绑定。
所以**引用必须初始化**

~~~cs
int &b;//所以这种未给引用初始化的是错误的
~~~
引用本身不是对象，所以**不能定义引用的引用**。

引用本身是给对象改名，所以对纯粹的数是不能使用引用的
如下是错误的

~~~cs
int &a = 10;//错误的，引用类型的初始值必须是对象
~~~

使用引用时要注意引用和被引用对象类型是否一致
如下

~~~cs
double a = 2.3;
int &b = a;//这是错误的，引用类型和被引用对象类型应一致
~~~

接下来是c系语言最强大也最令人难受的指针了
**二、指针**
----
指针和引用类似，但**不同于引用，它是个对象，允许被赋值和拷贝，也不需要被初始化赋值**

~~~cs
int *a;//指向整数类型的指针
~~~
指针存放的是对象的地址，所以以下写法很常见

~~~cs
int *a = &b;//a存放b的地址;或者说a是指向b的指针
~~~
和引用相似，**指针和其指向的对象类型需一致**

~~~cs
double *a;
double b;
int *c = &a;//错误
c = b;//错误
void* d = &a;//void*指针可以存放任意类型的指针
~~~
如何访问指针指向的对象，那就是用解引用符（*）
~~~cs
int a = 1024;
int *b = &a;
cout << *b;//输出的将是1024
*b = 0;//实际是为指针指向的对象赋值
cout << a;//输出为0
int *c;
c = b;//c，b都指向a
~~~
指针有一种不指向任何对象，那就是空指针。
如下
~~~cs
int *a = nullptr;
~~~
复合类型
----
~~~cs
int a = 1;
int *b = &a;
int **c = &b;//指向指针的指针

int *d;
int *&e = d;//指针的引用
e = &a;//引用了指针，实际上d指向a
*e = 0;//实际是 a = 0;
~~~
**三、常量**
----
~~~cs
const int a = 2;//用const限定的为常量
a = 5;//由于a是被const的常量，所以a无法
~~~
因为其值不能改变，const对象必须初始化
~~~cs
const int a;//这是错误的，没有初始化a
~~~
const对象仅在文件内有效，如果希望多个文件共享常量，需要声明，
如下
a.cpp文件初始化一个常量a，能被其他文件共享
~~~cs
extern const int a =44;

~~~
a.h文件，多个文件引入它，就能在多个文件共享常量a
~~~cs
extern const int a;//与a.cpp中定义的a是同一个
~~~
const的引用
--------
引用可以绑定到常量上，但**不能通过修改引用来改变常量**
~~~cs
const int a = 1;
const int &b = a;//常量的引用也需要const来修饰
b = 4;//这是错误的，不能修改常量的引用
int &c = b;//这是错误的，常量的引用需要const修定


~~~
注意常量引用和普通引用的区别
如下
~~~s
const int &a = 66;//这是正确的，这也是个常量引用
int &b = 10;//而这是错误的，原因之前已经阐述过

double c =83.3;
const int &d = c;//这是正确的，接下来会发生强制类型转换
int &e = c;//而这是错误的，原因之前已经阐述过
~~~
指针和常量
--------
~~~cs
const int a = 1;
int *b = &a;//这是错误的
const int *c = &a;//常量的指针也需要const来修定
*c = 99;//这是错的

//const限定指针，只是不允许通过指针改变指针所指向的对象
int d = 4;
c = &d;//这是正确的，但不能通过c改变a的值
~~~
运行把指针定义为常量，常量指针必须被初始化
~~~cs
int a = 4;
int *const b = &a;//这样就不能改变指针的指向了
const int *const c = &a;//指针的指向和指针所指向对象的值都不能通过指针改变
~~~
顶层const
-------
之前我们可以发现指针有2种，一种不能改变指针指向，但指针本身并不是常量，这就是顶层const;另一种不能改变指针指向常量的值，但指向可以改变，这种被称为底层const
~~~cs
int a = 233;
int *const b =&a;//不能改变b的值，这是个顶层const
const int c =44;//不能改变c的值，这是个顶层const
const int *d =&c;//运行改变d的值，这是个底层const
const int *const e =&d;//靠右的是顶层const，靠左的是底层const
const int &f =a;//常量引用都是底层const

d =e;//正确，指向类型相同，e顶层const部分不受影响，d,e都是底层const
d=&a;//int* 能转换成const int*
~~~
constexpr常量表达式
-------
常量表达式是指值不会改变且在编译过程中得到计算结果的表达式
**一个对象是不是常量表达式由它的数据类型和初值决定**
~~~cs
const int a = 9;//a是常量表达式
const int b = a+1;//b是常量表达式
~~~
c++11标准规定允许将变量声明为constexpr类型以便编译器来验证变量值是否是一个常量表达式
~~~cs
constexpr int a = 9;//9是常量表达式
constexpr int b = a+1;//a+1是常量表达式
~~~
指针和constexpr
------------
constexpr如果声明一个指针，constexpr只对指针有效，与指针指向常量无关
~~~cs
const int *p = nullptr;//指向整数常量的指针
constexpr int *q  =nullptr//指向整数的常量指针

constexpr int i =99;//i的类型是整型常量
int j = 33;
constexpr const int *p1 =&i;//p1是常量指针，指向整型常量
constexpr int *q1 =&j;//q1是常量指针，指向整数j
~~~
**四、类型别名**
------
类型别名typedef能帮助程序员理解类型的真实目的。（可能我很菜吧，嫌弃，感觉程序更复杂了）
~~~cs
typedef double wages;//wages是double的同义词
typedef wages  base,*p;//base是double的同义词，p是double*的同义词
wages o;//等同double o;
~~~
注意
~~~cs
typedef char *patring;
const patring cstr =0;//cstr是指向char的常量指针
const char *cstr=0;//指向const char的指针，2者不一样
~~~
**五、auto**
------
auto类型说明符能让编译器替我们分析表达式所属的类型
使用auto时需要注意auto在一个语句能声明多个变量，但只能有一个基本数据类型
~~~cs
auto i =0,*p =&i;//i是整数，p是整型指针
auto a=0,b=9.3;//这是错误的，a，b类型需一致
~~~
auto会忽略顶层const，底层const会被保留下来
~~~cs
auto i =0,*p =&i;
const int c =i,&d =c;
auto e =c;//e是个整数，顶层const被忽略
auto f =d;//f是个整数，d是c的别名，c是个顶层const
auto g =&i;//g是个整型指针
auto h =&c;//h是个指向整型常量的指针

//如果希望推断出的const是个顶层const,需要明确指出
const auto m =c;
~~~
**六、decltype**
------
作用是选择返回操作数的数据类型
~~~cs
decltype(f()) sum = x;//sum的类型是f的返回类型
const int a =0,&b =a;
decltype(a) c=0;//const int c =0;
decltypee(b) d =c;//const int &d =c;
decltype(b) e;//错误，引用需要初始化
~~~
使用decltype时，如果在变量多加上一层括号，会有所不同
~~~cs
int a =33;
decltype((a)) b//int &b;需要初始化
~~~