WebServer

目的：实现一个基础的多并发网络服务程序

* 线程互斥锁&条件互斥锁的封装
* 线程池的设计，用以支持并发
* 基础网络连接的实现
* http协议的简略支持

一、互斥锁&条件变量

1.互斥锁

上锁的目的：当多个线程同时读写同一个数据时，可能会造成脏读的情况，为了保证同一时刻只有一个线程可以访问该数据。

```c
// 初始化 mutex 对象
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int pthread_mutex_init(pthread_mutex_t *restrict mutex,
                        const pthread_mutexattr_t *restrict attr);
// mutex 加锁，在获得锁之前将会阻塞
int pthread_mutex_lock(pthread_mutex_t *mutex);
// mutex 加锁，如果能马上获取锁则返回0，无法获取锁则马上返回errno
int pthread_mutex_trylock(pthread_mutex_t *mutex);
// mutex 释放锁
int pthread_mutex_unlock(pthread_mutex_t *mutex);
// 销毁 mutex 对象
int pthread_mutex_destroy(pthread_mutex_t *mutex);
```

`RAII`:资源获取即初始化，是C++等的资源管理、避免内存泄漏的方法。

```c++
/**
 * @brief MutexLock 将 pthread_mutex 封装成一个类, 
 *        这样做的好处是不用记住那些繁杂的 pthread 开头的函数使用方式
 */ 
class MutexLock
{
private:
    pthread_mutex_t mutex_;
public:
    MutexLock()     { pthread_mutex_init(&mutex_, nullptr); }
    ~MutexLock()    { pthread_mutex_destroy(&mutex_); }
    void lock()     { pthread_mutex_lock(&mutex_); }
    void unlock()   { pthread_mutex_unlock(&mutex_); }
    pthread_mutex_t* getMutex() { return &mutex_; };
};
```

正常来说，我们使用锁时，需要经过以下过程：**获取锁->进入临界区->释放锁**。但在实际使用锁时，容易忘记释放锁，而这是一个非常严重的错误。因此我们可以实现一个 `MutexLockGuard`类，借助类的构造函数和析构函数，来帮助我们自动获取锁和释放锁，只需一个简单的声明即可**获取锁&释放锁**。

```c++
/**
 * @brief MutexLockGuard 主要是为了自动获取锁/释放锁, 防止意外情况下忘记释放锁
 *        而且块状的锁定区域更容易让人理解代码
 */ 
class MutexLockGuard
{
private:
    MutexLock& lock_;
public:
    /**
     * @brief 声明 MutexLockGuard 时自动上锁
     * @param lock 待锁定的资源
     */
    MutexLockGuard(MutexLock& mutex) : lock_(mutex) { lock_.lock(); }
    /**
     * @brief 当前作用域结束时自动释放锁, 防止遗忘
     */ 
    ~MutexLockGuard() { lock_.unlock(); }
};
```

2.条件变量

作用：相当于控制线程在管程中挂起和唤醒的作用。

一说起条件变量，就不得不说先说起**管程**。管程保证了**同一时刻只有一个线程在管程内活动**，但**不能保证**线程在进入管程后，能继续一次性执行下去直到管程结束。例如某个线程好不容易进入了管程，但执行了一段时间，突然发现某个条件没有满足，使得当前线程必须阻塞，无法继续执行。但要是该线程原地阻塞，一直占用这个管程，那其他的线程自然就无法进入管程，造成死锁。

由于该线程进入管程后可能会阻塞，因此非常肯定的是，必须在该进程进入阻塞状态前释放管程，否则会造成死锁。但是该线程已经进入管程，且没办法继续执行下去，因此只能**原地释放管程**，并等待**条件**满足后，**重新获取管程锁**，并将该线程唤醒，使其继续执行。

例：线程池中，当子线程需要读取事件队列来获取事件之前，需要先获取队列的锁。当子线程获取到锁以后，如果队列为空，则条件不满足（注意这里的条件是：**队列非空**），因此子线程就无法从中获取事件，没法继续执行。此时可以使用条件变量让子线程在管程中挂起，等到条件满足时再通过条件变量来唤醒，回到管程继续执行。

