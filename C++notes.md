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

