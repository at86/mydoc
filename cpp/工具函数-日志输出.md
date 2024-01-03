> win32 环境；visual studio 即时窗口

## at_mlog

本文上接`ss`，描述如何实现一个易用的`日志宏`涉及到的知识点，以及为了更好适配`visual studio 2022`热更新而选择的实现方式。

## 功能主要有：

- 1, 能输出日志点代码的文件名和行号
- 2, 日志输出的时间
- 3, 不限数量的输入参数

## 用于实现`功能1 和 功能2`的`函数对象``

任何定义了`函数调用操作符`的对象都是`函数对象`。

简单来说，类里面需要有至少一个类似`T operator(参数列表);`这样的成员函数。

在我实现的这个日志宏里面，就是通过使用`函数对象`，避免在调用地方手写`__FILE__`和`__LINE__`。这里是对`函数对象`的使用，主要参考了`MFC`里面`TRACE`宏的实现。

我这里定义的`函数对象`是`at::MFCLog`：

- `函数调用操作符`的声明是`void operator()(const char* pszFmt, ...);`
- 构造函数的声明是`MfcLog(const char* pszFileName, int nLineNo);`

日志宏定义为：`#define at_mlog at::MfcLog(__FILE__, __LINE__)`，然后，在代码里调用`at_mlog("%s", "hello")`，通过宏替换后变成了`at::MfcLog(__FILE__, __LINE__)("%s", "hello")`。
通过宏替换：

- 构造了一个对象，在构造函数内，保存文件名和行号，记录下当前开始时间
- 形参列表被该对象的`函数调用操作符`函数接收处理

## 用于实现`功能3`的`变参数函数`

> 这里可以用`std::vformat()`处理`不限数量的输入参数`，同时需要把函数对象的`函数调用操作符`声明成一个模板函数，但是在使用过程中，发现会影响`visual studio 2022`的代码热更新，例如是在一个函数内第一次使用`at_mlog`会提示热更新失败，需要停止重新调试。所以这里仅展示对`变参数函数`的使用描述。

### 使用到的`变参数函数`相关的函数：

- va_list

用于保有宏 va_start 、 va_arg 及 va_end 所需信息的完整对象类型。用该类型，声明一个对象`args`，需要通过`va_start()`对`args`进行初始化。

- va_start()

启用对可变函数实参的访问。

- va_end()

结束对可变函数实参的遍历。

### 转换`可变参数`到字符串

参考 https://zh.cppreference.com/w/c/io/vfprintf 里面的`声明(4)`
需要用到`int vsnprintf( char *restrict buffer, size_t bufsz, const char *restrict format, va_list vlist );`

- buffer - 指向要写入的字符串的指针
- bufsz - 至多可以写入 bufsz - 1 个字符，再加上空终止符
- format - 指向指定数据转译方式的空终止字符串的指针
- vlist - 包含要打印数据的变量参数列表

`声明(4)`对应描述为：将结果写入字符串 buffer。至多写入 bufsz 个字符。产生的字符串将以空字符终止，除非 bufsz 为零。若 bufsz 为零，则不写入任何内容，且 buffer 可为空指针，然而照样计算并返回返回值（本应写入的字节数，不包含空终止符）。

根据描述，进行两次调用：

- 第一次调用`vsnprintf`时，buffer 传`nullptr`，bufsz 传`0`，得到要写入的字节数`n`
- 第二次调用`vsnprintf`时，buffer 传`要写入的字符串的指针`，bufsz 传要写入的字节数`n+1`

## 代码

```cpp
//=========== log.h =========================
//== w1: 用`变参数函数`的实现,利于代码热更新
namespace at {
extern SYSTEMTIME _st;  // 定义系统时间结构体的对象
void getSystime();

class MfcLog {
public:
    MfcLog(const char* pszFileName, int nLineNo);
    void operator()(const char* pszFmt, ...);

    int lineNo;
    std::string strTm;
};
}  // namespace at

#define at_mlog at::MfcLog(__FILE__, __LINE__)


//=========== log.cpp =========================
namespace at {

SYSTEMTIME _st;
void getSystime() {
    GetLocalTime(&_st);
}

MfcLog::MfcLog(const char* pszFileName, int nLineNo) : lineNo(nLineNo) {
    getSystime();
    static char tmStr[64];
    sprintf(tmStr, "%02d:%02d:%02d:%03d [%d] %s:%d : ", _st.wHour, _st.wMinute, _st.wSecond,
            _st.wMilliseconds, std::this_thread::get_id(), pszFileName, lineNo);
    strTm += tmStr;
}

void MfcLog::operator()(const char* fmt, ...) {
    va_list args1;
    va_start(args1, fmt);
    auto n = std::vsnprintf(nullptr, 0, fmt, args1);
    auto strTmLen = strTm.size();
    strTm.resize(strTmLen + n, NULL);
    std::vsnprintf(strTm.data() + strTmLen, n + 1, fmt, args1);
    va_end(args1);
    auto ws = at::utf8_decode(strTm);
    OutputDebugString(ws.data());
}

}
```