注意：使用条件变量时，一定要确保**在已经获取到管程锁的前提下**使用，否则条件变量容易被多个子线程修改/使用。

```c++
// 初始化条件变量
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
int pthread_cond_init(pthread_cond_t *restrict cond,
                      const pthread_condattr_t *restrict attr);
// 唤醒 **至少一个** 被目标条件变量阻塞的线程
int pthread_cond_signal(pthread_cond_t *cond);
// 唤醒 **所有** 被目标条件变量阻塞的线程
int pthread_cond_broadcast(pthread_cond_t *cond);
// 让目标条件变量阻塞当前线程
int pthread_cond_wait(pthread_cond_t *restrict cond,
                      pthread_mutex_t *restrict mutex);
// 让目标条件变量阻塞当前线程，并设置最大阻塞时间
int pthread_cond_timedwait(pthread_cond_t *restrict cond,
                            pthread_mutex_t *restrict mutex,
                            const struct timespec *restrict abstime);
// 销毁条件变量
int pthread_cond_destroy(pthread_cond_t *cond);
```

```c++
/**
 * @brief 条件变量,主要用于多线程中的锁 
 *        与 MutexLock 一致,无需记住繁杂的函数名称
 *        条件变量主要是与mutex进行搭配,常用于资源分配相关的场景,
 *        例如当某个线程获取到锁以后,发现没有资源,则此时可以释放资源并等待条件变量
 * @note  注意: 使用条件变量时,必须上锁,防止出现多个线程共同使用条件变量
 */
class Condition
{
private:
    MutexLock& lock_;       // 目标 Mutex 互斥锁
    pthread_cond_t cond_;   // 条件变量
public:
    Condition(MutexLock& mutex) : lock_(mutex) { pthread_cond_init(&cond_, nullptr); }
    ~Condition()        { pthread_cond_destroy(&cond_); }
    void notify()       { pthread_cond_signal(&cond_); }
    void notifyAll()    { pthread_cond_broadcast(&cond_); }
    void wait()         { pthread_cond_wait(&cond_, lock_.getMutex()); }
    /**
     *  @brief  等待当前的条件变量一段时间
     *  @param  sec 等待的时间(单位:秒)
     *  @return 成功在时间内等待到则返回 true, 超时则返回 false
     */
    bool waitForSeconds(size_t sec)   
    {
        timespec abstime;
        // 获取当前系统真实时间
        clock_gettime(CLOCK_REALTIME, &abstime);
        abstime.tv_sec += (time_t)sec;
        return ETIMEDOUT != pthread_cond_timedwait(&cond_, lock_.getMutex(), &abstime);
    }
};
```

二、线程池

- 线程池是一种多线程的处理方式，常常用在高并发服务器上。线程池可以有效的利用高并发服务器上的线程资源。
- 线程用于处理各个请求，其流程大致为：**创建线程 => 传递信息至子线程 => 线程分离 => 线程运行 => 线程销毁**。对于较小规模的通信来说，上述的这个流程可以满足基本需求。但是对于高并发服务器来说，重复的创建线程与销毁线程，其开销不可忽视。因此可以使用线程池来让线程复用。
- 当前线程所要执行的函数，最好是与主线程没有较大关联，降低耦合
- 所要执行的事件，可以传入一个参数，但需要明确不能有返回值。
- 线程池除了一些特定的变量（线程个数、事件队列等），还需要**互斥锁**和**条件变量**。
  -  对于每个线程来说，这些线程是有可能同时访问线程池，因此需要在每个线程访问之前加以上锁。
  - 上锁之后，对于每个线程来说，有可能出现获取到锁，但没有事件可以执行的情况。对于这类情况，子线程必须先释放等待事件的到来，等到事件到来之后再重新上锁，获取事件，就利用到了条件变量。

```c++
// 每个线程的基本事件单元
struct ThreadpoolTask {
    void (* function) (void *);
    void *arguments;
}
```

由于子线程只会在线程池创建之时创建，在线程池销毁之时销毁，因此，在子线程中必然需要执行一个事件循环，其中重复执行获取事件、执行事件、的动作。

会面临的问题：

