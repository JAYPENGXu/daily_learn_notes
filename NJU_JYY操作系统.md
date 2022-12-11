**操作系统：系统程序的执行者和中断的管理者**

**操作系统：状态机的管理者**

#### 并发控制：互斥

一切都是状态机！！！

Q:如何在多处理器上实现线程互斥？

自旋锁和互斥锁的实现！！！

理解并发：线程（人）、共享内存（物理世界）

* 如果某个线程持有🔒，则其他线程的lock不能返回。

实现互斥的根本困难：不能同时读/写共享内存。

`load`的时候不能写，只能“看一眼就把眼睛闭上”，看到的东西马上就过时了

`store`的时候不能读，只能“闭着眼睛动手”，也不知道把什么改成了什么

解决问题的两种方法：

1 提出算法、解决问题

2 改变假设（软件不够，硬件来凑）

假设硬件能为我们提供一条“瞬时完成”的读+写指令

请所有人闭上眼睛。看一眼（load），然后贴上标签（store）

如果多人同时请求，硬件选出一个“胜者”

“败者”等待”胜者“完成后才能继续

X86原子操作：`LOCK指令前缀`

```c
//Atomic exchange(load + store)
//用xchg实现互斥
int xchg(volatile int *addr, int newval){
    int result;
    asm volatile("lock xchg %0, %1":"+m"(*addr), "=a"(result):"1"(newval));
    return result;
}
```

实现互斥：自旋锁

```c
int table = YES;
void lock(){
    retry:
    	int got = xchg(&table, NOPE);
    	if(got == NOPE)
            goto retry;
    	assert(got == YES);
}
void unlock(){
    xchg(&table, YES);
}
//--------------------
int locked = 0;
void lock(){while(xchg(&locked, 1));}
void unlock(xchg(&locked, 0);)
```

原子指令的模型：

保证之前的store都写入内存

保证load/store不与原子指令乱序

自旋锁的缺陷：

* 自旋（共享变量）会触发处理器间的缓存同步，增加延迟

* 除了进入临界区的线程，其他处理器上的线程都在空转

* 获得自旋锁的线程可能被操作系统切换出去

自旋锁的使用场景：

* 临界区几乎不拥堵
* 持有自旋锁时禁止执行流切换

操作系统内核的并发数据结构（短临界区）

Futex = Spin + Mutex

自旋锁（线程直接共享locked）

* 更快的fast path
  * xchg 成功 → 立即进入临界区，开销很小
* 更慢的slow path
  * xchg 失败 →浪费CPU自旋等待

睡眠锁（通过系统调用访问locked）

* (更快的fast path)上锁失败线程不再占用CPU
* (更慢的fast path)即便上锁成功也需要进出内核（syscall)

Futex（两者兼得）

* Fast path:一条原子指令，上锁成功立即返回
* Slow path:上锁失败，执行系统调用睡眠

#### 并发控制：同步

Q：如何在多处理器上协同多个线程完成任务？

典型的同步问题：生产者-消费者模型、哲学家吃饭

同步的实现方法：信号量、条件变量

线程同步：在某个时间点共同达到互相已知的状态

生产者-消费者问题：

```c
void Tproduce(){while(1) printf("(");} //生产资源、放入队列，嵌套深度不足n时才能打印
void Tconsume(){while(1) printf(")");} //从队列取出资源执行，嵌套深度>1时才能打印
```

同步：等到有空位再打印左括号，等到能配对时再打印右括号

条件变量API：

* wait(cv, mutex) zZz休眠

调用时必须保证已经获得mutex

释放mutex、进入休眠状态

* signal/notify(cv)私信：走起

如果有线程正在等待cv，则唤醒其中一个线程

* broadcast/notifyAll(cv) 所有人：走起

唤醒全部正在等待cv的线程

条件变量实现生产者-消费者(需要等待条件变量满足)

```c
void Tproduce() {
  mutex_lock(&lk);
  if (count == n) cond_wait(&cv, &lk);
  printf("("); count++; cond_signal(&cv);
  mutex_unlock(&lk);
}

void Tconsume() {
  mutex_lock(&lk);
  if (count == 0) cond_wait(&cv, &lk);
  printf(")"); count--; cond_signal(&cv);
  mutex_unlock(&lk);
}
```

条件变量实现并行计算

```c
struct job {
  void (*run)(void *arg);
  void *arg;
}

while (1) {
  struct job *job;

  mutex_lock(&mutex);
  while (! (job = get_job()) ) {
    wait(&cv, &mutex);
  }
  mutex_unlock(&mutex);

  job->run(job->arg); // 不需要持有锁
                      // 可以生成新的 job
                      // 注意回收分配的资源
}
```

信号量：（更衣室管理）

P(&sem) 

* 等待一个手环后返回
* 如果此时管理员手上有空闲的手环，立即返回

V(&sem)

* 变出一个手环送给管理员

信号量实现生产者-消费者

