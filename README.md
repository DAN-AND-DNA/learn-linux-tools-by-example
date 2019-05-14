# Linux 好用的工具教程

介绍一些好用的linux工具，教程所用的例程: https://github.com/DAN-AND-DNA/37net

## 内容

- [进程](#进程)
  - [lsof](#lsof)
- [网络](#网络)
  - [netstat命令](#netstat命令)
- [系统](#系统)
  - [free命令](#free命令)


## 进程

    

## 网络
### netstat命令

## 系统
### lsof

#### 输出的结果
    COMMAND   PID     USER        FD      TYPE     DEVICE      SIZE/OFF    NODE      NAME
    进程名字  进程ID   所属用户  文件描述符  文件类型   设备      文件大小    索引节点   文件名称
    
其中 FD:

     cwd 代表当前工作目录
     rtd 代表根目录
     txt 代表程序代码和数据
     mem 代表内存映射的文件
     NOFD 代表文件描述符无法打开
     
     数字+后缀 比如 0u,1u,2u,3u,4r,5w,6u等
     后缀r代表该文件描述符进行读访问
     后缀w代表该文件描述符进行写访问
     后缀u代表该文件描述符进行读写访问
其中 TYPE:
      
      REG     代表普通文件
      DIR     代表目录
      CHR     代表字符文件
      IPV4    代表ipv4套接字
      FIFO    代表FIFO文件
      a_inode 代表系统特殊文件
      unix    代表 UNIX domain 套接字
      PIPE    代表PIPE文件
      

其他详细参数请参考[man lsof]()
     
     
    
#### 查看指定用户的全部文件描述符信息
      
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
 
 
#### 查看当前用户网络相关的文件描述符信息
    
     lsof -i 
    
例子:
    
    $ lsof -i
    COMMAND   PID     USER     FD         TYPE       DEVICE   SIZE/OFF    NODE      NAME
    echo      10506   dan      3u         IPv4       46129    0t0         TCP       *:7737 (LISTEN)   
    
    
#### 查看当前用户符合条件的网络相关的文件描述符信息
    
    lsof -i [ip地址类型] [协议] [@ip] [:服务或者端口]
    ip地址类型: 4代表ipv4、 6代表ipv6
    协议: tcp、udp
    ip: ip地址
    服务或者端口: FTP、SSH、端口号
例子:    

    $ ./echo 
    $ lsof -i tcp:7737
    COMMAND   PID     USER     FD         TYPE       DEVICE   SIZE/OFF    NODE      NAME
    echo      10506   dan      3u         IPv4       46129    0t0         TCP       *:7737 (LISTEN)


#### 查看当前用户指定进程的文件描述符信息
    
    lsof -p PID
    PID: 进程id

例子:

    $ lsof -p 10506
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
    
#### 查看指定的文件描述符的信息
  
    lsof -d FD
    FD: 文件描述符
 例子:
    
    $ lsof -d7
    COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF  NODE NAME
    echo    14323  dan    7u  IPv4  66035      0t0   TCP welugame.com:7737->welugame.com:59542 (ESTABLISHED)
    testpr  14559  dan    7u  IPv4  62438      0t0   TCP welugame.com:59544->welugame.com:7737 (ESTABLISHED)
    lsof    14568  dan    7w  FIFO    0,9      0t0 72855 pipe

#### 查看指定进程名的信息
    
    lsof -c  COMMAND
    N: 进程名
例子:
    
    # lsof -c mysqld (or mysql)
    COMMAND  PID  USER   FD   TYPE             DEVICE  SIZE/OFF      NODE NAME
    mysqld  3526 mysql  cwd    DIR                8,3      4096  34326207 /var/lib/mysql
    mysqld  3526 mysql  rtd    DIR                8,3       224        64 /
    mysqld  3526 mysql  txt    REG                8,3 251284320    481669 /usr/sbin/mysqld
    mysqld  3526 mysql  mem    REG                8,3     61624     74572 /usr/lib64/libnss_files-2.17.so
    mysqld  3526 mysql  mem    REG                8,3    202312  34326204 /usr/lib64/mysql/plugin/validate_password.so
    mysqld  3526 mysql  DEL    REG               0,11               25793 /[aio]
    mysqld  3526 mysql  DEL    REG               0,11               25792 /[aio]
    mysqld  3526 mysql  DEL    REG               0,11               25791 /[aio]
    mysqld  3526 mysql  DEL    REG               0,11               25790 /[aio]
    mysqld  3526 mysql  DEL    REG               0,11               25789 /[aio]
    mysqld  3526 mysql  DEL    REG               0,11               25788 /[aio]
    mysqld  3526 mysql  DEL    REG               0,11               25787 /[aio]
    mysqld  3526 mysql  DEL    REG               0,11               25786 /[aio]
    mysqld  3526 mysql  DEL    REG               0,11               25785 /[aio]
    mysqld  3526 mysql  mem    REG                8,3     11448     64503 /usr/lib64/libfreebl3.so
    mysqld  3526 mysql  mem    REG                8,3   2151672     74554 /usr/lib64/libc-2.17.so
    mysqld  3526 mysql  mem    REG                8,3     88776        84 /usr/lib64/libgcc_s-4.8.5-20150702.so.1
    mysqld  3526 mysql  mem    REG                8,3   1137016     74562 /usr/lib64/libm-2.17.so
    mysqld  3526 mysql  mem    REG                8,3    991616     94474 /usr/lib64/libstdc++.so.6.0.19
    mysqld  3526 mysql  mem    REG                8,3     43776     74584 /usr/lib64/librt-2.17.so
    mysqld  3526 mysql  mem    REG                8,3     19288     74560 /usr/lib64/libdl-2.17.so
    mysqld  3526 mysql  mem    REG                8,3     40664     74558 /usr/lib64/libcrypt-2.17.so
    mysqld  3526 mysql  mem    REG                8,3     50752    313857 /usr/lib64/libnuma.so.1
    mysqld  3526 mysql  mem    REG                8,3      6264    287120 /usr/lib64/libaio.so.1.0.1
    mysqld  3526 mysql  mem    REG                8,3    141968     74580 /usr/lib64/libpthread-2.17.so
    mysqld  3526 mysql  mem    REG                8,3    163400     74547 /usr/lib64/ld-2.17.so
    mysqld  3526 mysql  DEL    REG               0,11               25784 /[aio]
    mysqld  3526 mysql  DEL    REG               0,11               25783 /[aio]
    mysqld  3526 mysql  DEL    REG               0,11               25782 /[aio]
    mysqld  3526 mysql    0r   CHR                1,3       0t0      1028 /dev/null
    
#### 刷新信息

    $ lsof -r T
    T: 刷新时间
例子:

    $ lsof -c echo -r 1
    COMMAND   PID USER   FD      TYPE DEVICE SIZE/OFF      NODE NAME
    echo    14323  dan  cwd       DIR    8,3       35 103040265 /home/dan/work/37net/example/build/bin
    echo    14323  dan  rtd       DIR    8,3      224        64 /
    echo    14323  dan  txt       REG    8,3    29120 103040261 /home/dan/work/37net/example/build/bin/echo
    echo    14323  dan  mem       REG    8,3  2151672     74554 /usr/lib64/libc-2.17.so
    echo    14323  dan  mem       REG    8,3   163400     74547 /usr/lib64/ld-2.17.so
    echo    14323  dan    0u      CHR  136,1      0t0         4 /dev/pts/1
    echo    14323  dan    1u      CHR  136,1      0t0         4 /dev/pts/1
    echo    14323  dan    2u      CHR  136,1      0t0         4 /dev/pts/1
    echo    14323  dan    3u     IPv4  62358      0t0       TCP *:7737 (LISTEN)
    echo    14323  dan    4u  a_inode   0,10        0      6397 [eventpoll]
    =======
    COMMAND   PID USER   FD      TYPE DEVICE SIZE/OFF      NODE NAME
    echo    14323  dan  cwd       DIR    8,3       35 103040265 /home/dan/work/37net/example/build/bin
    echo    14323  dan  rtd       DIR    8,3      224        64 /
    echo    14323  dan  txt       REG    8,3    29120 103040261 /home/dan/work/37net/example/build/bin/echo
    echo    14323  dan  mem       REG    8,3  2151672     74554 /usr/lib64/libc-2.17.so
    echo    14323  dan  mem       REG    8,3   163400     74547 /usr/lib64/ld-2.17.so
    echo    14323  dan    0u      CHR  136,1      0t0         4 /dev/pts/1
    echo    14323  dan    1u      CHR  136,1      0t0         4 /dev/pts/1
    echo    14323  dan    2u      CHR  136,1      0t0         4 /dev/pts/1
    echo    14323  dan    3u     IPv4  62358      0t0       TCP *:7737 (LISTEN)
    echo    14323  dan    4u  a_inode   0,10        0      6397 [eventpoll]
### free命令
