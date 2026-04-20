
# `popen`：管道通信

https://man7.org/linux/man-pages/man3/popen.3.html

通过 `popen` 创建管道，`fork` 子进程并调用 shell 来执行命令 `command`，从而开启一个进程，

使用 `pclose()` 关闭

```c
FILE *popen(const char *command, const char *type);
pclose(FILE* fp);
```

- `commane` 是要执行的命令字符串
- `type` 指定模式，`r` 表示读取 `w` 表示写入，**只能单向通信，要么读要么写**

管道是什么？

# `fgets`：从文件流获取字符串

https://man7.org/linux/man-pages/man3/fgets.3p.html

从文件流安全读取字符串，可以指定 `n` 因此可以防止缓冲区溢出。

读取最多 `n-1` 个字符直到碰到 `\n`，文件 `EOF` 或读取限制，获取字符串后会自动添加 `\0`，保留换行符。

```c
char *fgets(char *str, int n, FILE *stream);
```

- `str` 为存储读取到的字符串的缓冲字符数组指针
- `n` 包含了 `\0`，所以实际读取 `n-1`
- `stream` 指向读取的流，比如 `stdin` 或之前 `fopen`，`popen` 打开的文件指针。

# `std::string` 操作

```cpp
stod(); string 转double
```

## `substr`：截取

从原字符串中提取子串，从 `pos` 开始提取 `len` 位，不指定 `len` 为截取到末尾

```cpp
std::string::substr(size_t pos = 0, size_t len = npos) const;
```

# Day2

持有mutex的类不可以拷贝

lock_guard 使用，和 unique_lock 的区别（lock_guard 轻量，而且强制按照作用域范围解锁，如果需要提前解锁，或者设置条件变量要用 unique_lock）

QTime Qobject

# Day3

Qt 信号槽机制主要利用了事件循环（Event Loop）和元对象系统（Meta-Object System）的以下功能：

1. **事件循环（Event Loop）**:
2. **事件处理**：事件循环负责接收和分发事件。当一个信号被触发时，如果接收槽函数的对象位于不同的线程，事件循环将此信号作为事件处理，确保线程安全。
3. **跨线程通信**：事件循环支持跨线程的信号和槽调用。当一个信号在一个线程内被发射，而槽在另一个线程中，事件循环将信号槽调用作为事件加入目标线程的事件队列，从而实现线程间的安全通信。  
    
4. **元对象系统（Meta-Object System）**:  
    
5. **信号槽连接**：元对象系统提供了一种机制来动态识别和连接信号和槽。这是通过在运行时解析信号和槽的名字来完成的。
6. **反射能力**：元对象系统允许程序在运行时查询对象的信息，如类名、继承关系、属性、信号和槽等。这对于实现信号槽机制至关重要，因为它使得动态连接成为可能。
7. **运行时类型信息（RTTI）**：元对象系统使用一种特殊的运行时类型信息来支持信号和槽的匹配和验证，确保信号的参数类型与槽的参数类型兼容。