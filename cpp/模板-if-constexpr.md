通过下面代码看`if`特点, 来了解为何引入`if constexpr`.

- if

```cpp
template < class T >
std::string asString1(T x) {
    if (std::is_same_v< T, std::string >) {
        return x;  //1
    }
    if (std::is_arithmetic_v< T >) {
        return std::to_string(x);  //2, 和1,3处冲突
    }
    return std::string(x);  //3
}
```

因为对于`if`语句执行`运行时`检查, 对于参数`x`, 需要在所有出现的`3个`地方都成立, 但是位置`2`和`1,3`处冲突, 所以上面代码报错.

- `if constexpr`

```cpp
template < class T >
std::string asString2(T x) {
    if constexpr (std::is_same_v< T, std::string >) {
        return x;
    }
    if constexpr (std::is_arithmetic_v< T >) {
        return std::to_string(x);
    }
    return std::string(x);
}
```

对于`if constexpr`语句执行`编译期`检查, 在每一个实例化中，无效的分支代码块会在编译时被`丢弃`, 所以上面的编译成功.
`丢弃`不是不被编译器检查, 被`丢弃`的代码块是要能通过编译器检查的`有效代码`.

只能在函数体内使用`if constexpr`, 因此, 不能使用它来替换`预处理器`的`条件编译`。

## 一般的常量判断也能条件编译

c++编译器优化, 对于`常量判断`都能达到类似`if constexpr`条件编译效果, 如在: `if(true) {add1();} else {add2();}`, `true ? add1() : add2()`判断, 都能只编译`add1()`函数.

```cpp
#include <iostream>
using namespace std;

template <class T>
T add1(T n) {
    return n + n;
}
template <class T>
T add2(T n) {
    return n * n;
}
template <class T>
constexpr T test(T n) {
    T s = std::is_same_v<T, int> ? add1(n) : add2(n);
    return s;
}
int main() {
    const int n = 3;
    cout << test(n) << endl;
}
```

上面代码通过 https://godbolt.org/ 查看编译 asm, 看到只有`add1()`被编译.
