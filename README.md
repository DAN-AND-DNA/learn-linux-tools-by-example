# Linux 好用的工具教程

介绍一些好用的linux工具，教程所用的例程: https://github.com/DAN-AND-DNA/37net

## 内容

- [进程](#进程)
  - [lsof命令](#lsof命令)
- [网络](#网络)
  - [netstat命令](#netstat命令)
- [系统](#系统)
  - [free命令](#free命令)


## 进程

    

## 网络
### netstat命令

## 系统
### lsof命令
   lsof即 list open files，可以查看当前系统的打开的文件描述符，和该描述符所属的进程，或者进程所打开的文件描述符。
    
    启动用例
    $ ./echo 
    $ lsof -i  tcp:7737
    进程名字  进程ID   所属用户  文件描述符  文件类型   设备      文件大小    索引节点   文件名称
    COMMAND   PID     USER     FD         TYPE       DEVICE   SIZE/OFF    NODE      NAME
    echo      10506   dan      3u         IPv4       46129    0t0         TCP       *:7737 (LISTEN)
    $ lsof -p 10506
    进程名字  进程ID   所属用户  文件描述符  文件类型   设备      文件大小    索引节点   文件名称
    COMMAND   PID     USER     FD         TYPE       DEVICE   SIZE/OFF    NODE      NAME
    echo      10506   dan      cwd        DIR        8,3      35          103040265 /home/dan/work/37net/example/build/bin
    echo      10506   dan      rtd        DIR        8,3      224         64        /
    echo      10506   dan      txt        REG        8,3      29120       103040261 /home/dan/work/37net/example/build/bin/echo
    echo      10506   dan      mem        REG        8,3      2151672     74554     /usr/lib64/libc-2.17.so
    echo      10506   dan      mem        REG        8,3      163400      74547     /usr/lib64/ld-2.17.so
    echo      10506   dan      0u         CHR        136,0    0t0         3         /dev/pts/0
    echo      10506   dan      1u         CHR        136,0    0t0         3         /dev/pts/0
    echo      10506   dan      2u         CHR        136,0    0t0         3         /dev/pts/0
    echo      10506   dan      3u         IPv4       46129    0t0         TCP       *:7737 (LISTEN)
    echo      10506   dan      4u         a_inode    0,10     0           6397      [eventpoll]
    
    
    
### free命令
