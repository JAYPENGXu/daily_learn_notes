### Linux基础操作

##### 管道：类似文件重定向，可以将前一个命令的stdout重定向到下一个命令的stdin， 仅能处理stdout，会忽略stderr。

##### 文件重定向：左边为命令，右边为文件。

```shell
objdump -d myprog > output
# >是标准输出重定向符号, 可以将前一命令的输出重定向到文件output中. 这样, 你就可以使用文本编辑工具查看output了.
objdump -d myprog | tee output
# 如果你希望输出到文件的同时也输出到屏幕上, 你可以使用tee命令:

> empty                  # 创建一个名为empty的空文件
cat old_file > new_file  # 将文件old_file复制一份, 新文件名为new_file
#使用输出重定向还能很方便地实现一些常用的功能, 例如

# <是输入重定向，可以将前一命令的输入重定向到文件data中. 这样, 你只需要将myprog读入的数据一次性输入到文件data中, myprog就会从文件data中读入数据, 节省了大量的时间.
./myprog < data

time ./myprog < data | tee output
#这个命令在运行myprog的同时, 指定其从文件data中读入数据, 并将其输出信息打印到屏幕和文件output中. time工具记录了这一过程所消耗的时间, 最后你会在屏幕上看到myprog运行所需要的时间.
```

e g. : 统计当前目录下所有python文件的总行数

```shell
find . -name '*.py' | xargs cat | wc -l

xargs #将stdin中的数据用空格或回车分割成命令行参数

wc -l #统计行数
wc -w #统计单词数
wc -c #统计字节数
```

**统计磁盘使用情况(统计/usr/share目录下各个目录所占用的磁盘空间)**

```shell
du -sc /usr/share/* | sort -nr
# du是磁盘空间分析工具, du -sc将目录的大小顺次输出到标准输出, 继而通过管道传送给sort. sort是#数据排序工具, 其中的选项-n表示按照数值进行排序, 而-r则表示从大到小输出. sort可以将这些参数连#写在一起.

du -sc /usr/share/* | sort -nr | more
#然而我们发现, /usr/share中的目录过多, 无法在一个屏幕内显示. 此时, 我们可以再使用一个命令: more或less. q键退出
```



##### 环境变量：类似全局变量，可以被各个进程访问到，可以通过修改环境变量来方便的修改系统配置。

```shell
env #显示当前用户的变量
set #显示当前shell的变量，包括当前用户的变量
export #显示当前导出成用户的shell变量

echo $PATH
```

常见环境变量：

```shell
HOME: #用户的家目录
PATH: #可执行文件的存储路径，路径与路径之间用:分隔，当某个可执行文件同时出现在多个路径中时，会选择从左到右数第一个路径中的执行。
LD_LIBRARY_PATH: #用于指定动态链接库(.so文件)的路径，其内容是以冒号分隔的路径列表。
C_INCLUDE_PATH: #C语言的头文件路径，内容是以冒号分隔的路径列表。
CPLUS_INCLUDE_PATH: #CPP的头文件路径，内容是以冒号分隔的路径列表。
PYTHONPATH: #Python导入包的路径，内容是以冒号分隔的路径列表。
JAVA_HOME: #jdk的安装目录。
CLASSPATH: #存放Java导入类的路径，内容是以冒号分隔的路径列表。
```

**环境变量的修改**：为了将对环境变量的修改持久化，可以将修改命令放到  ~/.bashrc文件中，之后执行 source ~/.bashrc 

##### chmod 777 代表： 自己、同组、其他能不能读写执行 -rwx-rwx-rwx == 777

```shell
#搜索文件
find *.py
find *.cpp


grep ***
#从标准输入中读东西，如果包含***则将***高亮	

#组合用法
find thrift_lesson2/ -name *.cpp | xargs cat | grep 'thrift' #其有一个弊端，不能显示当前内容出自哪一个文件

#通过ag命令可以实现
ag 'thrift'
```

`cut 分割一行内容，从标准输入中读，echo $PATH |cut -d ':' -f 1，输出PATH用：分割后第1个数据`

`echo $PATH | cut -c 3,5`：输出PATH的第3、5个字符

