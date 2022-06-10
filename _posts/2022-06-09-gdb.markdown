---
layout: post
title:  "Ubuntu系统生成dump文件，然后用gdb来调试"
date:   2022-06-09 15:41:32 +0800
categories: blog
---
# 1.设置生成dump文件

　　用ulimit -c查看当前设置是否生成dump文件。如果为0(一般默认为0)，则表示不生成dump文件。用ulimit -c unlimited命令修改成生成dump文件。修改完后再用ulimit -c查看一下，返回unlimited，表示生成dump文件。

# 2.设置dump文件的保存位置

　　用/proc/sys/kernel/core_pattern查看当前设置的dump文件保存的位置。一般默认是将dump文件交给一个approve的守护进程处理，所以如果不修改dump文件的保存路径的话，你将不会看到生成的dump文件。用echo "/corefile/core-%e-%p-%t" \| sudo dd of=/proc/sys/kernel/core_pattern将dump文件保存的位置修改成“/corefile/core-%e-%p-%t”。"%e-%p-%t"分别表示文件名称加入执行文件名称，添加进程的pid，添加时间，前面的路径"/corefile"可以根据你自己的需要进行设置。用cat /proc/sys/kernel/core_pattern查看修改是否成功。如果你想在docker里面生成dump文件，直接在docker中修改可能会失败，只需要修改宿主机上的保存路径即可，docker里面会继承宿主机的修改。

# 3.运行程序，产生崩溃

　　配置好后，运行程序，产生崩溃。在你设置的dump文件保存目录下会生成相应的dump文件。

# 4.用gdb来调试

gdb ./应用程序  core.xxxx

`就会恢复现场到你的程序崩溃的那一刻`

(gdb) bt

`这个命令会列出程序崩溃时的堆栈信息，一层一层会有标号  #0  #1  #2 .......
如果你要查看某一层的信息，你需要在切换当前的栈，一般来说，程序停止时，最顶层的栈就是当前栈，如果你要查看栈下面层的详细信息，首先要做的是切换到你想看的栈`

(gdb) f  N

`N是你想要切换的栈的标号，达到后可以用  ‘p  变量’  查看变量的值，以查找异常出现的原因`

info args

`打印出当前函数的参数名及其值。`

info locals

`打印出当前函数中所有局部变量及其值。`

info catch

`打印出当前的函数中的异常处理信息。`
