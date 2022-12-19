《xv6: a simple, Unix-like teaching operating system》

一、 操作系统接口（Operating system interfaces）

操作系统的工作是在多个程序之间共享一台计算机，并提供一个比硬件本身支持的更有用的服务集，操作系统在多个程序之间共享硬件资源，以便它们（看似）同时运行，操作系统为程序交互提供受控方式，以便它们可以共享数据或者协同工作。

<img src="G:\typora_image_store\image-20221216124255467.png" alt="image-20221216124255467" style="zoom:80%;" />

每个运行的程序称之为进程，由包含指令的内存，数据和栈组成。指令实行程序的计算，数据交由计算执行，栈管理程序的调度。当一个进程需要唤醒一个kernel服务，它就唤醒了一个system call，每个system call 进入内核，内核完成服务并返回。kernel使用了硬件保护机制提供给CPU，确保每个进程在自己的用户空间只能访问自己的内存。用户程序执行时没有这种特权，当一个用户程序唤醒一个系统调用时，硬件会提升权限并开始执行预先在内核中排序好的程序。内核提供的系统调用集合是用户程序看到的接口。

shell是一个原生的程序用来读取指令并执行。实际上，shell是一个用户程序，而不是内核的一部分。

1.1 processes and memory

xv6包含用户空间内存（指令、数据、栈），和内核私有的每个进程状态。当进程不再执行时xv6将存储和这些进程相关的CPU寄存器直到下一次运行这些进程，内核通过PID连接每个进程。

`int fork()`：一个进程可以通过fork创建一个新的进程，提供相同的内存上下文包括指令和数据，父进程返回子进程的PID，子进程返回0。

```c
int pid = fork();  //fork之后，父子进程同时开始判断pid的值，看哪个进程先判断好pid才会决定输出顺序。
printf("pid=%d\n", pid);
if(pid > 0){
	printf("parent: child=%d\n", pid);
	pid = wait((int *) 0); //父进程发现子进程exit之后，wait执行完毕，打印输出。
	printf("child %d is done\n", pid);
} else if(pid == 0){
	printf("child: exiting\n");
	exit(0);  //子进程在判断pid==0之后将exit,
} else {
	printf("fork error\n");
}

---
输出：
pid=2018356
parent:child=2018356
pid=0
child:existing
child 2018356 is done  //当子进程退出后，父进程的wait就会返回。

```

`int exit(int status)`: 系统调用会停止执行并释放资源,如内存和打开的文件,需要一个整型的状态参数，返回值0表示正常状态退出，1表示非正常状态退出。

`int wait(int *status)`：系统调用返回当前进程被停止或被杀掉的子进程的状态，返回子进程的PID， 子进程的退出状态存储到 int *status这个地址中，并将状态码拷贝到内存中等待。如果调用者没有子进程，wait立即返回-1，如果父进程不在乎子进程的退出态，它可以给一个0地址给wait。父进程和子进程有着相同的内存上下文，但是在不同的内存和寄存器中执行。改变一个并不会改变另一个。如当wait的返回值被父进程存储到pid中时，它不会改变子进程中变量pid中的值，子进程中的变量pid的值还是0。

`int exec(char *file, char *argv[])`:加载一个文件，获取执行它的参数，执行，如果执行错误返回-1，执行成功不返回，而是开始从文件入口位置开始执行命令。文件必须有一个特殊的格式，必须指明那一部分是保存指令，那一部分是数据。xv6使用的是ELF文件格式，当exec成功时，他不会返回给调用程序，相反，从文件加载的指令在 ELF 标头中声明的入口点开始执行。exec输入两个参数：文件包含的可执行文件名和一组参数。

