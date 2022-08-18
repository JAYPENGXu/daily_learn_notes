### Linux基础操作

##### 管道：类似文件重定向，可以将前一个命令的stdout重定向到下一个命令的stdin， 仅能处理stdout，会忽略stderr。

##### 文件重定向：左边为命令，右边为文件。

e g. : 统计当前目录下所有python文件的总行数

```shell
find . -name '*.py' | xargs cat | wc -l

xargs #将stdin中的数据用空格或回车分割成命令行参数

wc -l #统计行数
wc -w #统计单词数
wc -c #统计字节数
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

环境变量的修改：为了将对环境变量的修改持久化，可以将修改命令放到  ~/.bashrc文件中，之后执行 source ~/.bashrc 

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

##### cut 分割一行内容，从标准输入中读，echo $PATH |cut -d ':' -f 1，输出PATH用：分割后第1个数据

##### echo $PATH | cut -c 3,5：输出PATH的第3、5个字符

##### more: more main.cpp,回车下一行，空格下一页，b上一页

##### less:与more类似，回车下一行，y上一行，pagedown下一页，pageup上一页

##### head -3 xxx

##### tail -3 xxx

##### history查看当前用户的所有操作记录

##### md5sum 计算md5哈希值 md5sum main.cpp

##### time command 统计command 命令的执行时间

time free

##### watch -n 0.1 command :每0.1秒执行一次command命令

watch -n 0.1 free

##### 压缩文件: tar -zcvf xxx.tar.gz /path/to/file/*

tar -zcvf rrr.tar.gz /tmp/*

##### 解压文件： tar-zxvf xxx.tar.gz 

tar -zxvf rrr.tar.gz

##### diff xxx yyy 查找xxx 和 yyy文件的不同点



##### sudo command ：以root身份执行command命令

##### apt-get install xxx：安装软件

##### pip install xxx --user --upgrade:安装python包



