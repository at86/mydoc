> win32 环境；visual studio 即时窗口

## at_win_mem

有时需要观察并`记录`下一段代码，在执行过程中的`实时内存信息`，通过`PROCESS_MEMORY_COUNTERS_EX` `GetProcessMemoryInfo` `GetCurrentProcess` 等 win32 api 和 数据结构，封装一个工具函数`at_win_mem`，放在要观察代码起始位置，结合日志输出函数进行记录。

- `PROCESS_MEMORY_COUNTERS_EX`

  存放指定进程扩展内存信息的数据结构，使用如下两个字段：

  - .PrivateUsage 虚拟内存
  - .WorkingSetSize 物理内存

- `GetCurrentProcess`
  检索当前进程的句柄。

- `GetProcessMemoryInfo`
  检索指定进程的内存使用情况的信息。

```cpp
float AT_MB = 1024 * 1024;
std::string at_mb(float val) {
    char buf[32];
    std::sprintf(buf, "%.3f", val);
    return std::string(buf).append("MB");
}

std::string at_win_mem() {
    PROCESS_MEMORY_COUNTERS_EX pmc{};
    GetProcessMemoryInfo(GetCurrentProcess(), ( PROCESS_MEMORY_COUNTERS* )&pmc, sizeof(pmc));
    std::string s(56, NULL);
    std::sprintf(s.data(), "vMem:%s, pMem:%s", at_mb(pmc.PrivateUsage / AT_MB).c_str(),
                 at_mb(pmc.WorkingSetSize / AT_MB).c_str());
    return s;
}
```

## OutputDebugString

通过该函数，在调试过程中，把内存信息数据，输出到`visual studio 即时窗口`。同时为记录内存信息打点的位置信息，可以结合`__FILE__`，`__LINE__` 两个常量。

这里给一个简单的`OutputDebugString`调用示例：

```cpp
OutputDebugStringA(std::format("{} {}: {}!", __FILE__, __LINE__, at_win_mem().c_str()).c_str());
```

## 待分享的 at_mlog

在下篇文章里，会分享一个我常用的日志输出宏`at_mlog`，主要是包装的上面的`OutputDebugString`，减少参数录入。