```c
//xv6 shell使用以上四个system call来为用户执行程序。在shell进程的main中主循环先通过getcmd来从用户获取命令，然后调用fork来运行一个和当前shell进程完全相同的子进程。父进程调用wait等待子进程exec执行完（在runcmd中调用exec）

char *argv[3];
argv[0] = "echo";
argv[1] = "hello";
argv[2] = 0;
exec("/bin/echo", argv);
printf("exec error\n");
//---------------------------------------------------
int main(){
    static char buf[100];
    int fd;
    // ensure that three file descriptions are open.
    while(fd = open("console", O_RDWR ) >= 0){
        if(fd >= 3){
            close(fd);
            break;
        }
    }
    // read and run input commands
    while(getcmd(buf, sizesof(buf)) >= 0){
        if(buf[0] == 'c' && buf[1] == 'd' && buf[2] == ' '){
            // chdir must be called by the parent, not the child.
            buf[strlen(buf) - 1] = 0;
            if(chdir(buf + 3) < 0)
                fprintf(2, "cannot cd %s\n", buf + 3);
            continue;
        }
        if(fork1() == 0)
            runcmd(parsecmd(buf));
        wait(0);
    }
    exit(0);
}
```

避免创建重复进程然后立即替换它的浪费,操作内核通过使用虚拟内存技术（如写时复制）优化此用例的 fork 实现。xv6 隐式分配大部分用户空间内存：fork 分配父内存的子副本所需的内存，exec 分配足够的内存来保存可执行文件。在运行时需要更多内存的进程（可能是 `malloc`）可以调用 `sbrk(n)` 将其数据内存增加 n 字节； `sbrk` 返回新内存的位置。

1.2 输入输出和文件描述符 (I/O and File descriptors)

文件描述符时用来表征一个进程将从那里读取或者写入到哪里的内核管理对象。一个进程可以通过打开一个文件、目录或设备或创建一个管道，或复制一个已经存在的描述符。文件描述符接口抽象了文件、管道和设备之间的差异，使它们看起来都像字节流。xv6 内核使用文件描述符作为每个进程表的索引，因此每个进程都有一个从零开始的文件描述符的私有空间。每个进程都拥有自己独立的文件描述符列表，其中0是标准输入，1是标准输出，2是标准错误。shell将保证总是有3个文件描述符是可用的。

`int read(int fd,char * buf,int n)`; 从文件描述符fd中读取n字节的数据拷贝到buf中，**返回值是读取的字节数**。每个文件描述符都有一个与之相关的offset。read 从当前文件偏移量读取数据，然后将该偏移量增加读取的字节数，下一个read将从新的offset开始读取字节，当没有更多的字节可以读取时，read返回0来表示读取到文件的结束。

`int write(int fd, char *buf,int n)`；从buf中写入n字节的数据到文件描述符fd中，返回值是有多少的字节数据被写入。当少于n个字节的数据被写入时互出现错误，通read一样存在偏移量。

```c
//cat命令的实现
char buf[512];
int n;
for(;;){
    n = read(0, buf, sizeof buf);
    if(n == 0) break;
    if(n < 0){
        fprintf(2, "read error\n");
        exit(1);
    }
    if(write(1, buf, n) != n){
        fprintf(2, "write error\n");
        exit(1);
    }
}
```

`int close(int fd)`:系统调用，释放一个文件描述符，让它变得可以被未来的open pipe 或者dup系统调用使用。

fork拷贝父进程的文件描述符到自己的内存空间中，因此子进程可以像父进程一样打开文件，exec代替了调用进程的内存，但是将它保留在文件表中。父进程的fd table将不会被子进程fd table的变化影响，但是文件中的offset将被共享。

```c
// cat 命令的实现 cat < input.txt
char *argv[2];
argv[0] = "cat";
argv[1] = 0;
if(fork() == 0){
    close(0);
    open("input.txt", O_RDONLY);
    exec("cat", argv);
}
```

关键字： O_RDONLY：打开文件for reading, OWRONLY:打开文件for writting, , ORDWR:打开文件for reading and writting , O_CREATE:如果文件不存在，就创造它, O_TRUNC：将文件截断到0的长度。

`int dup(int fd)`:系统调用，返回一个引用相同底层 I/O 对象的新对象。两个文件描述符共享一个偏移量，就像 fork 复制的文件描述符一样。

