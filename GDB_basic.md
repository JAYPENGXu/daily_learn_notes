## GDB_basic

#### 启动调试程序，带有参数形式

`run <arg1> <arg2>`... :简写为 r

#### 重复上一条指令

`按下回车键`

#### 单步步过

`nexti` :汇编层面， 简写ni

`next`:源码层面，简写n

#### 执行到返回

`finish` : 简写fini

#### 强制函数执行返回

`return` :简写ret

#### 查看所有设置的断点

`info break` : 简写info b

#### 删除断点

`del <break num>`

#### 使能/禁用断点

`enable breakpoints`

`distable breakpoints`

#### 查看某个寄存器的值

`print $rdi` : print简写p

#### 设置某个寄存器的值

`set $rdx = 0x01`

#### 设置内存上某个地址的值

`set {int} 0x7f0000 = 1`

#### 查看栈上的值，（传入参数为显示的栈数据个数）

`stack 20`

#### 在某个地址上设置断点（无参数时默认在 $rpi处设置）

b *0x40013

`b <line_number>` : 在当前源码文件中的第line_number行设置断点

`b <file_name>:<line_number>` : 在<file_name>源码文件中的第line_number行下断点

`tb *0x40013` ： 临时断点，只生效一次

`b *0x40013 if a == 1` ：条件断点

`ignore <breakponit_i> cnt`: 忽略特定断点编号为breakpoint_i的断点cnt次

#### 输出got表相关信息

`got`

#### 输出某个函数在libc上的地址

`print system`

#### 开启/关闭ASLR

`aslr off`

#### 显示当前进程空间内存分布

`vmmap`

#### 显示内存上的值

格式：

        # d 按十进制格式显示变量。
        # u 按十六进制格式显示无符号整型。
        # o 按八进制格式显示变量。
        # t 按二进制格式显示变量。
        # a 按十六进制格式显示变量。
        # c 按字符格式显示变量。
        # f 按浮点数格式显示变量。
大小：

      # b 一单元1字节
        # h 一单元2字节
        # w 一单元4字节
        # g 一单元8字节
`x/3uh 0x54320` :以地址0x54320为起始地址，返回3个单元的值，每个单元有两个字节，输出格式为无符号十六进制。

#### 切换线程

`thread <id>` ：切换至对应的id线程

`info thread` :查看当前进程中的所有线程信息

#### 退出GDB

`quit` 简写q

#### 选择父进程fork/exec之后的跟踪对象

`set follow-fork-mode parent/ child` : fork后跟踪父进程/子进程

`set foll-exec-mode new/same` ： 使用开启新inferior来跟踪exec程序

`set detach-on-fork on/off`：是否同时调试fork后的两个程序

调试线程时的设置

`set non-stop on` :设置其他没触发断点的线程继续执行

启动Ttext UI

`help tui`：查看tui的相关命令

`tui reg general`:启动tui时附带寄存器窗口

`tui enable` :启动tui

`tui disable` : 退出tui

#### # 启动layout UI

`help layout` ： 查看 layout 的相关命令
`layout src`   ： 显示源代码窗口
`layout asm`   ： 显示汇编窗口
`layout regs`  ：显示源代码/汇编和寄存器窗口
`layout split` ： 显示源代码和汇编窗口
`layout next`  ：显示下一个layout
`layout prev`  ： 显示上一个layout

#### 在GDB中直接调用源码中的某个函数
这个语句通常对于大型项目中，执行特定类型的Print()函数很有用

`call <expr>`

#### 环境变量相关
`show environment [key]` :显示环境变量
`set environment <key>=<val> ...` :添加环境变量
`unset environment [key]` :清空环境变量

#### signal 相关
`info signal / info handle` :查看 signal 设置

  #### operation如下：
- no stop         

  ####     当被调试的程序收到信号时，GDB不会停住程序的运行，但会打出消息告诉你收到这种信号。
- stop

  ####     当被调试的程序收到信号时，GDB会停住你的程序。
- print

  ####     当被调试的程序收到信号时，GDB会显示出一条信息。
- noprint

  ####     当被调试的程序收到信号时，GDB不会告诉你收到信号的信息。
- pass / noignore

  ####     当被调试的程序收到信号时，GDB不处理信号。这表示，GDB会把这个信号交给被调试程序会处理。
- nopass / ignore

  ####     当被调试的程序收到信号时，GDB不会让被调试程序来处理这个信号。
handle <SIG> <operation>

#### 扩展插件：

`Pwndbg、 Peda`

`pdb`（python debugger）:

用法： 使用 `python -m pdb filename.py`

- s(tep)：单步执行，相当于step into
- n(ext)：单步执行，相当于step over
- l(ist)：列出执行到的那行代码的前后十一行
- c(ontinue)：继续执行，直到遇到下一条断点
- b(reakpoint)：设置断点
- cl(ear)：清除断点
- r(eturn)：执行当前运行函数直到结束
- p(rint) <expression>：输出expression的值
- q(uit)：退出debugger
- restart：重新debug
- b <line> , b 6 在第6行设置断点
- p locals() : 查看所有变量当前的值

`hyperfine`：命令行基准测试工具

`hyperfine <command> ...`

`hyperfine --warmup 3 'fd -HI "[0-9]\.jpg" ' 'find -name "[0-9].jpg"'` :演示fd和find的基准测试

