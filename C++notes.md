#### C++ 基础语法操作

**数据类型：**

bool

int :2B

long :4B

long long: 8B

float : 4B

double : 8B

long double :16B

**字符的读入：**

```c++
scanf("%d%d", &a,&b); //会把空格读入
cin >> a >> b ; //会忽略中间的空格
```

**曼哈顿距离：** $d = |x_1 - x_2| + |y_1 - y_2|$

**浮点数比较：** $\sqrt{2} \times \sqrt{2}\quad!= 2$， 需使用 $|\sqrt{2} \times\sqrt{2} - 2| \le eps$ , $eps 一般取 1e-6$

**cstring常用函数用法：**

```c++
memset(f, -1, sizeof f);
memset(f, 0, sizeof f); 
//memset() 按字节赋值，只有赋值为0 和-1的时候才会符合预期
memset(f, 0x3f, sizeof f); // 0x3f3f3f3f
//--------------------------------------------------
memcpy(dst, src, sizeof src); //拷贝数组
```

**字符串读取：**

```c++
cin >> str; //不能读取含空格，换行符的字符串
getline(cin, str); //能读取含空格的字符串，同时自动去掉换行符\n

str.sbustr(begin, length);
str.pop_back(); //删除最后一个字符
```

**char数组难点：**

```c++
char a[] = {'c','+','+'}
char b[3] = {'p','y','\0'};

cout << a << endl; // 输出 "c++py" ,因为字符串a(a没有初始化长度，即使初始化了长度也需要+1的长度)不会自动添加'\0'，cout会读取到b的部分， 或者没有b会输出“c++”

//fgets()避坑， fgets()会读入 \n， 因此遍历字符串时用 for(i = 0, str[i] != '\n', i ++)
```

**结构体、类：**

```c++
//结构体
struct Node{
    int val;
    Node* next;
    
    //构造器
    Node(int _val):val(_val), next(NULL){} //初始化列表
};

Node* p = new Node(1);

//---------------------------------------
//类
class Node{
    private:
    
    public:
    
};
// 结构体默认是public, 类默认是private
```

**STL容器：**

```c++
vector<int> a; //一维数组
vector<vector<int>> a(rol, vector<int>(col)); //二维数组 rol行 col列

//优先队列
priority_queue<int> a; //默认大根堆
priority_queue<int, vector<int>, greater<int>> b; //小根堆

struct Rec{
    int a, b;
    
    //大根堆需要自定义类重载运算符 < 
    bool operator < (const Rec& t) const{
        return a < t.a;
    }
    
    //小根堆需要自定义重载运算符 > 
    bool operator > (const Rec& t) const{
        return a > t.a;
    }
}

priority_queue<Rec> a;  //大根堆
priority_queue<Rec, vector<Rec>, greater<Rec>> a;  //小根堆


//-------------
bitset s;
s.count(); // 1的个数
s.set(p);// 第 p位设为 1
s.reset(p); // 第p位设为0
    
```

**algorithm库：**

```c++
//记录常用操作
// 去重
unique(a, a + a.size()); // unique返回去重后最后一个元素的地址
int m = unique(a, a + a.size()) - a; //去重后数组的长度
a.erase(unique(a.begin(), a.end()), a.end())// 删除重复元素

//打乱
random_shuffle(a.begin(), a.end());
```

---

#### 《C++ primer》内容总结

`lambda`表达式形式：

`[capture list] (parameter list) -> return type {function body}`

capture list通常为空

不带参数: 

`auto f = [] {return 42;}`

带参数:

```c++
[] (const string &a, const string &b) {return a.size() < b.size()}
```

`constexpr`变量：c++11允许将变量声明为`constexpr`类型以便由编译器来验证变量的值是否是一个常量表达式。

把`引用`绑定到`const`对象上称之为对常量的引用，不能被用作修改它所绑定的对象。

```c++
int i = 42;
const int &r1 = i; //允许将const int &绑定到一个普通的int对象上
const int &r2 = 42;
const int &r4 = r1 + 2; //错误, r4是一个普通的非常量引用

const int &r2 = i; //r2也绑定对象i，但是不允许通过r2修改i的值
```

令指针指向常量或者非常量，指向常量的指针不能用于改变其所指对象的值。

