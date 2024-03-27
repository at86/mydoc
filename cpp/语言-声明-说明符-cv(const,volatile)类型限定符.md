本文参考cppreference 上的`cv（const 与 volatile）类型限定符`说明.

https://zh.cppreference.com/w/cpp/language/cv
https://zhuanlan.zhihu.com/p/393548388

https://zhuanlan.zhihu.com/p/344264694



# 简介
可出现于任何 类型说明符（包括 声明语法 的 声明说明符序列）中，以指定 被声明对象 或 被命名类型 的常量性（constness）或易变性（volatility）。

const ——定义类型为常量 ﻿类型，。

volatile ——定义类型为易变 ﻿类型



## const 出现位置，以及结合 volatile 使用效果

局部const存在栈上，通过转换，可去除const限定，运行时能写入:

无volatile限定时，因编译器常量折叠，不能读出修改后的值;

有volatile限定时，禁止编译器常量折叠，能读出修改后的值;

转换后，只是对栈上局部变量写入。

全局const存在只读的全局数据区，通过转换，可去除const限定，能编译，运行时写入报错。 转换后，对只读的全局数据区写操作，引发段错误。



# 例子
#include "pch.h"
#include "CsFvLang.h"

namespace {
const volatile int a = 10;
}
void CsFvLang::LangVolatile() {
    at_mlog("====== LangVolatile ======\n");
    //==局部 const: 在栈上分配空间,本质是局部变量,指针转换后,可修改.
    {
        const int a = 10;
        //a = 100;  // err: 表达式必须是可修改的左值
        int* aPtr = ( int* )&a;
        //=a是局部常量,转换后,可写入
        *aPtr = 100;
        //=a是常量的,访问a不重读a地址的值
        at_mtrace("%d,%d\n", a, *aPtr);  //10,100
    }
    {
        const volatile int a = 10;
        //a = 100;  // err: 表达式必须是可修改的左值
        int* aPtr = ( int* )&a;
        //=a是局部常量,转换后,可写入
        *aPtr = 100;
        //=a是常量易变的,访问a重读a地址的值
        at_mtrace("%d,%d\n", a, *aPtr);  //100,100
    }
    //==全局 const: 放在常量只读区(只读的全局数据区),指针转换后,可编译,运行时修改该区数据会引发段错误;
    {
        //a = 100;  // err: 表达式必须是可修改的左值
        int* aPtr = ( int* )&a;
        //=a是全局常量,转换后,编译ok,运行时写入报错
        //*aPtr = 100;//err: 运行时,写入访问权限冲突
        at_mtrace("%d,%d\n", a, *aPtr);
    }
}


## 输出

L17 : 10,100
L26 : 100,100
L34 : 10,10


# 解释

函数类型和引用类型以外的每个（可能不完整的）类型都是一个包含以下四个不同但有关联的类型的组中的一个类型：
无 cv 限定 ﻿版本

const 限定 ﻿版本

volatile 限定 ﻿版本

const volatile 限定 ﻿版本

组内的四个成员有着相同的表示和对齐要求。
数组类型被当做与它的元素类型有相同的 cv 限定。

## 常量和易变对象

当对象创建时，所用的 cv 限定符（可以是声明中的声明说明符序列 ﻿或 声明符 的一部分，或者是 new 表达式中的 类型标识 的一部分）决定对象的常量性或易变性，如下所示：

### 常量对象 ﻿是以下对象之一

类型具有 const 限定的对象

常量对象的不可变子对象

这种对象不能被修改：直接尝试这么做是编译时错误，而间接尝试这么做（例如通过到非常量类型的引用或指针修改常量对象）的行为未定义。

### 易变对象 ﻿是以下对象之一

类型具有 volatile 限定的对象

易变对象的子对象

常量易变对象的可变子对象

每次访问（读或写、调用成员函数等）易变类型之泛左值表达式，都当作优化方面可见的副作用（即在单个执行线程内，易变对象访问不能被优化掉，或者与另一先于或后于该易变对象访问的可见副作用进行重排序。这使得易变对象适用于与信号处理函数而非另一执行线程交流，参阅 std::memory_order）。试图通过非易变类型的泛左值访问易变对象（例如，通过到非易变类型的引用或指针）的行为未定义。

### 常量易变对象 ﻿是以下对象之一

类型具有 const volatile 限定的对象

常量易变对象的不可变子对象

易变对象的常量子对象

常量对象的不可变易变子对象

同时表现为常量对象与易变对象。



每个 cv 限定符（const 和 volatile）在任何 cv 限定符序列中都最多只能出现一次。例如 const const 和 volatile const volatile 都不是合法的 cv 限定符序列。




## mutable 说明符

mutable - 允许常量类类型对象修改相应类成员（即类成员可变）。

可以在非引用非常量类型的非静态数据成员的声明中出现：
class X
{
    mutable const int* p; // OK
    mutable int* const q; // 非良构
    mutable int&       r; // 非良构
};

mutable 用于指定不影响类的外部可观察状态的成员（通常用于互斥量、记忆缓存、惰性求值和访问指令等）。 
class ThreadsafeCounter
{
    mutable std::mutex m; // “M&M 规则”：mutable 与 mutex 一起出现
    int data = 0;
public:
    int get() const{
        std::lock_guard<std::mutex> lk(m);
        return data;
    }
 
    void inc(){
        std::lock_guard<std::mutex> lk(m);
        ++data;
    }
};

## 转换

存在一种基于限制性的增长顺序的 cv 限定符的偏序。从而可以称类型具有更多 ﻿或更少 ﻿的 cv 限定：
无限定 < const

无限定 < volatile

无限定 < const volatile

const < const volatile

volatile < const volatile


到 cv 限定类型的引用和指针，能被隐式转换成，更多 cv 限定的类型的引用和指针，详情见限定性转换。
想要将到 cv 限定类型的引用或指针，转换成，更少 cv 限定类型的引用或指针，就必须使用 const_cast。


## 关键词

const, volatile, mutable



## 注解

在未声明为 extern 的非局部非异变非模板 (C++14 起)非 inline (C++17 起)变量声明上使用 const 限定符，会给予该变量内部连接。这有别于 C，其中 const 文件作用域对象拥有外部连接。
C++ 语言语法把 mutable 当做存储类说明符而非类型限定符，但它不影响存储类或连接。


volatile 的一些用法被弃用：(C++20 起)

volatile 类型的左值作为内建自增/自减运算符的操作数；

volatile 类型的左值作为内建直接赋值运算符的左操作数，除非该直接赋值表达式出现于不求值语境或是弃值表达式；

具有 volatile 限定的对象类型作为函数参数类型或返回类型；

volatile 限定符在结构化绑定声明中。
