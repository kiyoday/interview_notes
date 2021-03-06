##  阻塞／非阻塞

这两个概念是针对 IO 过程中进程的状态来说的，阻塞 IO 是指`调用结果`返回之前，**当前线程会被挂起**；

相反，非阻塞指在不能立刻得到结果之前，该函数**不会阻塞当前线程，而会立刻返回**。

## 同步／异步

这两个概念是针对调用如果返回结果来说的，所谓同步，就是在发出一个功能调用时，在没有得到结果之前，该**调用就不返回**；

相反，当一个异步过程调用发出后，调用者不能立刻得到结果，实际处理这个调用的部件在完成后，通过`状态、通知和回调`来通知调用者。

## 阻塞与非阻塞

在介绍IO复用技术之前，先介绍一下阻塞和非阻塞，在我们前几节的WEB服务器中，调用`socket_accept`函数会使整个进程阻塞，直到有新连接，操作系统才唤醒进程继续执行。而非阻塞模式, `stream_socket_accept`的行为就不一样了，如果没有新连接，不会阻塞进程，而是马上返回false。

## IO 多路复用

多路复用（IO/Multiplexing）：为了提高数据信息在网络通信线路中传输的效率，在`一条物理通信线路`上建立`多条逻辑通信信道`，同时传输若干路信号的技术就叫做多路复用技术。对于 Socket 来说，应该说**能同时处理多个连接的模型**都应该被称为多路复用，目前比较常用的有 select/poll/epoll/kqueue 这些 IO 模型（目前也有像 Apache 这种每个连接用单独的进程/线程来处理的 IO 模型，但是效率相对比较差，也很容易出问题，所以暂时不做介绍了）。在这些多路复用的模式中，异步阻塞/非阻塞模式的扩展性和性能最好。

## select 轮询

使用`select`会轮询连接池，当有连接可读或可写时，`select`函数返回可读写的连接数，然后再轮询一遍连接池，查找活动连接进行读写操作。