```c
// another way to write hello world
fd = dup(1);
write(1, "hello ", 6);
write(fd, "world\n", 6);
```

如果两个文件描述符是通过一系列 fork 和 dup 调用从同一个原始文件描述符派生的，则它们共享一个偏移量,否则文件描述符不共享偏移量，即使它们来自对同一文件的打开调用.Dup 允许shell去实现一个这样的命令： ls existing-file non-existing-file > tmp1 2 >&1, 2>&1 告诉 shell 给命令一个文件描述符 2，它是描述符 1 的副本。现有文件的名称和不存在文件的错误消息都将显示在文件 tmp1 中.xv6 shell 不支持错误文件描述符的 I/O 重定向，但现在你知道如何实现它了。

1.3管道

管道是一个小的内核缓冲区，作为一对文件描述符暴露给进程，一个用于读取，一个用于写入。将数据写入管道的一端使该数据可用于从管道的另一端读取。 管道为进程提供了一种通信方式。（FIFO实现）

`int pipe(int p[])`:p[0]为读取的fd,p[1]为写入的fd。

```c
//以下示例代码运行程序 wc，标准输入连接到管道的读取端。
int p[2];
char *argv[2];
argv[0] = "wc";
argv[1] = 0;
pipe(p);
if(fork() == 0){
    close(0);
    dup(p[0]);
    close(p[0]);
    close(p[1]);
    exec("/bin/wc", argv);
}else {
    close(p[0]);
    write(p[1], "hello world\n", 12);
    close(p[1]);
}
//这段程序叫做pipe，它创造了一个新的管道来记录读和写的文件描述符在数组p中。，在fork之后，父进程和子进程都会有文件描述符指向这个管道，子进程调用close和dup来让文件描述符0引用管道的读取端，关闭文件描述符，之后调用exec执行wc指令，当wc读取它的标准输入时，它从管道读取，父进程关闭了管道的读取端，写管道，然后关闭写端。
```

如果没有数据可用，则管道上的读取等待写入数据或关闭引用写入端的所有文件描述符； 在后一种情况下，read 将返回 0，就像已到达数据文件的末尾一样。 读取阻塞直到新数据不可能到达的事实是子进程在执行上面的 wc 之前关闭管道的写入端很重要的一个原因：如果 wc 的文件描述符之一引用了写入端管道，wc永远看不到文件结束。

xv6的shell实现一个这样的命令： grep fork sh.c | wc -l，子进程创建一个管道来连接管道的左端和右端。之后在管道左端调用fork和runcmd指令，在管道右端调用fork和runcmd指令在管道的右端，然后等待两边都结束。管道的右端可能是一个命令，它本身包含一个管道（例如，a | b | c），它本身会派生两个新的子进程（一个用于 b，一个用于 c）。因此，shell 可以创建一个进程树。这棵树的叶子节点是命令，内部节点是等待左右子节点完成的进程。原则上，可以让内部节点运行在管道的左端，但如果这样做正确的话会使实施变得复杂。考虑仅进行以下修改：
将 sh.c 更改为不分叉 p->left 并在内部进程中运行 runcmd(p->left)。之后例如：例如，echo hi | wc不会产生输出，因为在runcmd中echo hi退出时，内部进程退出，从不调用fork运行右端管道,这种不正确的行为可以通过不在 runcmd 中为内部进程调用 exit 来修复，但此修复使代码复杂化：现在 runcmd 需要知道它是否在内部进程中。当不分叉 runcmd(p->right) 时也会出现并发症。例如，仅进行该修改，sleep 10 | echo hi 将立即打印“hi”和一个新提示，而不是在 10 秒后； 发生这种情况是因为 echo 立即运行并退出，而不是等待 sleep 完成。由于 sh.c 的目标是尽可能简单，它不会试图避免创建内部进程。

