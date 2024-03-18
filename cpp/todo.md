## 树

图解：什么是 `B-树、B+树、B*树`
https://cloud.tencent.com/developer/article/1691641

## 表达式 - cppreference.com

https://zh.cppreference.com/w/cpp/language/expressions

- 潜在求值表达式; 不求值操作数; todo
- 弃值表达式 done;

## 标识表达式(id-expression) todo

Identifier expression
https://zh.cppreference.com/w/cpp/language/identifiers

## 隐式转换 - cppreference.com

https://zh.cppreference.com/w/cpp/language/implicit_conversion#.E4.B8.B4.E6.97.B6.E9.87.8F.E5.AE.9E.E8.B4.A8.E5.8C.96

- 临时量实质化

## C++ coroutine generator 实现笔记

这里, 郑重向大家推荐由清华大学出版社出版的《C++20 实践入门》和《C++20 高级编程》. 这两本书是目前最新的一批介绍 C++20 的教程. 该书紧跟潮流, 就 C++20 的几大热点内容, 如 modules、concepts 、ranges 等作了详细介绍. 全卷内容质量上乘, 精雕细琢, 非那些在历史旧版本的基础上草草加两章节新内容圈钱的书可比也！

作者: IceBear
链接: https://zhuanlan.zhihu.com/p/590892907
来源: 知乎
著作权归作者所有. 商业转载请联系作者获得授权, 非商业转载请注明出处.

# 非局部变量

- 非局部变量的动态初始化 https://stackoverflow.com/questions/63951527/what-is-unordered-dynamic-initialization-partially-ordered-dynamic-initializati
  - https://timsong-cpp.github.io/cppwp/basic.start#dynamic
- C++模板及实例化与具体化 https://blog.csdn.net/chqaz123/article/details/129355028

## C++ 非局部变量生命期

No\_机器人

于 2013-11-11 09:48:48 发布

阅读量 859
收藏 1

点赞数 1
文章标签： c++ initialization 全局变量 语言
版权
非局部变量(non-local variable)的生命期(lifetime/lifecycle)是 C++语言的一个阴暗角落，隐藏着很多细节问题。由于 C++11 新加了线程支持，非局部变量从存储特性上分为 static storage duration 和 thread storage duration 两类。SSD 对象的初始化和终止化伴随程序初始化和终止发生，TSD 对象的初始化和终止化伴随线程的执行和退出发生（程序或线程的初始化/终止化是因，变量的初始化/终止化是果）。

非局部变量的初始化分为静态初始化和动态初始化两个阶段。其中静态初始化包括零初始化和可能的常量初始化（C++11 引入了常量表达式，扩大了常量初始化的范畴）。所谓动态初始化是指除了零初始化和常量初始化之外的任何初始化。一般而言动态初始化需要运行期进行计算。

所有非局部变量都有零初始化过程。而是否有常量初始化和动态初始化则视非局部变量的初始化器而定。C++语言保证静态初始化一定在动态初始化开始之前完成，而且零初始化一定在任何其它初始化之前完成。这一保证是对所有非局部变量而言，即所有零初始化都完成之后才会对需要常量初始化的对象执行常量初始化，然后再对需要动态初始化的对象执行动态初始化。由于用户代码获得控制权要么是通过 main 函数要么是通过动态初始化，不论如何，用户代码得以运行时静态初始化都已完成。
static int kZero; // zero-initialization
static int kOne = 1; // zero-initialization, constant initialization
static int\* kVar = new int(2); // zero-initialization, dynamic initialization

根据定义语法非局部变量分为 3 种：

A. 全局变量。是否使用 static 关键字修饰以及是否位于名空间内不影响全局变量的生命期，只影响其作用域；

B. 类静态成员变量。包括显式及隐式实例化的类模板的静态成员变量；

C. 函数局部静态变量。

全局（静态）变量的初始化

虽然我们明确静态存储对象的初始化过程由程序初始化过程所启动，但是 C++标准不保证动态初始化在进入 main 函数之前完成([1], 3.6.2 节)，亦不保证跨编译单元的多个全局静态变量之间的初始化顺序(单个编译单元内以定义为序)，仅保证多个静态区对象的析构顺序是它们构造顺序的逆序。为了避免由于全局静态对象初始化顺序不确定导致程序故障[7]，程序必须的初始化工作不应依赖于全局静态变量的初始化顺序。典型的实践是通过函数对全局数据进行保护，确保在引用全局数据前其已被正确初始化，例如单例模式（下例是非线程安全的）：

class Config
{
static Config \* \_ptr;

public:
static Config & Config::instance(){
if (!\_ptr) {
\_ptr = new Utility;
}
return \*\_ptr;
}
}

Config\* Config::\_ptr; // will be zero initialized

类静态成员变量的初始化

类静态成员变量的初始化并不需要与全局变量的初始化进行区分。需要注意的是模板实例的静态成员是否有被实例化（模板实例的成员必须被引用到才会被实例化，如果没有被实例化就谈不上初始化）以及实例化的顺序。虽然编译单元内全局变量的初始化顺序与它们被定义的顺序一致，但是类模板被隐式实例化的顺序并不等同于其静态成员被定义的顺序。要保证模板实例的静态成员按特定顺序被初始化，必须显式实例化[4]。

函数局部静态变量的初始化

函数局部静态变量的初始化策略则较为明确： 常量初始化在进入在第一次进入变量所在代码块时进行，动态初始化在程序控制流第一次经过变量声明之处时进行。编译器自动在变量初始化代码前后插入保护性代码，保证初始化代码线程安全以及只完整执行一次。若动态初始化过程中由于抛出异常而使得初始化未能完成，下一次控制流经过变量声明之处时会再次尝试初始化。

int bar(bool t) { if(t) throw 0; return 123; }
void foo(){
static int K = bar(false);
static int L = bar(true);
}

初始化 K 时正常结束，因而 bar(false)仅会被执行一次。而由于初始化 L 时异常，下一次进入 foo 函数时 bar(true)会被再次执行。
非局部变量的析构

非局部变量的析构相比其初始化来说是相当直白的。对于 SSD 变量，析构由 main 函数返回或者程序调用 exit 触发。对于 TSD 变量，析构由线程函数返回或者线程调用 exit 触发。C++保证 TSD 对象的析构先于 SSD 对象（亦是先构造后析构原则的体现）。由于程序已经进入消亡期，析构所需要的依赖关系大大低于初始化。绝大部分情况下我们只需要关心诸如异常、不适宜的 abort 等函数调用引起的问题（析构未能被执行）。

参考

[1] C++11working paper (N3242)

[2] Stackoverflow Question: Lifetime of a static variable ina C function

[3] DynamicInitialization and Destruction with Concurrency(N2325)

[4] Stackoverflow Question: C++ Static member initalization

[5] http://en.wikipedia.org/wiki/Object_lifetime

[6] http://akrzemi1.wordpress.com/2012/05/27/constant-initialization/

[7] http://www.parashift.com/c++-faq-lite/static-init-order.html
————————————————
版权声明：本文为 CSDN 博主「No\_机器人」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/no_robot/article/details/15334697
