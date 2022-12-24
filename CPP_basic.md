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

`右值引用`：绑定到右值的引用，`&&` ，重要性质：只能绑定倒一个将要销毁的对象。

右值也可以出现在赋值表达式的左边，但是不能作为赋值的对象，因为右值只在当前语句有效，赋值没有意义。`((i > 0) ? i : j) =1;` [参考链接](https://murphypei.github.io/blog/2018/08/cpp-right-reference.html#more)

```c++
void process_value(int &i){
    std :: cout << "Lvalue processed:" << i < std :: endl;
}

void process_value(itn && i){
    std :: cout<< "Rvalue processed:" << i << std:: endl;
}

int main(){
	int a = 0;
    process_value(a);
    provess_value(1);
}
/*
LValue processed: 0 
RValue processed: 1
*/
```

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

`转移语义`：可以将资源（堆、系统对象等）从一个对象转移到另一个对象，这样能够减少不必要的临时对象的创建、拷贝和销毁。

`std::move`:既然**编译器只对右值引用才能调用转移构造函数和转移赋值函数**，而**所有命名对象都只能是左值引用**，如果已知一个命名对象不再被使用而想对它调用转移构造函数和转移赋值函数，也就是**把一个左值引用当做右值引用来使用**，怎么做呢？将左值引用转换为右值引用。

```c++
void ProcessValue(int& i) { 
 std::cout << "LValue processed: " << i << std::endl; 
} 
 
void ProcessValue(int&& i) { 
 std::cout << "RValue processed: " << i << std::endl; 
} 

int main(){
    int a = 0;
    ProcessValue(a);
    ProcessValue(std::move(a));   
}
/*
LValue processed: 0 
RValue processed: 0
*/
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

`std::mutex`

用于保护共享数据不被多个线程同时访问

```c++
#include <iostream>
#include <chrono>
#include <thread>
#include <mutex>
 
int g_num = 0;  // protected by g_num_mutex
std::mutex g_num_mutex;
 
void slow_increment(int id) 
{
    for (int i = 0; i < 3; ++i) {
        g_num_mutex.lock();   //上锁
        ++g_num;
        // note, that the mutex also syncronizes the output
        std::cout << "id: " << id << ", g_num: " << g_num << '\n';
        g_num_mutex.unlock();  //解锁
 
        std::this_thread::sleep_for(std::chrono::milliseconds(234));
    }
}
 
int main()
{
    std::thread t1{slow_increment, 0};
    std::thread t2{slow_increment, 1};
    t1.join();
    t2.join();
}
```

成员函数： `lock(), unlock(), trylock(),`

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

`try_lock()`:

```c++
#include <chrono>
#include <mutex>
#include <thread>
#include <iostream> // std::cout
 
std::chrono::milliseconds interval(100);
 
std::mutex mutex;
int job_shared = 0; // both threads can modify 'job_shared',
    // mutex will protect this variable
 
int job_exclusive = 0; // only one thread can modify 'job_exclusive'
    // no protection needed
 
// this thread can modify both 'job_shared' and 'job_exclusive'
void job_1() 
{
    std::this_thread::sleep_for(interval); // let 'job_2' take a lock
 
    while (true) {
        // try to lock mutex to modify 'job_shared'
        if (mutex.try_lock()) {
            std::cout << "job shared (" << job_shared << ")\n";
            mutex.unlock();
            return;
        } else {
            // can't get lock to modify 'job_shared'
            // but there is some other work to do
            ++job_exclusive;
            std::cout << "job exclusive (" << job_exclusive << ")\n";
            std::this_thread::sleep_for(interval);
        }
    }
}
 
// this thread can modify only 'job_shared'
void job_2() 
{
    mutex.lock();
    std::this_thread::sleep_for(5 * interval);
    ++job_shared;
    mutex.unlock();
}
 
int main() 
{
    std::thread thread_1(job_1);
    std::thread thread_2(job_2);
 
    thread_1.join();
    thread_2.join();
}
```

`std::ref` ：用于取某个变量的引用，这个引入是为了解决一些传参问题。

`std::bind` :使用的是参数的拷贝而不是引用，因此，必须显示利用`std::ref`来进行绑定。[参考链接](https://murphypei.github.io/blog/2019/04/cpp-std-ref.html#more)

`std::lock_guard`

是一个mutex包装器，用于在作用域块的持续时间内拥有metex，当一个lock_gurad对象被创建时，它会尝试获取给它的互斥锁的所有权，当控制权离开创建lock_guard对象的范围时，lock_guard被销毁，同时mutex被释放。

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

c++11 关键字：thread_local

c++有四种存储周期：`automatic` 、`static`、 `dynamic`、 `thread`、`register`、 `mutable`、

有且只有`thread_local`关键字修饰的变量具有thread周期，这些变量在线程开始的时候被生成，在线程结束的时候被销毁，并且每一个线程都拥有一个独立的变量实例。[参考链接](https://zhuanlan.zhihu.com/p/77585472)、[参考链接2](https://murphypei.github.io/blog/2020/02/thread-local)

`register`:自动存储期，提示编译器将此变量置于寄存器中。

线程：c++11提供的线程类 `std::thread`

`std::thread t1(fun(), id);`

线程允许多个函数同时执行

成员函数：`joinable(), get_id(), native_handle()`,|| `join(), detach(), swap();`

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

C++线程类：std::thread:

线程允许多个函数同时执行，线程在构造相关线程对象后立即开始执行；

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

```c++
#include <iostream>
#include <thread>
#include <chrono>
 
void foo()
{
    // simulate expensive operation
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
void bar()
{
    // simulate expensive operation
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
int main()
{
    std::cout << "starting first helper...\n";
    std::thread helper1(foo);
 
    std::cout << "starting second helper...\n";
    std::thread helper2(bar);
 
    std::cout << "waiting for helpers to finish..." << std::endl;
    helper1.join();
    helper2.join();
 
    std::cout << "done!\n";
}
```

`detach()` ：进行线程分离，分离主线程和子线程。在线程分离之后，主线程退出也会销毁创建的所有子线程，在主线程退出之前，子线程可以脱离主线程继续独立运行，任务结束完毕之后，这个子线程会自动释放自己占用的系统资源。

```c++
#include <iostream>
#include <chrono>
#include <thread>
 
void independentThread() 
{
    std::cout << "Starting concurrent thread.\n";
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "Exiting concurrent thread.\n";
}
 
void threadCaller() 
{
    std::cout << "Starting thread caller.\n";
    std::thread t(independentThread);
    t.detach();  //
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Exiting thread caller.\n";
}
 
int main() 
{
    threadCaller();
    std::this_thread::sleep_for(std::chrono::seconds(5));
}
```

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

创建socket:

```c++
#include <sys/types.h>
#include <sys/socket.h>
int socket(int domain, int type, int protocol);
```

domain：使用哪个地城协议族；type指定服务类型，主要有SOCK_STREAM(流服务) 和SOCK_UGRAM（数据报）服务。对TCP/IP协议族而言，取SOCK_STREAM表示传输层使用TCP协议，取SOCK_UGRAM表示传输层使用UDP协议。

protocol表示在前两个参数构成的协集合下，再选择一个具体的协议。通常设置为0。

socket系统调用成功返回一个socker文件描述符，失败返回-1，并设置errno。

将一个socket与socket地址绑定称为给socket命名。只有命名后客户端才能知道该如何连接它。客户端通常不需要命名，而是采用匿名方式，命名socket的系统调用是bind（）：

```c++
#include <sys/types.h>
#include <sys/socket.h>
int bind(int sockfd, const struct scokaddr *myaddr, socklen_t addrlen);
```

bind将my_addr所指向的socket地址分配给未命名的sockfd文件描述符，addrlen值除该socket地址的长度。

socket被命名之后，还需要创建一个监听队列以存放待处理的客户连接：

```c++
#include <sys/socket.h>
int listen(int sockfd, int backlog);
```

sockfd指定被监听的socket， backlog提示内核监听队列的最大长度。

**Linux下实现IO复用：select、 poll、 epoll**

服务器要管理多个客户端的连接，而recv函数只能监听单个socket，因此引入了select/poll。

`select`系统调用：在一段指定时间内，监听用户感兴趣的文件描述符上的可读、可写和异常等事件。

```c++
#include <sys/select.h>
int select (int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
// 参数解释： nfds:被select管理的描述符个数， fd_set：表示一组描述符集合，select中是用一个位数组来实现， redset、writeset、exeptset ：可读、可写、异常事件集合， timeout：超时时间
//------其他接口定义
void FD_ZERO(fd_set *fdset); // clear all bits in fdset
void FD_SET(int fd, fd_set *fdset); // turn on the bit for fd in fdset
void FD_CLR(int fd, fd_set *fdset); // turn off the bit for fd in fdset
int FD_ISSET(int fd, fd_set *fdset); // is the bit for fd on fdset?
```

`poll`系统调用：在指定时间内轮询一定数量的文件描述符，以测试其中是否有就绪者。

```c++
#include <sys/poll.h>
int poll (struct pollfd *fds, nfds_t nfds, int timeout);
//  参数解释：fds:传入的pollfd数组的首地址， nfds：传入位fds数组的长度， timeout：超时时间
struct pollfd{
    int fd;
    short events;
    short revents; 
}
```

`select`和`poll`的区别:

|                            select                            |                             poll                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|            底层采用位数组实现，一个描述符对应一位            | 底层通过pollfd结构体实现，管理的描述符通过pollfd数组来组织，一个描述符对应一个pollfd对象 |
|           能监听的fd个数默认大小是FD_SETSIZE(1024)           |                       采用变长数组管理                       |
| 相同点：二者在调用时都需要从用户态拷贝管理的全量描述符到内核态，返回时都从 | 内核态拷贝全量的描述符到用户态，再由用户态遍历全量的描述符判断哪些描述符有就绪事件。 |

`epoll`是linux特有的IO复用函数，是linux内核为了处理大批量的文件描述符而改进的`poll`，可以显著提高程序在大量并发连接中只有少量活跃的的情况下的系统的CPU利用率。其使用一组函数来完成任务，而不是单个函数，其次，`epoll`把用户关心的文件描述符上的事件放在内核里的一个事件表中，从而无需像select和poll那样每次调用都要重复传入文件描述符或事件集。但`epoll`需要使用一个额外的文件描述符，来唯一标识内核中的这个事件表。这个文件描述符使用如下`epoll_create`函数来创建：

```c++
#include <sys/epoll.h>
int epoll_create(int size);
// linux2.6.8之后，size参数已被忽略，但必须>0; epoll_create()创建返回后的epollfd指向内核中的一个epoll实例，同时该epollfd用来调用所有和epoll相关的接口(epoll_ctl(), epoll_wait());当epollfd不再使用时，需要调用close()函数关闭。当所有指向epoll的文件描述符关闭后，内核会摧毁该epoll实例并释放和其关联的资源；失败返回-1；在内核中分配一段空间，并初始化管理监听描述符的数据结构：红黑树、就绪事件链表

int epoll_ctl(int epfd, int op, int fd, struct epoll_event * event);
// epfd：通过epoll_create()创建的epollfd； op: epoll_ctl_add()、 epoll_ctl_mod()、 epoll_ctl_del(); fd:待监听的描述符fd； event:要监听的fd的事件。（将哪个客户端fd的哪些事件event交给哪个epoll来管理）；暴漏给上层用户的对底层红黑树的增删改接口

int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);
//可以就绪事件链表中获取就绪时间关联的描述符，然后填充到events中并返回给上层用户
// epfd:待处理的epoll实例描述符；events:用于存放epoll_event的数组指针；maxevents：最长返回多少个时间至events数组中，该参数须大于0；timeout:最长等待/阻塞时间（毫秒），置为-1将导致该函数无期限阻塞，置为0函数立即返回，而不管是否存在可用事件。该时间基于CLOCK_MONOTONIC时钟测量。

int epoll_pwait(int epfd, struct epoll_event *events, int maxevents, int timeout, 
               const sigset_t *sigmask); //不同于epoll_wait的是可以指定忽略部分信号
//执行成功，返回处于就绪状态的文件描述符个数。
// 如果等待超时，返回0， 表示没有处于就绪状态的fd。
//发生错误，返回-1并设置errno。

typedef union epoll_data {
    void *ptr;
    int fd;
    uint32_t u32;
    uint64_t u64;
} epoll_data_t;

struct epoll_event {
    uint32_t events;  // epoll events
    epoll_data_t data; // user data variable
};
```

`epoll`所支持的`fd`上限是最大可以打开的数目，具体可`cat/proc/sys/fs/file-max`查看。  当存在部分活跃socket时，传统select/poll会线性扫描整个socket集合，这会让效率随着socket数量的增加而线性降低，而这就限制了select最大监视数量的原因，程序被唤醒后不知道哪个socker处于活跃状态，需要遍历。而`epoll`是基于每个fd上面的`callback`函数实现的，云此只会对活跃的socket进行处理，效率更高。

select、poll、epoll之间的区别：

`select`:只知道有I/O时间发生，但不知道是具体哪一个，因此只能无差别的遍历有所流，时间复杂度O(n).

`poll`:大致同上，轮询所有套接字，时间复杂度O(n)，但fd没有数量限制，因为它是用链表存储fd。

`epoll`:时间驱动，epoll会把哪个流发生了怎样的I/O时间通知给用户，时间复杂度O(1)。

epoll的ET（边缘触发）模式和LT（水平触发）模式：

|                ET                |                            LT                            |
| :------------------------------: | :------------------------------------------------------: |
| 仅当监控的描述符有事件就绪时触发 | 当监控的描述符有时间iuxu或就绪事件未完全处理完时都会触发 |
|         系统调用次数更少         |                     系统调用次数更多                     |
|   数据完整性交由上层用户态保证   |       数据完整性交由内核来保证，epoll默认是LT模式        |

网络模型总结：

| 开源组件  | epoll触发模式 |         网络模型          |
| :-------: | :-----------: | :-----------------------: |
|   redis   |      LT       |    Single-Reactor模型     |
|  skynet   |      LT       |    Single-Reactor模型     |
| memcached |      LT       | Multi-Reactor(多线程)模型 |
|   nginx   |      ET       | Multi-Reactor(多进程)模型 |
|   netty   |      ET       | Multi-Reactor(多线程)模型 |

long long 是一个有符号类型，对应的无符号类型 unsigned long long；

long long int == long long

unsigned long long == unsigned long long

C++ 标准还为其定义LL 和 ULL 作为两种类型的字面量后缀 -> long long x = 65536LL;

long long 用于枚举和位域

```c++
enum longlong_enum : long long {
	x1,
	x2
}
```

```c++
struct longlong_struct {
	long long x1 : 8;
	long long x2 : 24;
	long long x3 : 32;
}
```

C++11标准为三种编码提供了新前缀用于声明三种编码字符和字符串的字面量，分别是UTF-8的前缀u8,UTF-16的前缀u，和UTF-32的前缀U。

```c++
char uft8c = u8'a';
char16_t utf16c = u'好';
char32_t utf32c = U'好';
char utf8[] = u8"你好世界";
char16_t utf16[] = u"你好世界";
char32_t utf32[] = U"你好世界";
```

char uft8c = u8'a'; 在C++11中编译报错，由于u8只能作为字符串字面量的前缀，而无法作为字符的前缀。

char utf8c = u8"好"编译报错，存储"hao"需要三个字节，显然utf8c只能存储1字节。