`more`: more main.cpp,回车下一行，空格下一页，b上一页

`less`:与more类似，回车下一行，y上一行，pagedown下一页，pageup上一页

`head -3 xxx`

`tail -3 xxx`

`history`：查看当前用户的所有操作记录

`md5sum` 计算md5哈希值 md5sum main.cpp

`sha1sum` :输出一个文件的SHA1哈希值

`time command` 统计command 命令的执行时间

`time free`

`watch -n 0.1 command` :每0.1秒执行一次command命令

`watch -n 0.1 free`

`tar -zcvf xxx.tar.gz /path/to/file/*` 压缩文件

`tar -zcvf rrr.tar.gz /tmp/*`

`tar-zxvf xxx.tar.gz` 解压文件 

`tar -zxvf rrr.tar.gz`

`diff xxx yyy` 查找xxx 和 yyy文件的不同点

`pip install xxx --user --upgrade`:安装python包

`^E` *means* `ctrl+E`

`alias`：可以为命令设置别名

`alias matlab="/usr/local/Polyspace/R2021a/bin/matlab"`

`source ~/.bashrc`

`gcc -g main.c -o main` 编译main.c文件 -g保留调试信息

`g++ thread.cpp -lpthread -o thread` 编译c++文件， thread不是linux标准库

`gdb: file filename -> start -> s` 

`layout src` 

`layout asm`

**GDB调试代码常用命令:**

| file<文件名> |                行为                 |
| :----------: | :---------------------------------: |
|     run      |            重新开始允许             |
|    start     |              单步执行               |
|     list     |          查看源码(简写：l)          |
|     set      |            设置变量的值             |
|     next     |          单步调试(简写：n)          |
|     step     |          单步调试(简写：s)          |
|    frame     |       切换函数的帧栈(简写：f)       |
|     info     | 查看函数内部局部变量的数值(简写：i) |
|    finish    |   结束当前函数，返回到函数调用点    |
|    print     |        打印值及地址(简写：p)        |
|   continue   |          继续运行(简写：c)          |
|    break     |          设置断点(简写：b)          |

调试一个正在运行的程序：

`gcc -g filename` :生成可执行文件

`./ a.out &` ：使可执行文件后台运行

`gdb -p pid` :通过正在运行的程序的pid，调试该程序

makefile用法：

```makefile
cc = gcc
main: main.c tool.o
	$(cc) main.c tool.o -o main
tool.o: tool.c
	gcc -c tool.c
clean:
	rm *.o main
```

多个函数生成一个含main的可执行文件:

```makefile
CC = gcc
main: main.c bar.o foo.o
	$(CC) main.c bar.o foo.o -o main
bar.o: bar.c 
	gcc -c bar.c 
foo.o: foo.c 
	gcc -c foo.c 
clean:
	rm *.o main
```

**常用shell命令**

`echo hello > hello.txt`  ：重定向到hello.txt

`cat< hello.txt  > hello2.txt` ：将hello.txt重定向到hello2.txt

`cat < hello.txt >> hello2.txt`  :将hello.txt中的内容复制两遍到hello2.txt

`tail -n5`

`ls -l / | tail -n4`

`ls -l / | tail -n4 > ls.md`

`curl --head --silent google.com | grep -i content-Type`

`tee` 用于从标准输入读入数据，并将其内容输出成文件

echo "val is $foo"

echo 'val is $foo'

cat <(ls) <(ls ..) > test.md

./example.sh mcd.sh scripts.py example.sh

`find  **/test/*.sh -type f`  ：找到未知父目录下的test文件夹下的以.sh结尾的文件

`find . -name "*.tmp" -exec rm {} \；`  :找到以.tmp后缀结尾的文件，然后执行删除操作

`groups xp` :查看xp用户所在的组

`traceroute www.google.com` : linux查看路由器表和跳转命令

`route -n` ：查看路由表

妙用sed的替换，对行操作

`sed 's/unix/linux/g' test.txt` ：用linux替换全部unix

`sed '3 s/unix/linux/g' test.txt` ：只替换第三个

`sed '1,3 s/unix/linux/' test.txt` ：替换一个范围

 awk指令,对列操作

`awk '{print $1, $2}' filename`