* 获取事件时，需要给线程池上锁，因为要访问消息队列，必要时刻还需要设置条件变量来暂时释放锁。
* 当线程池被销毁时，子线程如何终止？方法1：在线程池设置一个标志，子线程定期轮询该标志以确认是否退出；方法2：添加一个退出事件到事件队列中，子线程执行到该事件时自动退出。

注意，`pthread_cond_signal` 会唤醒**至少**一个线程，注意是**至少**。因此可能会出现唤醒多个线程但只有一个事件等待处理的情况。针对于这种情况，只需设置子线程在被唤醒后，循环检测是否有剩余事件等待处理即可。

```c
void* ThreadPool::TaskForWorkerThreads_(void* arg)
{
    ThreadPool* pool = (ThreadPool*)arg;
    // 启动当前线程
    ThreadpoolTask task;
    // 对于子线程来说,事件循环开始
    for(;;)
    {
        // 首先获取事件
        {
            // 获取事件时需要上个锁
            MutexLockGuard guard(pool->threadpool_mutex_);
            /** 
             * 如果好不容易获得到锁了,但是没有事件可以执行
             * 则陷入沉睡,释放锁,并等待唤醒
             * NOTE: 注意, pthread_cond_signal 会唤醒至少一个线程
             *       也就是说,可能存在被唤醒的线程仍然没有事件处理的情况
             *       这时只需循环wait即可.
             */ 
            while(pool->task_queue_.size() == 0)
                pool->threadpool_cond_.wait();
            // 唤醒后一定有事件
            assert(pool->task_queue_.size() != 0);
            task = pool->task_queue_.front();
            pool->task_queue_.pop();
        }
        // 执行事件
        (task.function)(task.arguments);
    }
    // 注意: UNREACHABLE, 控制流不可能会到达此处
    // 因为线程的退出不会走这条控制流,而是执行退出事件
    assert(0 && "TaskForWorkerThreads_ UNREACHABLE!");
    return nullptr;
}
```

创建线程池：

循环创建线程即可。

```c
ThreadPool::ThreadPool(size_t threadNum, ShutdownMode shutdown_mode, size_t maxQueueSize)
        : threadNum_(threadNum),
          maxQueueSize_(maxQueueSize), 
          // 使用 类成员变量 threadpool_mutex_ 来初始化 threadpool_cond_
          threadpool_cond_(threadpool_mutex_), 
          shutdown_mode_(shutdown_mode)
{
    // 开始循环创建线程 
    while(threads_.size() < threadNum_)
    {
        pthread_t thread;
        // 如果线程创建成功,则将其压入栈内存中
        if(!pthread_create(&thread, nullptr, TaskForWorkerThreads_, this))
        {
            threads_.push_back(thread);
            // // 注意这里只修改已启动的线程数量
            // startedThreadNum_++;
        }
    }
}
```

添加事件:

添加新事件时，需要设置一下锁，防止脏读，在新事件添加完后，使用条件变量来唤醒其中某一个空闲线程以执行新的事件。

```c++
bool ThreadPool::appendTask(void (*function)(void*), void* arguments)
{
    // 由于会操作事件队列,因此需要上锁
    MutexLockGuard guard(threadpool_mutex_);
    // 如果队列长度过长,则将当前task丢弃
    if(task_queue_.size() > maxQueueSize_)
        return false;
    else
    {
        // 添加task至列表中
        ThreadpoolTask task = { function, arguments };
        task_queue_.push(task);
        // 每当有新事件进入之时,只唤醒一个等待线程
        threadpool_cond_.notify();
        return true;
    }
}
```

销毁线程池：

销毁方式： 1. IMMEDIATE_SHUTDOWN：线程池将马上清空事件队列中的全部事件，并添加与线程个数相对应量的退出事件，这将会使每个线程在执行完当前事件后，马上执行退出事件以退出 。

```c++
auto pthreadExit = [](void *) {pthread_exit(0);}
```

2. GRACEFUL_QUIT：知识简单的添加退出事件，没有额外的清空之前的事件，这样线程池只会在所有事件全部结束后才真正的被销毁。

