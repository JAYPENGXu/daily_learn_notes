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
