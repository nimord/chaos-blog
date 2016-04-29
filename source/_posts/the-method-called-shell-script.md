---
title: shell脚本调用的三种不同方式
date: 2016-04-28 21:30:55
categories: 技术
tags: shell
---

bash的命令分为两类：外部命令和内部命令。外部命令是通过系统调用或独立的程序实现的，如sed、awk等；内部命令是由特殊的文件格式（.def）所实现，如cd、history、exec等。shell脚本调用有三种不同方法：`fork`, `exec`, `source`。其中fork是linux的系统调用，用来创建子进程。exec和source都属于bash内部命令（builtins commands）。

- **fork** (/directory/script.sh)：在子命令执行完后再执行父级命令，**子级的环境变量不会影响到父级**。

	+ fork是最常用的，就是直接在脚本里面用/directory/script.sh来调用script.sh这个脚本。运行的时候开一个sub-shell执行调用的脚本，sub-shell执行的时候， parent-shell还在。sub-shell执行完毕后返回parent-shell。 sub-shell从parent-shell继承环境变量，但是sub-shell中的环境变量不会带回parent-shell。

	+ 子进程是父进程(parent process)的一个副本，从父进程那里获得一定的资源分配以及继承父进程的环境。子进程与父进程唯一不同的地方在于pid（process id）。

- **exec**(exec /directory/script.sh)：** 执行子级的命令后，不再执行父级命令**。

	+ exec命令在执行时会把当前的shell process关闭，然后换到后面的命令继续执行。

	+ exec与fork不同，不需要新开一个sub-shell来执行被调用的脚本。被调用的脚本与父脚本在同一个shell内执行。但是使用exec调用一个新脚本以后，父脚本中exec行之后的内容就不会再执行了。这是exec和source的区别。

- **source**(source /directory/script.sh)：** 执行子级命令后继续执行父级命令，同时子级设置的环境变量会影响到父级的环境变量**。

	+ source命令即点(.)命令。

    + 与fork的区别是不新开一个sub-shell来执行被调用的脚本，而是在同一个shell中执行。所以被调用的脚本中声明的变量和环境变量, 都可以在主脚本中得到和使用。

可以通过下面这两个脚本来体会三种调用方式的不同:

1.sh
```bash
#!/bin/bash
A=B
echo "PID for 1.sh before exec/source/fork:$$"
export A
echo "1.sh: \$A is $A"
case $1 in
        exec)
                echo "using exec…"
                exec ./2.sh ;;
        source)
                echo "using source…"
                . ./2.sh ;;
        *)
                echo "using fork by default…"
                ./2.sh ;;
esac
echo "PID for 1.sh after exec/source/fork:$$"
echo "1.sh: \$A is $A"
```
2.sh
```bash
#!/bin/bash
echo "PID for 2.sh: $$"
echo "2.sh get \$A=$A from 1.sh"
A=C
export A
echo "2.sh: \$A is $A"
```
执行情况：
```
$ ./1.sh
PID for 1.sh before exec/source/fork:5845364
1.sh: $A is B
using fork by default…
PID for 2.sh: 5242940
2.sh get $A=B from 1.sh
2.sh: $A is C
PID for 1.sh after exec/source/fork:5845364
1.sh: $A is B

$ ./1.sh exec
PID for 1.sh before exec/source/fork:5562668
1.sh: $A is B
using exec…
PID for 2.sh: 5562668
2.sh get $A=B from 1.sh
2.sh: $A is C

$ ./1.sh source
PID for 1.sh before exec/source/fork:5156894
1.sh: $A is B
using source…
PID for 2.sh: 5156894
2.sh get $A=B from 1.sh
2.sh: $A is C
PID for 1.sh after exec/source/fork:5156894
1.sh: $A is C

```

**其他**
系统调用exec是以新的进程去代替原来的进程，但进程的PID保持不变。因此可以这样认为，exec系统调用并没有创建新的进程，只是替换了原来进程上下文的内容。原进程的代码段，数据段，堆栈段被新的进程所代替。

整理本文参考链接：
[http://mindream.wang.blog.163.com/blog/static/2325122220084624318692/](http://mindream.wang.blog.163.com/blog/static/2325122220084624318692/)

