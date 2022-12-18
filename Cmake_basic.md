CMake基础

**PROJECT关键字：**

可以用来指定工程的名字和支持的语言，默认支持所有语言

```cmake
PROJECT(HELLO)
PROJECT(HELLO CXX) #指定工程名字，并且支持语言为C++
PROJECT(HELLO C CXX) #指定工程名字，并且支持语言为C和C++
```

**SET关键字：**

```cmake
SET(SRC_LIST main.cpp) #用来显示的指定变量 ，SRC_LIST变量就包含了main.cpp
SET(SRC_LIST main.cpp test.cpp)
```

**MESSAGE关键字：** 

向终端输出用户自定义的信息；主要包含三种： SEND_ERROR产生错误，生成过程被跳过，STATUS输出前缀为-的信息； FATAL_ERROR立即终止所有的cmake过程。

```cmake
MESSAGE(STATUS "This is BINARY dir " ${HELLO_BINARY_DIR})   
MESSAGE(STATUS "This is SOURCE dir "${HELLO_SOURCE_DIR})
```

**ADD_EXECUTABLE关键字：**

```cmake
ADD_EXECUTABLE(hello ${SRC_LIST}) #生成的可执行文件名是hello，源文件读取变量SRC_LIST中的内容，也可以直接写ADD_EXECTABLE(hello main.cpp)
```

一个示例：

```cmake
PROJECT (HELLO)     #HELLO 项目名
SET(SRC_LIST main.c)
#STATUS 前面有-- 的消息到终端
MESSAGE(STATUS "This is BINARY dir " ${HELLO_BINARY_DIR})   
MESSAGE(STATUS "This is SOURCE dir "${HELLO_SOURCE_DIR})
ADD_EXECUTABLE(hello ${SRC_LIST})    #hello 文件名

#可以将本例写成更简单的
PROJECT(HELLO)
ADD_EXECUTABLE(hello main.c)
```

**ADD_LIBRARY关键字：**

```cmake
ADD_LIBRARY(libanswer STATIC answer.cpp) #添加libanswer库文件，STATIC指定为静态库
```

**TARGET_LINK_LIBRARIES关键字：**

```cmake
TARGET_LINK_LIBRARIES(answer libanswer) //为answer可执行目标链接 libanswer
```

**FIND_PACKAGE关键字：**

```cmake
FIND_PACKAGE(CURL REQUIRED) #用于在系统中群招已经安装的第三方库的头文件和库文件的位置，并创建一个名为 CRUL::libcurl 的库文件，以供链接。
```

**基本语法规则：**

1，变量使用${}方式取值，但是在 IF 控制语句中是直接使用变量名
2，指令(参数 1 参数 2...)
参数使用括弧括起，参数之间使用空格或分号分开。
以上面的 ADD_EXECUTABLE 指令为例，如果存在另外一个 func.c 源文件，就要写成：
ADD_EXECUTABLE(hello main.c func.c)或者
ADD_EXECUTABLE(hello main.c;func.c)
按一个风格写就可以
3，指令是大小写无关的，参数和变量是大小写相关的。但，推荐你全部使用大写指令。

**外部构建：**

会把生成的临时文件放在build目录下，不会对源文件有任何影响，推荐使用外部构建

```cmake
#1.新建build目录
#2. cd build
#3. cmake ..
#4. make
```

**ADD_SUBDIRECTORY关键字：**

用于向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置

```cmake
ADD_SUBDIRECTORY(src bin) #将src子目录加入工程并指定编译输出路径为bin目录，如果不进行bin目录的指定，那么编译产生的文件都将存放在build/src目录
```

**EXCLUDE_FROM_ALL关键字：**

将写的目录从编译中排除

**TARGET_COMPILE_FEATURES关键字：**

可以针对target要求编译feature （即指定要使用C/C++的什么特性）

```cmake
target_compile_features(libanswer INTERFACE cxx_std_20)
```



----

**Makefile**

* Makefile is notoriously picky about the use of `Space` instead of `Tab`.
* you can fix this by changing the `Spaces` to actual `Tab` characters. -> vim Makefile -> `:%s/^[ ]\+/^I/`

```makefile
hello:main.cpp
	$(CXX) -o hello main.cpp
	echo "done"
```

改进

```makefile
CC := clang
CXX := clang++ #可通过make CXX=g++形式覆盖
objects := main.o

#使用变量
hello : $(objects)  #hello 是我们最终要生成的可行性文件名，它依赖objects中的所有目标文件
	$(CXX) -o $@ $(objects) # $@是自动变量，表示target名
	
#main.o 目标文件依赖 main.cpp	
main.o: main.cpp  
	$(CXX) -c main.cpp
	
.PHONY:clean
clean :
	rm -f hello $(objects)
```

改进

```makefile
CC := clang
CXX := clang++

.PHONY : all
all : answer
objects := main.o answer.o

answer : $(objects)
		$(CXX) -o $@ $(objects)
		
# make 可以自动推断 .o目标文件需要依赖同名的 .cpp文件，
#所以不需要在依赖中指定main.cpp和answer.cpp
#也不需要编译commands，他知道要用CXX变量指定的命令作为C++的编译器
# 这里只需要指定目标文件所依赖的头文件，使头文件变动时可以
# 重新编译对应目标文件。
main.o : answer.hpp
answer.o : answer.hpp

.PHONY: clean
clean: 
		rm -f answer $(objects)
		
# .PHONY ->  不是一个文件就用这个，伪目标
```

很难用，改进☞使用cmake,创建CMakeList.txt

`ccmake  -B build`  带图形化展示界面