在这种情况下，管道比临时文件至少有四个优势。第一：管道会自动清理自己，使用文件重定向，shell必须小心删除/tmp/xyz当它执行完成后。其次，管道可以传递任意长的数据流，而文件重定向需要磁盘上有足够的可用空间来存储所有数据。 第三、管道允许管道并行执行阶段，而文件方法要求第一个程序在第二个程序开始之前完成。 第四，如果你正在实现进程间通信，管道的阻塞读写比文件的非阻塞语义更高效。

1.4文件系统

xv6 文件系统提供数据文件，其中包含未解释的字节数组和目录，其中包含对数据文件和其他目录的命名引用。这些目录形成一棵树，从称为root的特殊目录开始。不以 / 开头的路径是相对于调用进程的当前目录进行评估的，该目录可以通过 chdir 系统调用进行更改。这两个代码片段都打开同一个文件（假设所有涉及的目录都存在).

```c
chdir("/a");
chdir("b");
open("c", O_RDONLY);
open("/a/b/c", O_RDONLY);
//第一个片段将进程的当前目录更改为/a/b； 第二个既不引用也不更改进程的当前目录。
```

有创建新文件和目录的系统调用：mkdir 创建新目录，使用 O_CREATE 标志打开创建新数据文件，mknod 创建新设备文件。 这个例子说明了所有三个：

```c
mkdir("/dir");
fd = open("/dir/file", O_CREATE|O_WRONLY);
close(fd);
mknod("/console", 1, 1);
```

`mknod` 创建一个引用设备的特殊文件.与设备文件相关联的是主设备号（major device）和次设备号(minor device),mknod 的两个参数，它们唯一地标识内核设备。当进程稍后打开设备文件时，内核将read和write系统调用转移到这个内核设备上，而不是将它们传递给文件系统。

文件名与文件本身不同； 同一个底层文件，称为索引节点（inode），可以有多个名称，称为links。每个链接都包含目录中的一个条目； 该条目包含一个文件名称和对 inode 的引用。inode 保存有关文件的元数据，包括其类型（文件或目录或设备）、长度、文件内容在磁盘上的位置以及文件的链接数。

`int fstat(int fd, struct stat *st)` ：系统调用从文件描述符**fd**引用的 inode 检索信息，将inode中的相关信息存储到**st**中。执行成功返回0， 失败返回-1。在 stat.h (kernel/stat.h) 中定义为：

```c
#define  T_DIR 1
#define T_FILE 2
#define T_DEVICE 3
struct stat{
    int dev; //file system's disk device
    uint ino; //inode number
    short type; //type of file
    short nlink; // number of links to file
    uint64 size; //size of file in bytes
};
```

`link`:系统调用，创建一个指向同一个inode的文件名。`unlink`则是将一个文件名从文件系统中移除，只有当指向这个inode的文件名的数量为0时，这个inode以及其存储的文件内容才会被从硬盘上移除。 此片段创建一个名为 a 和 b 的新文件。

```c
open("a", O_CREATE|O_WRONLY);
link("a", "b");
```

从 a 读取或写入 a 与从 b 读取或写入相同。每个 inode 都由唯一的 inode 编号标识。在上面的代码序列之后，可以通过检查 fstat 的结果来确定 a 和 b 引用相同的底层内容：两者都将返回相同的 inode 号（ino），并且 nlink 计数将设置为 2.unlink 系统调用从文件系统中删除一个名称。 只有当文件的链接计数为零并且没有文件描述符引用它时，文件的索引节点和保存其内容的磁盘空间才会被释放。 从而添加 unlink("a")；到最后一个代码序列使 inode 和文件内容可访问为 b。

```c
fd = open("/tmp/xyz", O_CREATE|O_RDWR);
unlink("/tmp/xyz");
```

是创建一个没有名称的临时 inode 的惯用方法，当进程关闭 fd 或退出时将被清理.

