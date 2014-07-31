---
layout:    post
title:     僵尸进程
category:  基础
description: 僵尸进程...
tags: 僵尸进程 进程
---
当父进程有子进程的时候，父进程需要知道子进程是否结束。wait4()系统调用允许进程等待，直到其中的一个子进程结束，wait4()返回已中止进程的进程标识符（Process ID，PID）。

内核在执行这个系统调用时，检查子进程是否已经终止。引入僵尸进程的特殊状态是为了表示终止进程：父进程执行完wait4()调用之前，进程就一直停留在那种状态。系统调用处理程序从进程描述符字段中获取有关资源使用的一些数据；一旦得到了数据，就可以释放进程描述符。当进程执行wait4()调用时如果没有子进程结束，内核就把父进程设置为等待状态，一直到子进程结束[^1]。

[^1]: 很多内核也实现了waitpid()系统调用，它允许进程等待一个特殊的子进程。

在父进程发出wait4()调用之前，让内核保存子进程的有关信息是很重要的，但是，假设父进程终止而没有发出wait4()调用，这些信息占用了系统中的资源，而这些资源本来是可以给其他活着的进程使用，于是这些资源无法被释放，就成为了僵尸进程。其产生是由于，在Unix操作系统下，程序必须由一个进程或另一个用户杀死[^2]，进程的父进程在子进程终止时必须调用或者已调用wait4()系统调用，这就是向内核证实父进程已经确认了子进程的结束。

只有在程序终止并且wait4()调用不成功的情况下，才会出现僵尸状态。在进程终止后，其数据没有从进程表中删除之前，进程总是处于僵尸状态。有时候[^3]，进程可能稳定的在进程表中无法删除，直至下一次系统重启。

[^2]: 通常是通过发送SIGTERM或者SIGKILL信号来完成，这等价于正常的进程终止。

[^3]: 例如糟糕的父进程编程。

有一个解决办法是使用一个名为*init*的特殊系统进程，它在系统初始化的时候创建，当一个进程终止时，内核改变所有现有子进程的进程描述符指针，使这些子进程成为*init*的孩子。*init*监控所有子进程的执行，并按照常规发布wait4()系统调用，其作用就是清除掉所有的僵尸进程。

僵尸进程可以使用ps或者top工具查看，因为残余的数据在内核中占用较少，所以几乎不是问题。