```c++
const double pi = 3.14;
const double *cptr = &pi; //cptr可以指向一个双精度常量
*cptr = 42; //错误，不能给*ptr赋值
```

`const指针`，指针是对象，而引用不是。允许把指针本身定位常量，常量指针必须初始化，一旦完成初始化，它的值（存放在指针中的那个地址）就不能再改变了。 把`*`放在`const`关键字之前用以说明指针是一个常量。

```c++
int errNumb = 0;
int *const curErr = &errNumb; //curErr将一直指向errNumb （常指针）
const double *const pip = &pi; //pip是一个指向常量对象的常量指针
```

`顶层const`：表示<u>指针本身</u>是个常量； `底层const`：表示<u>指针所指的对象</u>是一个常量。

```c++
int *const p1 = &i; //不能改变p1的值，这是一个顶层const
const int *p2 = &ci; //允许改变p2的值，这是一个底层const
```

函数重载：同一作用域内的几个函数名字相同但形参列表不同。对于重载函数，他们应该在形参数量或形参类型上有所不同，不允许两个函数除了返回类型外其他所有的要素都相同。

`const_cast`在重载函数的情境下最有用。

右值引用：绑定到右值的引用，`&&` ，重要性质：只能绑定倒一个将要销毁的对象。

<u>左值表达式表示的是一个对象的身份</u>，<u>右值表达式表示的是对象的值</u>。左值与右值的根本区别在于是否允许取地址运算符获得对应的内存地址。

* 左值转化为右值，如整数变量`i`,在表达式（`i+3`）
* 数组名是常量左值，在表达式中转化为数组首元素的地址值
* 函数名是常量左值，在表达式中转化为函数的地址名

`字面常量是右值：1、2、88`

C++11引用绑定规则

* 非常量左值引用(X&):只能绑定到X类型的左值对象
* 常量左值引用（const X&）：可以绑定到X、const X类型的左值对象，或X、const X类型的右值。
* 非常量右值引用（X&&）：只能绑定到X类型的右值
* 常量右值引用（const X&&）：可以绑定到X、const X类型的右值

函数返回值是右值数据类型还是右值引用类型，区别在于前者是传值，而后者是传引用，可以修改被引用的对象：

```c++
#include <iostream>
#include <utility>

int i = 101, j = 101;

int foo(){ return i; }
int&& bar(){ return std::move(i); }
void set(int&& k){ k = 102; }
int main()
{
	foo();
	std::cout << i << std::endl;
	set(bar());
	std::cout << i << std::endl; 	 
}
out: 101、 102
```

完美转发：

~~这种实现的问题是不能支持移动语义，形参使用右值引用可以解决完美转发问题。~~

```c++
#include <iostream>
#include <utility>
#include <memory>

using namespace std;

template <typename T, typename Arg>
shared_ptr<T> factory(const Arg& arg)
{
    return shared_ptr<T> (new T(arg));
}

int main()
{
    auto x = factory<int>(22);
    cout << *x; 
}
out : 22
```

完美转发解决方案：其中`std::forward`是定义在c++11标准库中的模板函数

```c++
template <typename T, typename Arg> 
shared_ptr<T> factory(Arg&& arg)
{
    return shared_ptr<T>( new T(std::forward<Arg>(arg) ) );
}
```

模板参数类型推导：对函数模板 `template<typename T>void foo(T&&);`可推导出如下结论：

* 如果实参是类型A的左值，则模板参数T的类型为A&， 形参类型为A&；
* 如果实参是类型A的右值，则模板参数T的类型为A&&, 形参类型为A&&;

这同样适用于类模板的成员函数模板的类型推导：

```c++
template <class T> class vector{
    public:
    void push_back(T&& x); //T是类模板参数，该成员函数不需要类型推导，这里的函数参数类型就是T的右值引用
    template <class Args> void emplace_back(Args&& args);//该成员函数是个函数模板，有自己的模板参数，需要类型推导
}
```

---

`mutex`，lock_guard与mutex配合使用，把锁放到`lock_guard`中时，mutex自动上锁，lock_guard析构时，同时把mutex解锁，*lock_guard is an object that manages a mutex object by keeping it always locked* 。

