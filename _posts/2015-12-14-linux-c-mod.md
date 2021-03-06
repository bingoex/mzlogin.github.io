---
layout: post
title: Linux下开发－权限详解
categories:  C/C++
description: 
keywords: 
---

权限的一些基本概念可以查看《鸟哥的私房菜》，本文只讲解跟API相关的权限问题。

# 基本概念

**UID和GID这样的标识符会影响文件的所有权和访问许可，以及向其它进程发送信号的能力**。这些属性统称为凭证。 

内核会给每个**进程**关联两个和进程ID无关的用户ID，一个是**真实用户ID**，还有一个是**有效用户ID**（setuid、setuserID）。**真实用户ID用于标识由谁为正在运行的进程负责**，还用于通过kill系统调用向其它**进程发送信号时的许可检查**。**有效用户ID**用于为**新创建的文件分配所有权、检查文件访问许可**。

有效UID和有效GID影响文件的创建和访问。**在创建文件的时候，内核将文件的所有者属性设置成创建进程的有效UID和有效GID**。在访问文件的时候，内核使用进程的有效UID和GID来判断它是否能够访问该文件。**真实UID和真实GID标识进程的真实所有者，会影响到发送信号的权限**。对于一个没有超级用户权限的进程来说，仅当它的真实或有效UID等于另一个进程的真实UID匹配时它才能向那个进程发送信号。 

当一个用户登录的时候，login程序会把两对ID设置成密码数据库（/etc/passwd文件，或某些如SunMicrosystems的NIS之类的分布式机制）中指定的UID和GID。**当一个进程fork的时候，子进程将从父进程那儿继承它的凭证**。 

有三个系统调用可以改变凭证。如果一个进程调用exec执行一个suid模式的程序，内核将把进程的有效UID修改成文件的所有者。同样，如果该程序安装为sgid模式，内核则会去修改调用进程的有效GID。

**一个用户还可以通过调用setuid或setgid来改变它的凭证。超级用户可以通过这些系统调用改变真实的和有效的UID以及GID。普通用户则只能通过这些调用来把它们的有效UID或GID改回到真实的数值**



# chmod和fchmod
```c
<sys/types.h>
<sys/stat.h>
int chmod(const char *path, mode_t mode);
int fchmod(int fildes, mode_t mode);
```

为了改变一个文件的权限位，进程有效用户ID，必须等于文件的所有者ID，或者该进程必须具有超级用户权限。

chmod要求给出的是文件或目录所在的位置，而**fchmod主要针对的是文件的文件描述符**。（chmod函数在指定的文件进行操作，而fchmod函数则对已经打开的文件进行操作）

Chmod函数的mode常量

mode | 说明
S_ISUID | 执行时设置用户ID
S_ISGID | 执行时设置组ID
S_ISVIX | 保存正文(粘住位)
S_IRWXU | 用户(所有者)读、写和执行
S_IRUSR | 用户(所有者)读
S_IWUSR | 用户(所有者)写
S_IXUSR | 用户(所有者)执行
S_IRWXG | 组读、写和执行
S_IRGRP | 组读
S_IWGRP | 组写
S_IXGRP | 组执行
S_IRWXO | 其他读、写和执行
S_IROTH | 其他读
S_IWOTH | 其他写
S_IXOTH | 其他执行




# S_ISVTX位（粘住位）

文件粘住位：如果一个**可执行程序文件的这一位被设置了**，那么在该程序**第一次被执行并结束时，其程序正文部分的一个副本仍被保存在交换区中**，这使得下次执行该程序时能较快地将其装入内存区。交换区占用连续磁盘空间，可将它视为连续文件，而且一个程序的正文部分在交换区中也是连续存放的。现今较新的UNIX系统大多都配置有虚拟存储系统以及快速文件系统，所以**不再需要使用这种技术**。

目录粘住口：如果对一个**目录设置了粘住位**，则只有对该目录具有写权限的用户在**满足下列条件之一的情况下，才能删除**或更名该目录下的文件：1.拥有该文件    2. 拥有该目录   3. 是超级用户。

示例：目录/tmp和/var/spool/uucppublic是设置粘住位的典型候选者——任何用户都可在这两个目录中创建文件。任一用户(用户、组和其他)对这两个目录的权限通常都是读、写和执行。但用户不应删除或更名属于其他人的文件。



# umask
```c
#include <sys/stat.h>
mode_t umask(mode_t cmask);
```
umask的主要作用是在**创建文件时设置或者屏蔽掉文件的一些权限**。在创建一个文件时要指明该文件的权限，**open函数的最后一个参数mode并不是要设置的权限**，它需要执行以下操作**mode & (~cmask)**。







