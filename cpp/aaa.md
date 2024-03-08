https://blog.csdn.net/qq_44824574/article/details/123472640
vector及实现自定义vector以及allocator和iterator

https://blog.csdn.net/qq_44824574/article/details/124001624
STL空间配置器（一级配置器及二级配置器）

https://zhuanlan.zhihu.com/p/674710855
译文：C++内存池（Memory Pool in C++）

https://zhuanlan.zhihu.com/p/139835423
内存池设计与实现

https://www.cnblogs.com/lemaden/p/10150665.html
C++-new 的六种重载形式

https://zhuanlan.zhihu.com/p/354046948
详解C++重载new, delete

int main()
{
 Foo* m = new Foo;
 Foo* m2 = new(m) Foo;
 std::cout << sizeof(m) << std::endl;
    // delete m2;
 delete m;
 return 0; 
}
placement new就是operator new重载的一种形式。上文说到，operator new的主要作用就是分配空间，初始化对象的工作是new关键字的。

经过这样重载，placement new完全没有分配新的空间，而是把ptr的地址传了出去。由于调用构造函数的工作是new关键字，所以placement new不影响对象初始化。

placement new由于不需要申请新内存和释放旧内存。所以在内存池中，意外的便捷。