```c
ThreadPool::~ThreadPool()
{
    // 向任务队列中添加退出线程事件,注意上锁
    // 注意在 cond 使用之前一定要上 mutex
    {
        // 操作 task_queue_ 时一定要上锁
        MutexLockGuard guard(threadpool_mutex_);
        // 如果需要立即关闭当前的线程池,则
        if(shutdown_mode_ == IMMEDIATE_SHUTDOWN)
            // 先将当前队列清空
            while(!task_queue_.empty())
                task_queue_.pop();

        // 往任务队列中添加退出线程任务
        for(size_t i = 0; i < threadNum_; i++)
        {
            auto pthreadExit = [](void*) { pthread_exit(0); };
            ThreadpoolTask task = { pthreadExit, nullptr };
            task_queue_.push(task);
        }
        // 唤醒所有线程以执行退出操作
        threadpool_cond_.notifyAll();
    }
    for(size_t i = 0; i < threadNum_; i++)
    {
        // 回收线程资源
        pthread_join(threads_[i], nullptr);
    }
}
```

三、网络连接

执行完一次完整的网络连接通常需要执行数个socket类函数。

socket函数的声明如下：

```c
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>

// 执行成功则返回一个新文件描述符fd，失败则返回-1并设置errno
int socket(int domain, int type, int protocol);
// domain: 1.AF_UNIX/AF_LOCAL 本地通信，用于进程间通信，不经过网卡，2. AF_INET：IPv4协议，数据需要经过网卡； type:1.SOCK_STREAM：TCP通信，2.SOCK_DGRAM：UDP通信，3.SOCK_NONBLOCK：设置非阻塞socket，使用或运算符来附加属性，4.SOCK_CLOEXEC： 设置若当前程序执行 exec 时，对应文件描述符将在子进程中给关闭。使用 or 运算符来附加属性； protocol:用于指定某一个特定的套接字协议，如果给定协议系列只有一个协议可以支持特定的套接字类型，则protocol可以指定为0，但是若给定协议系列中可能存在多个可以支持套接字的类型，这时候就必须设置 protocol 以指定具体类型。

// 创建一个IPv4的TCP套接字：
int listen_fd = socket(AF_INET, SOCK_STREAM, 0);
```

bind:

对于一个**新创建**的 socket（注意是**新创建**的），还没有任何的地址用于赋给该 socket。而 bind 函数就是用于赋以一个地址给该 socket。

需要注意的是：如果当前 socket 在执行 bind 前已经被使用，则**操作系统将会自动分配地址以及端口号**等等，这也是为什么一些网络程序向外通信时使用的端口号是随机的，因为操作系统会在后面调控。

```c
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>

// 执行成功则返回0，失败则返回-1并设置errno
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
// sockfd:传入目标socket文件描述符， addr:其中使用的规则和结构体、在地址族之间有所不同，addrlen表示第二个参数addr所指向的结构体的size。
```

```c
//对于AF_INET：所使用到的address format如下
#include <netinet/in.h> // ! 注意头文件！！
struct sockaddr_in {
    sa_family_t    sin_family; /* address family: AF_INET 始终为AF_INET*/
    in_port_t      sin_port;   /* port in network byte order 设置目标端口*/ 
    struct in_addr sin_addr;   /* internet address 保存监听目标的地址*/
};

/* Internet address. */
struct in_addr {
    uint32_t       s_addr;     /* address in network byte order */
};

#define INADDR_ANY ((int_addr_t) 0x00000000)
```

提示：由于现代计算机可能有多张网卡，因此指定 sin_addr 可以使得只监听特定网卡的连接。如果想监听**全部网卡的连接**，则可以使用宏定义 **INADDR_ANY**（实际上就是 0.0.0.0）。而如果绑定 127.0.0.1 回环地址，则**只能监听到主动发送至回环地址的请求**，其他发送到当前该机器但目标IP非回环地址的请求则不会被处理。

socket提供了端序转换的一些函数，便于转换，

```c
/* Functions to convert between host and network byte order.

Please note that these functions normally take `unsigned long int' or
`unsigned short int' values as arguments and also return them.  But
this was a short-sighted decision since on different systems the types
may have different representations but the values are always the same.  */

extern uint32_t ntohl (uint32_t __netlong) __THROW __attribute__ ((__const__));
extern uint16_t ntohs (uint16_t __netshort)
__THROW __attribute__ ((__const__));
extern uint32_t htonl (uint32_t __hostlong)
__THROW __attribute__ ((__const__));
extern uint16_t htons (uint16_t __hostshort)
__THROW __attribute__ ((__const__));
```