Unix 提供可从 shell 调用的文件实用程序作为用户级程序，例如 `mkdir`、`ln` 和 `rm`等。这种设计允许任何人通过添加新的用户级程序来扩展命令行界面。事后看来，这个计划似乎很明显，但在 Unix 时代设计的其他系统通常将此类命令内置到 shell 中（并将 shell 内置到内核中）。一个例外是 cd，它内置于 shell (user/sh.c:160) 中。 cd 必须更改 shell 本身的当前工作目录。 如果 cd 作为常规命令运行，则 shell 将派生一个子进程，子进程将运行 cd，而 cd 将更改子进程的工作目录。 父级（即 shell 的）工作目录不会改变。

1.5 Real world

Unix 将“标准”文件描述符、管道和用于操作它们的方便的 shell 语法相结合，这是编写通用可重用程序的重大进步。 这个想法引发了一场 “软件工具”文化,也是 Unix 强大和流行的主要原因，而 shell 是第一个所谓的“脚本语言”。 Unix 系统调用接口今天在 BSD、Linux 和 macOS 等系统中仍然存在。Unix 系统调用接口已通过可移植操作系统接口 (POSIX) 标准进行了标准化。xv6 不符合 POSIX：它缺少许多系统调用（包括基本的系统调用，例如 lseek），并且它提供的许多系统调用与标准不同。 xv6 的主要目标是简单明了，同时提供简单的类 UNIX 系统调用接口。 为了运行基本的 Unix 程序，一些人用更多的系统调用和一个简单的 C 库扩展了 xv6。然而，与 xv6 相比，现代内核提供了更多的系统调用和更多种类的内核服务。 例如，它们支持网络、窗口系统、用户级线程、许多设备的驱动程序等等。 现代内核不断快速发展，并提供许多超越 POSIX 的功能。

Unix 使用一组文件名和文件描述符接口统一访问多种类型的资源（文件、目录和设备）。 这个想法可以扩展到更多种类的资源；一个很好的例子是计划 9 [14]，它将“资源是文件”的概念应用于网络、图形等。然而，大多数 Unix 派生的操作系统并没有遵循这条路线。

文件系统和文件描述符是强大的抽象。 即便如此，还有其他操作系统接口模型。 Multics 是 Unix 的前身，它以一种使文件存储看起来像内存的方式抽象了文件存储，从而产生了一种截然不同的界面风格。 Multics 设计的复杂性直接影响了 Unix 的设计者，他们试图构建更简单的东西。

Xv6 不提供用户或保护一个用户免受另一个用户侵害的概念； 在 Unix 术语中，所有 xv6 进程都以 root 身份运行。

1.6 Exercises

“乒乓”：创建两个进程在管道的两端，父进程发送字节到子进程，子进程接收到字节打印自己的pid和recevied ping，然后通过管道写字节到父进程，父进程接收到后打印pid和recevied pong然后退出。

```c
#include"kernel/types.h"
#includeuser/user .h"
const int MESSAGE_SIZE = 7;
int main(int argc, char *argv[]){
    int fd[2];
    char buf[MESSAGE_SIZE];
    pipe(fd);
    int pid = fork();
    if(pid > 0){
        write(fd[1], "114514", 6);
        read(fd[0], buf, 6);
        printf("%d: received pong\n", getpid());
        exit(0);
    }else{
        read(fd[0], buf, 6);
        printf("%d: received ping\n", getpid());
        write(fd[1], "114514", 6);
        exit(0);
    }
}
```

素数筛法：使用pipe和fork实现流水线，将一组数范围为2到35喂入到一个进程中，先打印出最小的一个数，这个数是素数，之后用其他的数来除这个素数，如果可以整除则将其drop，不能整除的喂入到下一个进程，直到打印出所有的素数。





实现find：使用递归的方式找到指定的文件夹下的目标文件，参考user/ls.c实现方法：

```c
// hints:1.
struct dirent {
    ushort inum;
    char name[DIRSIZE];
}
//hints:2.
void *memmove(void *str1, const void *str2, size_t n);//从str2复制n个字符到str1，返回一个指向目标存储区str1的指针。
```





实现xargs：

`xargs`的作用是把stdin中的数据用空格或回车分割成命令行参数

例：`find .-name '*.py' | xargs cat | wc -l` :统计当前目录下所有python文件的总行数

