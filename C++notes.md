#### C++ 基础语法操作

**数据类型：**

boo

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

