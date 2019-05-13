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
   
查看指定用户的全部文件描述符信息
      
      lsof -u [USER]
      USER: 用户名
    
 例子:
      
      $ lsof -u dan
      COMMAND   PID USER   FD      TYPE DEVICE  SIZE/OFF      NODE NAME
      sshd     3853  dan  cwd   unknown                            /proc/3853/cwd (readlink: Permission denied)
      sshd     3853  dan  rtd   unknown                            /proc/3853/root (readlink: Permission denied)
      sshd     3853  dan  txt   unknown                            /proc/3853/exe (readlink: Permission denied)
      sshd     3853  dan NOFD                                      /proc/3853/fd (opendir: Permission denied)
      bash     3854  dan  cwd       DIR    8,3        35 103040265 /home/dan/work/37net/example/build/bin
      bash     3854  dan  rtd       DIR    8,3       224        64 /
      bash     3854  dan  txt       REG    8,3    964608 100664772 /usr/bin/bash
      bash     3854  dan  mem       REG    8,3 106075056  33608548 /usr/lib/locale/locale-archive
      bash     3854  dan  mem       REG    8,3     61624     74572 /usr/lib64/libnss_files-2.17.so
      bash     3854  dan  mem       REG    8,3   2151672     74554 /usr/lib64/libc-2.17.so
      bash     3854  dan  mem       REG    8,3     19288     74560 /usr/lib64/libdl-2.17.so
      bash     3854  dan  mem       REG    8,3    174576    109034 /usr/lib64/libtinfo.so.5.9
      bash     3854  dan  mem       REG    8,3    163400     74547 /usr/lib64/ld-2.17.so
      bash     3854  dan  mem       REG    8,3     26254     94467 /usr/lib64/gconv/gconv-modules.cache
      bash     3854  dan    0u      CHR  136,0       0t0         3 /dev/pts/0
      bash     3854  dan    1u      CHR  136,0       0t0         3 /dev/pts/0
      bash     3854  dan    2u      CHR  136,0       0t0         3 /dev/pts/0
      bash     3854  dan  255u      CHR  136,0       0t0         3 /dev/pts/0
      bash     4463  dan  cwd       DIR    8,3      4096 103241941 /home/dan/work/leafserver/src
      bash     4463  dan  rtd       DIR    8,3       224        64 /
      bash     4463  dan  txt       REG    8,3    964608 100664772 /usr/bin/bash
      ......
 
 
查看当前用户网络连接相关的文件描述符信息
    
     lsof -i 
    
例子:
    
    $ lsof -i
    进程名字  进程ID   所属用户  文件描述符  文件类型   设备      文件大小    索引节点   文件名称
    COMMAND   PID     USER     FD         TYPE       DEVICE   SIZE/OFF    NODE      NAME
    echo      10506   dan      3u         IPv4       46129    0t0         TCP       *:7737 (LISTEN)   
    
    
查看当前用户符合条件的网络连接相关的文件描述符信息
    
    lsof -i [ip地址类型] [协议] [@ip] [:服务或者端口]
    ip地址类型: 4代表ipv4、 6代表ipv6
    协议: tcp、udp
    ip: ip地址
    服务或者端口: FTP、SSH、端口号
例子:    

    $ ./echo 
    $ lsof -i tcp:7737
    进程名字  进程ID   所属用户  文件描述符  文件类型   设备      文件大小    索引节点   文件名称
    COMMAND   PID     USER     FD         TYPE       DEVICE   SIZE/OFF    NODE      NAME
    echo      10506   dan      3u         IPv4       46129    0t0         TCP       *:7737 (LISTEN)


查看当前用户指定进程的文件描述符信息
    
    lsof -p [PID]
    PID: 进程id

例子:

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