```c
void producer() {
  P(&empty);   // P()返回 -> 得到手环
  printf("("); // 假设线程安全
  V(&fill);
}
void consumer() {
  P(&fill);
  printf(")");
  V(&empty);
}
```

避免死锁：避免死锁产生的四个必要条件：

* 互斥，一个资源每次只能被一个进程使用
* 请求与保持，一个进程请求资源阻塞时，不释放已获得的资源
* 不剥夺，进程一获得的资源不能强行剥夺
* 循环等待，若干进程之间形成头尾相接的循环等待资源关系

数据竞争：不同的线程同时访问同一段内存，且至少一个是写。

（b'\xcd' * 80).decode('gb2312')



`gcc -O2 main.c -g`

`gdb a.out`

`strace ./a.out`

`start`

`layout src / layout asm`

`record full `： 记录所有的状态

`record stop`： 结束记录

`p val` ：查看val的值

`rsi` ：回退到上一个状态

gdb 显示 *no source available* :在编译阶段 添加 **-g**选项

状态机模型理解世界

程序执行==状态机执行

strace/gdb

操作系统的内核启动：CPU reset -> Firmware -> Boot loader -> Kernel_start() -> 执行第一个程序/bin/init -> 中断/异常的处理程序

退出QEMU  ^a + x



`starti`

`info inferiors`

`pmap process id` ：查看一个进程的所有地址空间

操作系统上的进程

程序：状态机

* C代码视角：语句
* 汇编/机器码视角：指令
* 与操作系统交互的方式：syscall

虚拟化：操作系统再物理内存中保存多个状态机

创建状态机： fork:做一份状态机完整的复制（内存，寄存器现场）

`int fork();`

* 立即复制状态机
* 新创建进程返回0
* 执行fork的进程返回子进程的进程号

状态机管理：替换状态机

`execve()`

* 将当前运行的状态机重置成另一个程序的初始状态

`int execve(const char *filename, char * const argv, char *const envp);`

* 执行名为filename的程序
* 允许对新状态机设置参数`argv(v)`和环境变量`envp(e)`，刚好对应了`main()`的参数

`exit()` 终止状态机

* 立即摧毁状态机
* 销毁当前状态机，并允许有一个返回值
* 子进程终止会通知父进程

环境变量：

* 使用`env`命令查看 `env | grep DISPLAY`
* PATH：可执行文件搜索路径
* PWD
* HOME：目录
* DISPLAY：图形输出
* PS1:shell：的提示符
* export：告诉shell再创建子进程时设置环境变量

进程的地址空间是如何创建的，如何更改的？

进程地址空间的管理API： `mmap`

静态链接：代码、数据、堆栈、堆区

动态链接：代码、数据、堆栈、堆区、`INTERP`

整个计算机系统如何“构建”：

1. 硬件：从CPU reset开始执行指令

2. Firmware：加载操作系统
3. 操作系统：状态机的管理者，初始化第一个进程，执行系统调用

shell:用户能直接操作的程序管理操作系统对象，（把用户指令翻译成系统调用的编程语言）。

`man shell` 

内核提供系统调用；shell提供用户接口

**`sh-xv6.c`** : a zero-dependency UNIX Shell 

一个功能完整的shell使用的操作系统对象和API

* session, process group, controlling terminal
* 文件描述符：open, close, pipe, dup, read, write
* 状态机管理：fork, execve, exit, wait, signal, kill, setpgid

C标准库

libc：纯粹的计算；文件描述符；更多的进程/操作系统功能；地址空间；无止境封装

workload分析：

* 越小的对象创建/分配越频繁
  * 字符串、临时变量等，生存周期可长可短
* 较为频繁地分配中等大小的对象
  * 较大的数组，复杂的对象，更长的生存周期
* 低频率的大对象
  * 巨大的容器，分配器，很长的生命周期

malloc , fast and slow

设置两套系统：

* fast path ：使所有CPU都能并行的申请内存
  * 性能极好、并行度极高、覆盖大部分情况
  * 但有小概率会失败
* slow path
  * 不在乎那么快
  * 但把困难的事情做好
* fast path设计
  * 线程都事先瓜分一些内存
  * 默认从自己的领地里分配
    * 除了再另一个CPU释放
  * 如果自己的领地不足，就从全局的池子里借一点

`ls > a.txt` 重定向

`ls | wc -l` 管道

`ls &` 后台

`strace -f ./a.out`

`strace -f ./a.out |& vim -`

`strace -f ./a.out 1 2 3 |& vim -`

:setnowrap

fork的应用：

文件描述符：一个指向操作系统内对象的“指针”

* 对象只能通过OS允许的方式访问
* 从0开始编号（0，1，2分别是stdin, stdout, stderr）
* 可以通过open取得，close释放，dup复制
* 对于数据文件，文件描述符会记住上次访问文件的位置
  * write(2,"a", 1); write(3,"b", 1);

可执行文件：状态机的描述，一个描述了状态机的初始状态+迁移的数据结构
