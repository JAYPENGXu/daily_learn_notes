#### thrift_lesson

##### C++代码的编译： g++ -c main.cpp match_server/*.cpp

##### C++代码的链接： g++ *.o -o main 

##### C++ 代码的链接 ： g++ *.o -o main -lthrift ，加上thrift的动态链接库 （用这个）

##### C++ 代码的链接：如果用到了线程 g++ *.o -o main -lthrift -pthread

---

#### 命名空间

**namespace cpp match_server**

**namespace py match_client**

namespace 语言名称 自定义名称



#### 数据类型

thrift ------cpp

string  -   string

i32     -    int 

i64     -    long

i16 十六位整型

byte   --  char

bool   -- bool

double -- double



```thrift
struct User{
1: i32 id,
2: string name,
3: i32 score
}
```



#### 函数接口

其对所有的描述都放在 **service**中，service的名字可自定义。

```
service match_system{
i32 add_user(1: User user, 2: string info),
i32 remove_user(1: User user, 2: string info),
}
```

#### 服务端的建立

生成各种配置和连接文件，以及代码框架，在框架中实现自己的业务。

```
thrift -r --gen cpp ../../match_system
```



##### C++中锁的头文件、线程的头文件

```c++
#include "mutex"
#include "thread"
#include "condition_valiable" //条件变量的头文件
```



##### 获取一个字符串的md5值

md5sum -> 输入字符串 -> 回车后按ctrl + d







