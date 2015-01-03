---
layout: post
title: "Linux 中僵尸进程"
description: ""
category: linux 
tags: [linux, process]
---
{% include JB/setup %}


Linux 系统中僵尸进程和现实中僵尸（虽然我也没见过）类似，虽然已经死了，但是由于没人给它们收尸，还能四处走动。僵尸进程指的是那些虽然已经终止的进程，但仍然保留一些信息，等待其父进程为其收尸。配图源自 [Flickr](http://www.flickr.com/photos/dhollister/2596483147/) 
![2596483147_58d6bae3b1_z](https://f.cloud.github.com/assets/3265880/1352596/ab904876-3739-11e3-87dd-1faa6e8e42de.jpg)


## 僵尸进程如何产生的？ ##
如果一个进程在其终止的时候，自己就回收所有分配给它的资源，系统就不会产生所谓的僵尸进程了。那么我们说一个进程终止之后，还保留哪些信息？为什么终止之后还需要保留这些信息呢？

一个进程终止的方法很多，进程终止后有些信息对于父进程和内核还是很有用的，例如进程的ID号、进程的退出状态、进程运行的CPU时间等。因此进程在终止时，回收所有内核分配给它的内存、关闭它打开的所有文件等等，但是还会保留以上极少的信息，以供父进程使用。父进程可以使用 wait/waitpid 等系统调用来为子进程收拾，做一些收尾工作。

因此，一个僵尸进程产生的过程是：父进程调用fork创建子进程后，子进程运行直至其终止，它立即从内存中移除，但进程描述符仍然保留在内存中（进程描述符占有极少的内存空间）。子进程的状态变成 `EXIT_ZOMBIE`，并且向父进程发送 **SIGCHLD** 信号，父进程此时应该调用 `wait()` 系统调用来获取子进程的退出状态以及其它的信息。在 wait 调用之后，僵尸进程就完全从内存中移除。因此一个僵尸存在于其终止到父进程调用 wait 等函数这个时间的间隙，一般很快就消失，但如果编程不合理，父进程从不调用 wait 等系统调用来收集僵尸进程，那么这些进程会一直存在内存中。

在 Linux 下，我们可以使用 ps 等命令查看系统中僵尸进程，僵尸进程的状态标记为‘Z’：
![screenshot from 2013-10-17 22 36 17](https://f.cloud.github.com/assets/3265880/1352587/8c728030-3739-11e3-8424-d898bc393538.png)


## 产生一个僵尸进程 ##
根据上面的描述，我们很容易去写一个程序来产生僵尸进程，如下代码：

{% highlight c %}
#include <stdio.h>
#include <sys/types.h>

int main()
{
    //fork a child process
    pid_t pid = fork();

    if (pid > 0)   //parent process
    {
        printf("in parent process, sleep for one miniute...zZ...\n");
        sleep(60);
		printf("after sleeping, and exit!\n");
    }
    else if (pid == 0)  
    {
        //child process exit, and to be a zombie process
        printf("in child process, and exit!\n");
        exit(0);
    }

    return 0;
}
{% endhighlight %}

父进程并没有写 wait 等系统调用函数，因此在子进程退出之后变成僵尸进程，父进程并没有为其去收尸。我们使用下面命令编译运行该进程，然后查看系统中进程状态：

{% highlight bash %}
guohailin@guohailin:~/Documents$ gcc zombie.c -o zombie
guohailin@guohailin:~/Documents$ ./zombie 
in parent process, sleep for one miniute...zZ...
in child process, and exit!

# 打开另一个终端:
guohailin@guohailin:~$ ps aux | grep -w 'Z'
1000      2211  1.2  0.0      0     0 ?       Z    13:24   6:53 [chromium-browse] <defunct>
1000      4400  0.0  0.0      0     0 ?        Z    10月16   0:00 [fcitx] <defunct>
1000     10871  0.0  0.0      0     0 pts/4    Z+   22:32   0:00 [zombie] <defunct>
{% endhighlight %}

从上面可以看出，系统中多了一个僵尸进程。但如果等父进程睡眠醒来退出之后，我们再次查看系统进程信息，发现刚才的僵尸进程不见了。
{% highlight bash %}
guohailin@guohailin:~/Documents$ ./zombie 
in parent process, sleep for one miniute...zZ...
in child process, and exit!
after sleeping, and exit!
guohailin@guohailin:~/Documents$ ps aux | grep -w 'Z'
1000      2211  1.2  0.0      0     0 ?        Z    13:24   6:53 [chromium-browse] <defunct>
1000      4400  0.0  0.0      0     0 ?        Z    10月16   0:00 [fcitx] <defunct>
{% endhighlight %}

这是为什么呢？父进程到死都也没有为其子进程收尸呀，怎么父进程退出之后，那个僵尸进程就消失了呢？难道父进程在退出时会为子进程收拾吗？其实不然....真正的原因是：父进程死掉之后，其所有子进程过继给 init 进程，init 进程成为该僵尸进程的新进程，init 进程会周期性地去调用 wait 系统调用来清除它的僵尸孩子。因此，你会发现上面例子中父进程死掉之后，僵尸进程也跟着消失，其实是 init 进程为其收尸的！


## 怎样避免僵尸进程的产生 ##
不能使用 kill 后接 `SIGKILL` 信号这样的命令像杀死普通进程一样杀死僵尸进程，因为僵尸进程是已经死掉的进程，它不能再接收任何信号。事实上，如果系统中僵尸进程并不多的话，我们也无需去消除它们，少数的僵尸进程并不会对系统的性能有什么影响。

那么在编程时，如果能避免系统中大量产生僵尸进程呢？根据上面描述的，子进程在终止时会向父进程发 SIGCHLD 信号，Linux 默认是忽略该信号的，我们可以显示安装该信号，在信号处理函数中调用 wait 等函数来为其收尸，这样就能避免僵尸进程长期存在于系统中了。示例代码如下：

{% highlight c %}
#include <stdio.h>
#include <signal.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>

sig_atomic_t child_exit_status;

void clean_up_child_process(int signal_num)
{
    /* clean up child process */
    int status;
    wait (&status);

    /* store its exit status in a global variable */
    child_exit_status = status;
}

int main()
{
    /* handle SIGCHLD by calling clean_up_child_process  */
    struct sigaction sigchild_action;
    memset(&sigchild_action, 0, sizeof(sigchild_action));
    sigchild_action.sa_handler = &clean_up_child_process;
    sigaction(SIGCHLD, &sigchild_action, NULL);

    /* fork a child, and let the child process dies before parent */
    pid_t c_pid;
    c_pid = fork();
    if (c_pid > 0)
    {
        printf("in parent process, and sleep for on mininute...zZ...\n");
        sleep(60);
    }
    else if(c_pid == 0)
    {
        printf("in child process, and exit now\n");
        exit(0);
    }
    else
    {
        printf("fork failed!\n");
    }

    return 0;
}	
{% endhighlight %}


## 参考资料 ##

* [http://en.wikipedia.org/wiki/Zombie_process](http://en.wikipedia.org/wiki/Zombie_process)
* [http://simplestcodings.blogspot.com/2010/10/story-of-zombie-process.html](http://simplestcodings.blogspot.com/2010/10/story-of-zombie-process.html)
* [http://www.howtogeek.com/119815/](http://www.howtogeek.com/119815/)
