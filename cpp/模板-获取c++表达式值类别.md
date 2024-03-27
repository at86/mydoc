最近看到[C++ value categories and decltype demystified](https://www.scs.stanford.edu/~dm/blog/decltype.html), 这篇文章提出通过`decltype((E))`获取`表达式值类别`的方法.

```cpp
namespace at {
template < typename T >
constexpr char ExprValCate[] = "prvalue";
template < typename T >
constexpr char ExprValCate< T& >[] = "lvalue";
template < typename T >
constexpr char ExprValCate< T&& >[] = "xvalue";
}  // namespace at

///获取表达式值类别 https://www.scs.stanford.edu/~dm/blog/decltype.html
#define at_exprvc(expr) at::ExprValCate< decltype((expr)) >

```

# 举个例子

```cpp
struct F1 {
};

F1 f1;
F1& f1_lRef = f1;
F1&& f1_prRef = std::move(f1);
F1* f1_ptr = &f1;

atlog(at_exprvc(std::move(f1)),  //xvalue
      at_exprvc(F1()),  //prvalue
      at_exprvc(f1),  //lvalue
      at_exprvc(f1_ptr),  //lvalue
      at_exprvc(std::move(f1_ptr)),  //xvalue
      at_exprvc(*std::move(f1_ptr)),  //lvalue
      at_exprvc(f1_prRef),  //lvalue
      at_exprvc(f1_lRef),  //lvalue
      at_exprvc("abc"),  //lvalue
      at_exprvc(1)  //prvalue
);

```
