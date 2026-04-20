# 内核事件表和系统调用

Linux 特有的 I/O 复用函数，创建一个文件描述符标识内核的一个事件表，然后把用户关心的文件描述符上的事件放在这个事件表中，使用 `epoll_create` 或 `epoll_create1` 创建事件表：

```cpp
#include <sys/epoll.h>
int epoll_create(int size);
int epoll_create1(int flag);
```

返回文件描述符。`epoll_create` 的 `size` 仅用于提示内核事件表的大小。`epoll_create1` 更现代化，引入了`flags`参数，允许传递标志来控制`epoll`实例的特性，例如 `EPOLL_CLOEXEC` 设置`close-on-exec`标志，使得这个文件描述符在执行`execve()`后自动关闭，避免资源泄露。

使用 `epoll_ctl` 操作内核事件表：

```cpp
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

成功返回 `0`，失败返回 `-1` 并设置 `errno`。`epfd` 为刚才创建好的事件表的文件描述符，`op` 指定操作类型，`fd` 为要操作的文件描述符，`event` 指定事件。

操作类型有3种宏：`EPOLL_CTL_ADD` 往事件表中注册 `fd` 的事件，`EPOLL_CTL_MOD` 表示修改，`EPOLL_CTL_DEL` 表示删除。

`event` 的类型是结构体 `epoll_event`，包括事件和对应用户数据，其中用户数据的类型是一个 `union`：

```cpp
struct epoll_event {
	__uint32_t events; // epoll 事件
	epoll_data_t data; // 用户数据
}

typedef union epoll_data{
	void* ptr; // 指定相关用户数据
	int fd; // 指定事件所属的目标文件描述符
	uint32_t u32;
	uint64_t u64;
} epoll_data_t;
```

设置好内核事件表后，使用 `epoll_wait` 系统调用，它在一段超时时间内等待一组文件描述符上的事件：

```cpp
int epoll_wait(int epfd, struct epoll_event* events, int maxevents, int timeout);
```

成功时返回就绪的文件描述符个数，失败返回 `-1` 并设置 `errno`，超时返回 `0`。

`maxevents` 指定最多监听多少个事件。`timeout` 控制其阻塞行为，允许应用程序在等待多个文件描述符事件时实现非阻塞、阻塞或带超时等待。`timeout = -1` (无限期阻塞直到事件发生)，`timeout = 0`(立即返回，不阻塞)，大于 0 (阻塞指定毫秒数，但不保证精确)

`epoll_wait` 如果检测到事件，就把所有就绪的事件从 `epfd` 指向的内核事件表复制到 `events` 指向的数组，该数组只输出 `epoll_wait` 检测到的就绪事件，提高了应用程序索引文件描述符的效率，和 `select` 和 `poll` 的数组有所不同。
# 操作模式
默认以 LT 工作
## LT(Level Trigger)
相当于效率较高的 `poll`。支持非阻塞和阻塞 socket。`epoll_wait` 监测到该事件 `fd` 已经就绪并通知应用程序后，可以不立即处理该事件（对一个 fd 进行 I/O 操作），然后每调用一次 `epoll_wait` 就继续通知直到该事件被处理。
## ET(Edge Trigger)
只支持非阻塞 socket。`epoll_wait` 监测到该事件 `fd` 已经就绪并通知应用程序后，必须立即处理该事件，做了某些操作直到该 `fd` 不再是就绪状态。注意变成就绪态后，如果一直不对这个 `fd` 作 I/O操作从而导致它再次变成未就绪，内核不会发送更多的通知。ET 降低同一个事件被重复触发的次数，效率比 LT 更高。