```c++
std::mutex //该类表示普通的互斥锁，不能递归使用
std:: timed_mutex //该类表示定时互斥锁，不能递归使用

#include <iostream>
#include <thread>
#include <mutex>
#incldue <stdexcept>
    
std :: mutex mtx;

void print_even(int x){
    if(x % 2 == 0) std :: cout << x << "is even\n";
    else throw(std::logic_error("not even"));
}

void print_thread_id(int id){
    try{
        std :: lock_guard<std::mutex> lck(mtx);
        print_even(id);
    }
    catch(std::logic_error&){
        std::cout << "[exception caught]\n";
    }
}

int main(){
    std::thread threads[10];
    for(int i = 0; i < 10; i ++){
        threads[i] = std::thread(print_thread_id, i + 1);
    }
    for(auto &th : threads) th.join();
    return 0;
}
```



动态内存的管理通过一对运算符来完成：`new`、`delete`，动态内存的使用易出问题，因为确保在正确的时间释放内存是极其困难的，容易造成内存泄漏。`智能指针`：行为类似常规指针，重要的区别在于它负责自动释放所指向的对象，`shared_ptr`允许多个指针指向同一个对象，`unique_ptr`则独占所指向的对象，`weak_ptrs`是一种弱引用，指向`shared_ptr`所管理的对象。

```c++
shared_ptr<string> p1; //指向string
shared_ptr<list<int>> p2; //指向int的list
```

最安全的分配和使用动态内存的方法是调用一个名为`make_shared`的标准库函数。此函数在动态内存中分配一个对象并初始化它，返回指向此对象的`shared_ptr`。

```c++
shared_ptr<int> p3 = make_shared<int> (42); //指向一个值为42的int的shared_ptr

shared_ptr<string> p4 = make_shared<string> (10, '9'); //指向一个值为9999999999的string
```

---

```c++
void bar(std::unique_ptr<Entity> e){
    // bar owns e.
    // e will be automatically destroyed.
}
var foo(){
    auto e = std::make_unique<Entity>();
    e->DoSomething();
    bar(std::move(e));
}
foo();
//No memory leak.
```

<img src="G:\typora_image_store\image-20221005205311643.png" alt="image-20221005205311643" style="zoom:53%;" />

<img src="G:\typora_image_store\image-20221005205742340.png" alt="image-20221005205742340" style="zoom:50%;" />

<img src="G:\typora_image_store\image-20221005205959069.png" alt="image-20221005205959069" style="zoom:50%;" />



<img src="G:\typora_image_store\image-20221005210055013.png" alt="image-20221005210055013" style="zoom:53%;" />

虚函数：必须为每一个虚函数都提供定义，而不管它是否被用到了，这是因为连编译器也无法确定到底会使用哪个虚函数。动态绑定只有当我们通过指针或者引用调用虚函数时才会发生。在c++11中我们可以使用`override`关键字来说明派生类中的虚函数。

函数重载：如果同一作用域内的几个函数名字相同，但形参列表不同，我们称之为重载函数（`overloaded`）。

`explicit`关键字：用来修饰类的构造函数，被修饰的构造函数的类，不能发生相应的隐式类型转换，只能以显示的方式进行类型转换。只能作用域类内部的构造函数声明上。

迭代器：所有标准库容器都可以使用迭代器，string对象不属于容器类型，string支持迭代器。`begin`、`end`，end指向尾元素的下一个位置，如果容器为空，则begin和end返回的是同一个迭代器，都是尾后迭代器。begin和end返回的举体类型由对象是否为常量决定，如果对象为常量，则返回`const_iterator`;如果对象不是常量，返回`iterator`。

```c++
vector<int>::iterator it; //it能读写vector<int>的元素
string:: iterator it2; //it2能读写string对象中的元素
vector<int>::const iterator it3; //it3只能读元素，不能写元素
string::const iterator it4; //it4只能读元素，不能写元素
```

`->`：解引用和成员访问两个操作结合在一起。

如果只是读取元素，而不存在写操作，使用`cbegin`、`cend`来控制整个迭代过程。

函数模板，`template <typename T>`

拷贝控制操作：

一个类通过定义五种特殊的成员函数来控制这些操作，包括：**拷贝构造函数**、**拷贝赋值函数**、**移动构造函数**、**移动赋值函数**、**析构函数**，

* 拷贝和移动构造函数定义了当用同类型的额另一个对象初始化本对象时做什么。

* 拷贝和移动赋值运算符定义了将一个对象赋予同类型的另一个对象时做什么。