如果需要网络端序IP地址和字符串类型zhuan'huan字符串类型转换，参考以下函数：

```c
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int inet_aton(const char *cp, struct in_addr *inp);

in_addr_t inet_addr(const char *cp);

in_addr_t inet_network(const char *cp);

char *inet_ntoa(struct in_addr in);

struct in_addr inet_makeaddr(in_addr_t net, in_addr_t host);

in_addr_t inet_lnaof(struct in_addr in);

in_addr_t inet_netof(struct in_addr in);
```

对于一个 AF_NET 地址族来说，执行 bind 的一个简单例子如下：

```c
// 绑定端口
sockaddr_in server_addr;
// 初始化一下
memset(&server_addr, '\0', sizeof(server_addr));
// 设置一下基本操作
server_addr.sin_family = AF_INET;
server_addr.sin_port = htonl((unsigned short)port);
server_addr.sin_addr.s_addr = htonl(INADDR_ANY);

// 试着bind
if(bind(listen_fd, (sockaddr*)&server_addr, sizeof(server_addr)) == -1)
    return -1;
```

listen:

listen 函数将会使得传入的 socket fd 变为**等待连接状态**。该函数原型如下：

```c
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>

// 成功则返回0，失败则返回-1并设置 errno
int listen(int sockfd, int backlog);
```

accept:

- accept 函数将会取出 **listen_fd的挂起连接等待队列** 中的第一个连接，创建一个新的 socket fd（client fd），并将其返回。后续与该连接的交互都是通过该client fd 完成。
- 需要注意的是， accept 会从 listen_fd 中取出挂起的连接，并尝试连接。一旦完成连接后，将会创建一个新的 client_fd。**原先的 listen_fd 不会有任何改变**。
- 如果当前 listen_fd 为**阻塞式**的，则如果当前挂起连接等待队列中不存在任何连接，那么**执行 accept 时将阻塞**，直到有新连接的到来。

```c
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen); 
// sockfd:监听socket listen_fd， addr:accept会把远程socket的address写入目标结构体中， addrlen存放结构体大小的指针
```

read/recv & write/send

由于 read/recv & write/send 操作涉及到**阻塞与非阻塞式读写**，因此我们需要额外对其做一些异常处理。

需要注意的是，socket读写中，除了使用read/write以外，还可以使用专用于套接字通信的send/recv函数族等等

* 错误码： 该类函数中**最常返回的错误**为 **EINTR** 以及 **EAGAIN**，其他错误暂时忽略。其中：**EINTR**：该错误常见于**阻塞式**的操作，提示当前操作被**中断**。如果一个进程在一个慢系统调用中阻塞时，捕获到信号并执行完信号处理例程返回时，这个系统调用将**不再被阻塞，而是被中断**，返回 EINTR。对于读写函数来说，当返回这类错误时，最常用的做法就是**重新执行**目标函数。**EAGAIN**：该错误常见于**非阻塞式**的操作，提示用户稍后再**重新执行**。

如：当以**非阻塞**方式大量发送数据时，如果缓冲区爆满，则产生 Resource temporarily unavailable的错误（资源暂时不可用），并返回 EAGAIN。当以**非阻塞**模式下读取数据，如果多次读取数据但没有数据可读，则此时不会阻塞等待数据，而是直接返回 EAGAIN。

对于 read 函数来说，由于数据取决于**远程**，因此当接收到 EAGAIN 时终止读取，直接返回；但对于 write 函数来说，由于数据取决于**当前服务器**，因此可以继续循环写入，直至数据完全写入。

对于 recv/send 函数来说，与 read/write 相比，将会额外多出部分专用于 socket 的错误码，例如 ECONNREFUSED、EPIPE 以及 ECONNRESET 等等。出于调试目的，在实现 读写函数的 wrapper时，将这两类读写函数全部集成在 wrapper中，并用一个bool参数来控制启用 read/write 还是 recv/send 函数。