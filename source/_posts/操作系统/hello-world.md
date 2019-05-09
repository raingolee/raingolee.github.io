---
title: linux进程,fork,exec,wait
layout: post
tags:
 - apue 
categories:
  - 操作系统
  - APUE
---

此文章
要了解多进程，先要知道Linux的基础，如进程的概念，fork系统调用，exce库函数

<!-- more -->
#### 什么为进程
什么是进程？一个进程只是一个正在执行的程序的实例。 比如说，当服务器代码被执行的时候，它被加载到内存里，然后这个正在执行的程序的实例就叫做进程。 内核记录了有关这个进程的一大串的信息—— 比如进程的 ID ——为了方便跟踪这个进程。


当程序文件运行为进程时，进程在内存中获得空间。这个空间是进程自己的小屋子。
![内存使用情况](http://upload-images.jianshu.io/upload_images/699442-002f1c1aca80d260.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

栗子：
```
#include <stdio.h>

int glob=0;                                             /*global variable*/

void main(void) {
  int main1=5;                                          /*local variable of main()*/
  int main2;                                            /*local variable of main()*/
  main2 = inner(main1);                                 /* call inner() function */
  printf("From Main: glob: %d \n", glob);
  printf("From Main: main2: %d \n", main2);
}

int inner(int inner1) {                                 /*inner1 is an argument, also local to inner()*/
  int inner2=10;                                        /*local variable of inner()*/
  printf("From inner: glob: %d \n", glob);
  return(inner1+inner2);
}
```

>Text区域用来储存指令(instruction)，说明每一步的操作。Global Data用于存放全局变量，栈(Stack)用于存放局部变量，堆(heap)用于存放动态变量 (dynamic variable. 程序利用malloc系统调用，直接从内存中为dynamic variable开辟空间)。Text和Global data在进程一开始的时候就确定了，并在整个进程中保持固定大小。
> 
栈(Stack)以帧(stack frame)为单位。当程序调用函数的时候，比如main()函数中调用inner()函数，stack会向下增长一帧。帧中存储该函数的参数和局部变量，以及该函数的返回地址(return address)。
>
此时，计算机将控制权从main()转移到inner()，inner()函数处于激活(active)状态。位于栈最下方的帧，和全局变量一起，构成了当前的环境(context)。激活函数可以从环境中调用需要的变量。典型的编程语言都只允许你使用位于stack最下方的帧 ，而不允许你调用其它的帧 (这也符合stack结构“先进后出”的特征。但也有一些语言允许你调用栈的其它部分，相当于允许你在运行inner()函数的时候调用main()中声明的局部变量，比如Pascal)。当函数又进一步调用另一个函数的时候，一个新的帧会继续增加到栈的下方，控制权转移到新的函数中。当激活函数返回的时候，会从栈中弹出(pop，读取并从栈中删除)该帧，并根据帧中记录的返回地址，将控制权交给返回地址所指向的指令(比如从inner()函数中返回，继续执行main()中赋值给main2的操作)。


#### linux init
实际上，当计算机开机的时候，内核(kernel)只建立了一个init进程。Linux内核并不提供直接建立新进程的系统调用，剩下的所有进程都是init进程通过fork机制建立的。

新的进程要通过老的进程复制自身得到，这就是fork。fork是一个系统调用。进程存活于内存中。每个进程都在内存中分配有属于自己的一片空间 (address space)。当进程fork的时候，Linux在内存中开辟出一片新的内存空间给新的进程，并将老的进程空间中的内容复制到新的空间中，此后两个进程同时运行。

老进程成为新进程的父进程(parent process)，而相应的，新进程就是老的进程的子进程(child process)。一个进程除了有一个PID之外，还会有一个PPID(parent PID)来存储的父进程PID。如果我们循着PPID不断向上追溯的话，总会发现其源头是init进程。所以说，所有的进程也构成一个以init为根的树状结构。

可以通过pstree查看一下系统进程的情况

![pstree](http://upload-images.jianshu.io/upload_images/699442-dbf7197ad81610e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### fork && exce

![fork](http://upload-images.jianshu.io/upload_images/699442-be0338be9a20f86b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

fork通常作为一个函数被调用。这个函数会有两次返回，将子进程的PID返回给父进程，0返回给子进程。实际上，子进程总可以查询自己的PPID来知道自己的父进程是谁，这样，一对父进程和子进程就可以随时查询对方。

通常在调用fork函数之后，程序会设计一个if选择结构。当PID等于0时，说明该进程为子进程，那么让它执行某些指令,比如说使用exec库函数(library function)读取另一个程序文件，并在当前的进程空间执行 (这实际上是我们使用fork的一大目的: 为某一程序创建进程)；而当PID为一个正整数时，说明为父进程，则执行另外一些指令。由此，就可以在子进程建立之后，让它执行与父进程不同的功能。
```
#include <unistd.h>
#include <sys/types.h>
#include <errno.h>
#include <stdio.h>
#include <sys/wait.h>
#include <stdlib.h>

int var_glb; /* A global variable*/

int main(void)
{
    pid_t childPID;
    int var_lcl = 0;

    childPID = fork();

    if(childPID >= 0) // fork was successful
    {
        if(childPID == 0) // child process
        {
            var_lcl++;
            var_glb++;
            printf("\n Child Process :: var_lcl = [%d], var_glb[%d]\n", var_lcl, var_glb);
        }
        else //Parent process
        {
            var_lcl = 10;
            var_glb = 20;
            printf("\n Parent process :: var_lcl = [%d], var_glb[%d]\n", var_lcl, var_glb);
        }
    }
    else // fork failed
    {
        printf("\n Fork failed, quitting!!!!!!\n");
        return 1;
    }

    return 0;
}
```
> 
现在，我们可以更加深入地了解fork和exec了。当一个程序调用fork的时候，实际上就是将上面的内存空间，包括text, global data, heap和stack，又复制出来一个，构成一个新的进程，并在内核中为改进程创建新的附加信息 (比如新的PID，而PPID为原进程的PID)。
>
此后，两个进程分别地继续运行下去。新的进程和原有进程有相同的运行状态(相同的变量值，相同的instructions...)。我们只能通过进程的附加信息来区分两者。
程序调用exec的时候，进程清空自身内存空间的text, global data, heap和stack，并根据新的程序文件重建text, global data, heap和stack (此时heap和stack大小都为0)，并开始运行。
>
(现代操作系统为了更有效率，改进了管理fork和exec的具体机制，但从逻辑上来说并没有差别。具体机制请参看Linux内核相关书籍)

还有注意的是：当一个父进程 fork 一个新的子进程时，子进程获得了一份父进程的文件描述符的拷贝


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/699442-404d66d2cec11a00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 子进程的终结(termination)
当子进程终结时，它会通知父进程，并清空自己所占据的内存，并在内核里留下自己的退出信息(exit code，如果顺利运行，为0；如果有错误或异常状况，为>0的整数)。在这个信息里，会解释该进程为什么退出。父进程在得知子进程终结时，有责任对该子进程使用wait系统调用。

这个wait函数能从内核中取出子进程的退出信息，并清空该信息在内核中所占据的空间。但是，如果父进程早于子进程终结，子进程就会成为一个孤儿(orphand)进程。孤儿进程会被过继给init进程，init进程也就成了该进程的父进程。init进程负责该子进程终结时调用wait函数。

当然，一个糟糕的程序也完全可能造成子进程的退出信息滞留在内核中的状况（父进程不对子进程调用wait函数），这样的情况下，子进程成为僵尸（zombie）进程。当大量僵尸进程积累时，内存空间会被挤占。

**孤儿进程：一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作。**

**僵尸进程：一个进程使用fork创建子进程，如果子进程退出，而父进程并没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中。这种进程称之为僵死进程。**

任何一个子进程(init除外)在exit()之后，并非马上就消失掉，而是留下一个称为僵尸进程(Zombie)的数据结构，等待父进程处理，如果存在大量的僵尸进程，内核会终耗尽所有可用的进程（max user processes）

那要如何避免这些僵尸进程呢，可以有以下的方法
- signal && wait
大家应该知道进程间通信有几种方法吧，我们可以利用信号的机制，例如子进程在完成的时候触发 SIGCHLD通知父进程，父进程对子进程进行wait or waitpid 函数，这个wait函数能从内核中取出子进程的退出信息，并清空该信息在内核中所占据的空间。

- fork两次
A -- fork -- A 父进程，B子进程
B -- fork -- B 父进程，C进程
这时候B进程马上退出，A进程帮B进程收尸(wait)清空在内核的信息，这时候C进程的父进程就变成了init，这样init就负责对c进程收尸而不会造成僵尸进程了