* 析构函数定义了当此类型对象销毁时做什么。

拷贝构造函数：如果一个构造函数的<u>第一个参数是自身类类型的引用</u>，且任何额外参数都有默认值，则此构造函数是拷贝构造函数。

```c++
class Foo{
    public:
    	Foo();  //默认构造函数
    	Foo(const Foo&); //拷贝构造函数
}
```

使用`=default`：我们可以通过将拷贝控制成员定义为=default来显示的要求编译器生成合成的版本

```c++
class Sales_data{
    public:
    	Sales_data() = default;
    	Sales_data(const Sales_data) = default;
    	~Sales_data() = default;
}
```

`volatile`关键字：遇到这个关键字声明的变量，编译器访问该变量的代码就不再进行优化，即编译器生成的汇编代码会重新从变量的地址读取数据，从而实现提供对特殊地址的稳定访问。[参考链接](https://www.runoob.com/w3cnote/c-volatile-keyword.html)

线程：c++11提供的线程类 `std::thread`

命名空间 `std::this_pthread`,此命名空间提供四个公共的成员函数，

`get_id()`

```c++
thread::id get_id() noexcept;
```

`sleep_for()` ：线程会从运行态变成阻塞态并在这种状态下休眠一定的时长

线程被创建出来有五种状态：创建态、就绪态、阻塞态、运行态、退出态；

```c++
template <class Rep, class Period>
    void sleep_for(const chrono::duration<Rep, Period>&rel_time);
```

`sleep_until`:指定线程阻塞到某一时间点**time_point类型**，之后解除

```c++
template <class Clock, class Duration>
    void sleep_until(const chrono::time_point<Clock, Duration>& abs_time);
```

`yield()` 线程使用了yield之后会放弃CPU资源，但是这个变为就绪态的线程会马上参与到下一轮CPU资源的争夺中，不排除其会继续抢到CPU时间片的情况

```c++
void yield() noexcept;
```

C++线程类：

默认构造函数、移动构造函数、创建线程构造对象；

```c++
thread() noexcept; //默认构造,在此线程中不执行任何处理动作
thread(thread && other) noexcept; //移动构造,将 other的线程所有权转移给新的thread对象，之后other不在便是执行线程
template<class Function, class ..Args> //创建线程对象
explicit thread (Function&& f, Args&& ..args);
```

`get_id()` 获取线程ID的函数，

当启动一个线程后，回收线程资源，thread库提供两种方式：

`join()` ：调用此函数的线程会被阻塞，但是子线程对象中的任务函数会继续执行。当任务执行完毕之后join()函数会清理当前子线程中的相关资源后返回，同时该线程函数会解除阻塞继续执行。

`detach()` ：进行线程分离，分离主线程和子线程。在线程分离之后，主线程退出也会销毁创建的所有子线程，在主线程推出之前，子线程可以脱离主线程继续独立运行，任务结束完毕之后，这个子线程会自动释放自己占用的系统资源。

fork系统调用： `man fork`

在父进程中返回的是子进程的pid，在子进程中返回0。

`wait`函数将阻塞进程，直到该进程中的某个子进程结束运行为止，它返回结束运行的子进程的pid。

`waitpid`只等待由pid参数指定的子进程，如果pid取值为-1，那么它和wait函数相同，即等待任意一个子进程结束。

线程同步方式：信号量、互斥锁、条件变量；

`pthread_create()`:创建线程,成功返回0。

```c++
int pthread_create(pthread_t *tidp, const pthread_attr_t *attr, (void *)(*start_rtn)(void *), void *args); //pthread是一个整型，attr用于设置新线程的属性，start_rtn指定新线程将运行的函数，arg函数的参数。
```

`pthread_exit()`:确保安全干净的退出。

```c++
void pthread_exit(void *retval); //通过retrval参数向线程的回收者传递其退出信息。
```

`pthread_join()`:回收其他线程，即等待其他线程结束，成功返回0.

```c++
int pthread_join(pthread_t thread, void **retval); //等待其他线程结束，该函数会一直阻塞，直到被回收的线程结束为止。
```

`pthread_cancel()`:取消线程， 成功返回0。

```c++
int pthread_cancel(pthread_t thread);//thread参数是目标线程的标识符。
```

线程属性：`pthread_attr_t`结构体定义了一套完整的线程属性；

```c++
#define __SIZEOF_PTHREAD_ATTR_T 36
typedef union{
    char __size[__SIZEOF_PTHREAD_ATTR_T];
    long int __align;
}pthread_attr_t;
```

POSIX信号量：线程同步的机制：POSIX信号量、互斥量、条件变量

POSIX信号量函数名字以**sem_**开头，常用的POSIX信号量函数如下5个：

```c++
#include <semaphore.h>
int sem_init(sem_t *sem, int pshared, unsigned int value); //初始化一个未命名的信号量
int sem_destroy(sem_t *sem);//销毁信号量，释放其占用的内核资源
int sem_wait(sem_t *sem); //以原子操作的方式将信号量的值减1，如果信号量的值为0，则该函数被阻塞
int sem_trywait(sem_t *sem);//与wait相似，始终立即返回。
int sem_post(sem_t *sem); //以原子操作的方式将信号量的值加1
```

互斥锁：互斥锁的类型是pthread_mutex_t结构体；

`pthread_mutex_init()`:初始化互斥锁

`pthread_mutex_destory()`:销毁互斥锁

`pthread_mutex_lock()`:以原子操作的方式给一个互斥量加锁

`pthread_mutex_trylock()`:始终立即返回，不论被操作对象的互斥锁是否已经被加锁，相当于非阻带版本。

`pthread_mutex_unlock()`:以原子操作的方式给一个互斥锁解锁

以上五个函数，成功返回0。

互斥锁属性：`pthread_mutexattr_t`结构体定义了互斥锁属性，线程库中提供了一系列函数来操作pthread_mutexattr_t类型的变量，主要函数包括以下：

```c++
#include <pthread.h>
int pthread_mutexattr_init(pthread_mutexattr_t *attr);
int pthread_mutexattr_destroy(pthread_mutexattr_t *attr);
int pthread_mutexattr_getpshared(const pthread_mutexattr_t *attr, int *pshared);
int pthread_mutexattr_setpshared(pthread_mutexattr_t *attr,int pshared);
int pthread_mutexattr_gettype(const pthread_mutexattr_t *attr, int *type);
int pthread_mutexattr_settype(pthread_mutexattr_t *attr, int type);
```

pshared指定是否允许跨进程共享互斥锁，其取值包含`PTHREAD_PROCESS_SHARED`(互斥锁可以被跨进程共享), `PTHREAD_PROCESS_PRIVATE`(互斥锁只能被和锁的初始化线程隶属于同一个进程的线程共享)。



```c
//c代码      <---> c++代码
pthread_self() = getpid()
pthread_create() = fork()
pthread_cancel() = kill()
pthread_join() = wait()
pthread_exit()
pthread_detach()
pthread_equal()
```

C中的`typeof`关键字：

```c
extern int foo();
typeof(foo()) var;  //声明了一个整型的变量var
typeof(int *) a, b; //等价于 int *a, int *b;
```

`__thread`是GCC内置的用于多线程编程的基础设施，用`__thread`修饰的变量，每个线程都拥有一份实体，相互独立。

```c
#include<iostream>  
#include<pthread.h>  
#include<unistd.h>  
using namespace std;
__thread int i = 1;
void* thread1(void* arg);
void* thread2(void* arg);
int main()
{
  pthread_t pthread1;
  pthread_t pthread2;
  pthread_create(&pthread1, NULL, thread1, NULL);
  pthread_create(&pthread2, NULL, thread2, NULL);
  pthread_join(pthread1, NULL);
  pthread_join(pthread2, NULL);
  return 0;
}
void* thread1(void* arg)
{
  cout<<++i<<endl;//输出 2  
  return NULL;
}
void* thread2(void* arg)
{
  sleep(1); //等待thread1完成更新
  cout<<++i<<endl;//输出 2，而不是3
  return NULL;
}
```

`memory barrier`：防止乱序执行，包括三种；编译器只需保证单线程内的一致性即可，中间过程的任意优化是允许的。

1. `acquire barrier`。 其之后的指令不能也不会被拉到该acquire barrier之前执行，对应 lock()。
2. `release barrier`。其之前的指令不能也不会被拉到该release barrier之后执行，对应unlock()。
3. `full barrier`。以上两种的合集。

`__sync_synchronize`:一种full barrier。
