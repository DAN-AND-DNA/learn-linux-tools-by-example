# Linux 好用的工具教程

介绍一些好用的linux工具，教程所用的例程: https://github.com/DAN-AND-DNA/37net

## 内容

- [进程](#进程)
  - [valgrind](#valgrind)
  - [KCachegrind](#KCachegrind)
  - [proc](#proc)
- [网络](#网络)
  - [ss](#ss)
  - [netstat](#netstat)
- [系统](#系统)
  - [lsof](#lsof)
  - [dstat](#dstat)
  - [free](#free)


## 进程
## valgrind
利用valgrind进行函数调用剖析，例如:
      
    $ valgrind --tool=callgrind ./echo
    ==4137== Callgrind, a call-graph generating cache profiler
    ==4137== Copyright (C) 2002-2017, and GNU GPL'd, by Josef Weidendorfer et al.
    ==4137== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
    ==4137== Command: ./echo
    ==4137== 
    ==4137== For interactive control, run 'callgrind_control -h'.
    pagesize:4096
    count :0
    ==4137== 
    ==4137== Events    : Ir
    ==4137== Collected : 296028
    ==4137== 
    ==4137== I   refs:      296,028

就会生成 callgrind.out.4137文件，可以用callgrind_annotate打开，其中左侧的Ir代表指令数，约大的值越是优化的热点参考位置，例如:
    
    $ callgrind_annotate callgrind.out.4137  /home/dan/work/37net/src/37EventLoop.c | grep -v "???"
    --------------------------------------------------------------------------------
    Profile data file 'callgrind.out.4137' (creator: callgrind-3.13.0)
    --------------------------------------------------------------------------------
    I1 cache: 
    D1 cache: 
    LL cache: 
    Timerange: Basic block 0 - 48715
    Trigger: Program termination
    Profiled target:  ./echo (PID 4137, part 1)
    Events recorded:  Ir
    Events shown:     Ir
    Event sort order: Ir
    Thresholds:       99
    Include dirs:     
    User annotated:   /home/dan/work/37net/src/37EventLoop.c
    Auto-annotation:  off

    --------------------------------------------------------------------------------
        Ir 
    --------------------------------------------------------------------------------
    296,028  PROGRAM TOTALS

    --------------------------------------------------------------------------------
     Ir  file:function
    --------------------------------------------------------------------------------
    120,092  /home/dan/work/37net/src/37EventLoop.c:CreateEventLoop [/home/dan/work/37net/example/build/bin/echo]
        140  /home/dan/work/37net/src/37Socket.c:CreateServer [/home/dan/work/37net/example/build/bin/echo]

    --------------------------------------------------------------------------------
    -- User-annotated source: /home/dan/work/37net/src/37EventLoop.c
    --------------------------------------------------------------------------------
        Ir 

    -- line 9 ----------------------------------------
        .  typedef struct Epoll37 {
        .      int                     m_iEfd;
        .      struct epoll_event*     m_EpollEvents;
        .  } Epoll37;
        .  
        .  int iDone = 0;
        .  
        .  inline EventLoop37* CreateEventLoop(int iSize)
        4  {
        1      EventLoop37* pstEventLoop = NULL;
        10      if ((pstEventLoop = malloc(sizeof(EventLoop37))) == NULL)
        .      {
        .          goto ERROR;
        .      }
        .  
        3      pstEventLoop->m_iSize = iSize;
        9      pstEventLoop->m_pstEvents = malloc(sizeof(Event37) * (uint32_t)(iSize));
        9      pstEventLoop->m_pstFiredEvents = malloc(sizeof(FiredEvent37) * (uint32_t)(iSize));
        .  
        8      if(pstEventLoop->m_pstEvents == NULL || pstEventLoop->m_pstFiredEvents == NULL)
        .      {
        .          goto ERROR;
        .      }
        .  
        4      Epoll37* pstEpoll = malloc(sizeof(Epoll37));
        2      if(pstEpoll == NULL)
        .      {
        .          goto ERROR;
        .      }
        .  
        12      pstEpoll->m_EpollEvents = malloc(sizeof(struct epoll_event) * (uint32_t)(iSize));
        4      if(pstEpoll->m_EpollEvents == NULL)
        .      {
        .          goto ERROR;
        .      }
        .  
        9      pstEpoll->m_iEfd = epoll_create1(EPOLL_CLOEXEC);
        4      if(pstEpoll->m_iEfd == -1)
        .      {
        .          goto ERROR;
        .      }
        .      
        .      int i;
    50,006      for(i = 0; i < pstEventLoop->m_iSize; i++)
        .      {
    70,000          pstEventLoop->m_pstEvents[i].m_iAction = 0;
        .      }
        .  
        3      pstEventLoop->m_pstEpoll = pstEpoll;
        2      return pstEventLoop;
        .  
        .  ERROR:
        .      if(pstEpoll != NULL)
        .      {
        .          free(pstEpoll->m_EpollEvents);
        .          free(pstEpoll);
        .      }
        .  
    -- line 66 ----------------------------------------
    -- line 68 ----------------------------------------
        .      {
        .          free(pstEventLoop->m_pstEvents);
        .          free(pstEventLoop->m_pstFiredEvents);
        .          free(pstEventLoop);
        .      }
        .  
        .      return NULL;
        .  
        2  }
        .  
        .  inline int AddEvent(EventLoop37* pstEventLoop, int iFd, int iAction, Handle* pstHandle, void* pstUserData)
        8  {
        4      if(iFd > pstEventLoop->m_iSize)
        .      {
        .          return -1;
        .      }
        .  
        7      Event37* e = &(pstEventLoop->m_pstEvents[iFd]);
        3      Epoll37* ep = pstEventLoop->m_pstEpoll;
        .      
        .      struct epoll_event ee;
        .     // bzero(&ee, sizeof(struct epoll_event));
        .     // ee.events = 0;
        .      
        1      int op = EPOLL_CTL_ADD;
        4      if(e->m_iAction != 0)
        .      {
        .          op = EPOLL_CTL_MOD;
        .      }
        .  
        6      e->m_iAction |= iAction;
        3      e->m_pstUserData = pstUserData;
        .      
        4      if(iAction & EPOLLIN)
        .      {
        3          e->m_pstHandleRead = pstHandle;
        .      }
        .  
        4      if(iAction & EPOLLOUT)
        .      {
        .          e->m_pstHandleWrite = pstHandle;
        .      }
        3      ee.events = (uint32_t)(e->m_iAction);
        2      ee.data.fd = iFd;
        14      if(epoll_ctl(ep->m_iEfd, op, iFd, &ee) == -1) 
        .      {
        .          return -1;
        .      }
        .  
        1      return 0;
        2  }
        .  
        .  inline int DelEvent(EventLoop37* pstEventLoop, int iFd, int iAction)
        .  {
        .      if(iFd > pstEventLoop->m_iSize)
        .      {
        .          return -1;
        .      }
        .  
    -- line 126 ----------------------------------------
    -- line 152 ----------------------------------------
        .          }
        .  
        .      }
        .      return 0;
        .  }
        .  
        .  
        .  inline int Poll(EventLoop37* pstEventLoop)
        4  {
        3      Epoll37* ep = pstEventLoop->m_pstEpoll;
        15      int fired = epoll_wait(ep->m_iEfd, ep->m_EpollEvents, pstEventLoop->m_iSize, 10000);
        .  
        .      int i;
        5      for(i = 0; i < fired; i++)
        .      {
        .          struct epoll_event* e = ep->m_EpollEvents + i; 
        .          if(e->events & (EPOLLIN | EPOLLPRI | EPOLLRDHUP))
        .          {
        .              pstEventLoop->m_pstFiredEvents[i].m_iFd  = e->data.fd;
        .              pstEventLoop->m_pstFiredEvents[i].m_iAction = EPOLLIN;
        .          }
        .  
        .          if(e->events & EPOLLOUT)
        .          {    
        .              pstEventLoop->m_pstFiredEvents[i].m_iFd  = e->data.fd;
        .              pstEventLoop->m_pstFiredEvents[i].m_iAction = EPOLLOUT;
        .          }
        .      }
        1      return fired;
        2  }
        .  
        .  
        .  inline int ProcessFired(EventLoop37* pstEventLoop)
        4  {
        4      int fired = Poll(pstEventLoop);
       867  => /home/dan/work/37net/src/37EventLoop.c:Poll (1x)
        .    
        .      int i;
        5      for(i = 0; i < fired; i++)
        .      {
        .          int fd = pstEventLoop->m_pstFiredEvents[i].m_iFd;
        .          int iAction = pstEventLoop->m_pstFiredEvents[i].m_iAction;
        .          Event37* e = &(pstEventLoop->m_pstEvents[fd]);
        .          if(iAction & EPOLLIN)
        .          {
        .              e->m_pstHandleRead(pstEventLoop, fd, e->m_pstUserData, iAction);
        .          }
    -- line 197 ----------------------------------------
    -- line 198 ----------------------------------------
        .          if(iAction & EPOLLOUT)
        .          {
        .              e->m_pstHandleWrite(pstEventLoop, fd, e->m_pstUserData, iAction);
        .          }
        .  
        .          iDone++;
        .      }
        .         
        1      return iDone;
        2  }
        .  
        .  
        .  inline void Run(EventLoop37* pstEventLoop)
        4  {
        .     // while(!pstEventLoop->m_iStop)
        .      {
        3          ProcessFired(pstEventLoop);
      883  => /home/dan/work/37net/src/37EventLoop.c:ProcessFired (1x)
        .      }
        2  }
        .  
        .  inline void Stop(EventLoop37* pstEventLoop)
        .  {
        .      Epoll37* ep = pstEventLoop->m_pstEpoll;
        .      close(ep->m_iEfd);
        .      free(ep->m_EpollEvents);
        .      free(pstEventLoop->m_pstEpoll);
        .      free(pstEventLoop->m_pstEvents);
    -- line 224 ----------------------------------------

    --------------------------------------------------------------------------------
    Ir 
    --------------------------------------------------------------------------------
    41  percentage of events annotated

## KCachegrind
如果Linux里有KDE可以用KCachegrind打开callgrind.out.* ，如果需要显示源代码，需要在Settings里Configure KCachegrind添加源代码位置，例如:
    
    Ir 代表指令数热点，也就是需要进行优化的参考位置。
    Ir per call 代表指令的数量
![demo](https://github.com/DAN-AND-DNA/learn-linux-tools-by-example/blob/master/img/2019516-102102.jpg)



## proc
可以通过该文件查看进程的内存信息

查看指定进程的代码区、数据区、堆区、共享库、共享内存、内存映射、栈区、内核区的虚拟内存空间地址，例如:
    
    结果每个域为
        虚拟内存开始地址-结束地址 
        可读|可写|可执行  私有|共享 
        内存映射的文件偏移量 
        设备号 索引节点|文件号             
        路径
        
    其中输出结果中没有路径名的为mmap创建的匿名映射
    
    $ cat /proc/16078/maps 
    
    00400000-00403000                 r-xp 00000000 08:03 103040261                  /home/dan/work/37net/example/build/bin/echo
    00602000-00603000                 r--p 00002000 08:03 103040261                  /home/dan/work/37net/example/build/bin/echo
    00603000-00604000                 rw-p 00003000 08:03 103040261                  /home/dan/work/37net/example/build/bin/echo
    02470000-024c1000                 rw-p 00000000 00:00 0                          [heap]
    7f065caba000-7f065cc7c000         r-xp 00000000 08:03 74554                      /usr/lib64/libc-2.17.so
    7f065cc7c000-7f065ce7c000         ---p 001c2000 08:03 74554                      /usr/lib64/libc-2.17.so
    7f065ce7c000-7f065ce80000         r--p 001c2000 08:03 74554                      /usr/lib64/libc-2.17.so
    7f065ce80000-7f065ce82000         rw-p 001c6000 08:03 74554                      /usr/lib64/libc-2.17.so
    7f065ce82000-7f065ce87000         rw-p 00000000 00:00 0 
    7f065ce87000-7f065cea9000         r-xp 00000000 08:03 74547                      /usr/lib64/ld-2.17.so
    7f065d04e000-7f065d0a0000         rw-p 00000000 00:00 0 
    7f065d0a6000-7f065d0a8000         rw-p 00000000 00:00 0 
    7f065d0a8000-7f065d0a9000         r--p 00021000 08:03 74547                      /usr/lib64/ld-2.17.so
    7f065d0a9000-7f065d0aa000         rw-p 00022000 08:03 74547                      /usr/lib64/ld-2.17.so
    7f065d0aa000-7f065d0ab000         rw-p 00000000 00:00 0 
    7ffe80a4d000-7ffe80a6e000         rw-p 00000000 00:00 0                          [stack]
    7ffe80b7b000-7ffe80b7d000         r-xp 00000000 00:00 0                          [vdso]
    ffffffffff600000-ffffffffff601000 r-xp 00000000 00:00 0                          [vsyscall]

    
    


查看指定进程的总体内存信息，例如:

    输出结果中
        VmSize 代表占用虚拟内存的大小
        VmRSS 代表实际使用的物理内存的大小
        VmStk 代表使用的栈虚拟内存大小
        VmData 代表数据段的虚拟内存大小
        VmExe 代表代码段虚拟内存大小
        VmLib 代表动态库的虚拟内存大小
        Threads 代表已经使用的线程数
        
        
    $ cat /proc/16078/status
    Name:   echo
    Umask:  0002
    State:  S (sleeping)
    Tgid:   16078
    Ngid:   0
    Pid:    16078
    PPid:   14273
    TracerPid:      0
    Uid:    1000    1000    1000    1000
    Gid:    1000    1000    1000    1000
    FDSize: 256
    Groups: 10 1000 
    VmPeak:     4860 kB
    VmSize:     4860 kB
    VmLck:         0 kB
    VmPin:         0 kB
    VmHWM:       880 kB
    VmRSS:       880 kB
    RssAnon:             416 kB
    RssFile:             464 kB
    RssShmem:              0 kB
    VmData:      692 kB
    VmStk:       132 kB
    VmExe:        12 kB
    VmLib:      1936 kB
    VmPTE:        32 kB
    VmSwap:        0 kB
    Threads:        1
    SigQ:   0/31936
    SigPnd: 0000000000000000
    ShdPnd: 0000000000000000
    SigBlk: 0000000000000000
    SigIgn: 0000000000001000
    SigCgt: 0000000000000002
    CapInh: 0000000000000000
    CapPrm: 0000000000000000
    CapEff: 0000000000000000
    CapBnd: 0000001fffffffff
    CapAmb: 0000000000000000
    Seccomp:        0
    Speculation_Store_Bypass:       vulnerable
    Cpus_allowed:   3f
    Cpus_allowed_list:      0-5
    Mems_allowed:                           00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000001
    Mems_allowed_list:      0
    voluntary_ctxt_switches:        147116
    nonvoluntary_ctxt_switches:     392
    
查看一个进程的代码区、数据区、堆区、共享库、共享内存、内存映射、栈区、内核区所占用的内存信息，例如:
   
    输出结果中
        Size 代表占用虚拟内存的大小
        Rss 代表实际使用物理内存(包含共享库占用的内存)
        Shared_Clean 代表未被修改的共享内存page大小，换页无需写入磁盘
        Shared_Dirty 代表被修改了共享内存page大小，换页需写入磁盘
        Private_Clean 代表未被修改的私有内存page大小，换页无需写入磁盘
        Private_Dirty 代表被修改的私有内存page大小，换页需写入磁盘
        
    
    $ cat /proc/16078/smaps
    00400000-00403000 r-xp 00000000 08:03 103040261                          /home/dan/work/37net/example/build/bin/echo
    Size:                 12 kB
    Rss:                   8 kB
    Pss:                   8 kB
    Shared_Clean:          0 kB
    Shared_Dirty:          0 kB
    Private_Clean:         8 kB
    Private_Dirty:         0 kB
    Referenced:            8 kB
    Anonymous:             0 kB
    AnonHugePages:         0 kB
    Swap:                  0 kB
    KernelPageSize:        4 kB
    MMUPageSize:           4 kB
    Locked:                0 kB
    VmFlags: rd ex mr mp me dw sd 
    00602000-00603000 r--p 00002000 08:03 103040261                          /home/dan/work/37net/example/build/bin/echo
    Size:                  4 kB
    Rss:                   4 kB
    Pss:                   4 kB
    Shared_Clean:          0 kB
    Shared_Dirty:          0 kB
    Private_Clean:         0 kB
    Private_Dirty:         4 kB
    Referenced:            4 kB
    Anonymous:             4 kB
    AnonHugePages:         0 kB
    Swap:                  0 kB
    KernelPageSize:        4 kB
    MMUPageSize:           4 kB
    Locked:                0 kB
    VmFlags: rd mr mp me dw ac sd 
    00603000-00604000 rw-p 00003000 08:03 103040261                          /home/dan/work/37net/example/build/bin/echo
    Size:                  4 kB
    Rss:                   4 kB
    Pss:                   4 kB
    Shared_Clean:          0 kB
    Shared_Dirty:          0 kB
    Private_Clean:         0 kB
    Private_Dirty:         4 kB
    Referenced:            4 kB
    Anonymous:             4 kB
    AnonHugePages:         0 kB
    Swap:                  0 kB
    KernelPageSize:        4 kB
    MMUPageSize:           4 kB
    Locked:                0 kB
    VmFlags: rd wr mr mp me dw ac sd 
    02470000-024c1000 rw-p 00000000 00:00 0                                  [heap]
    Size:                324 kB
    Rss:                  12 kB
    Pss:                  12 kB
    Shared_Clean:          0 kB
    Shared_Dirty:          0 kB
    Private_Clean:         0 kB
    Private_Dirty:        12 kB
    Referenced:           12 kB
    Anonymous:            12 kB
    AnonHugePages:         0 kB
    Swap:                  0 kB
    KernelPageSize:        4 kB
    MMUPageSize:           4 kB
    Locked:                0 kB
    VmFlags: rd wr mr mp me ac sd 
    7f065caba000-7f065cc7c000 r-xp 00000000 08:03 74554                      /usr/lib64/libc-2.17.so
    Size:               1800 kB
    Rss:                 372 kB
    Pss:                  20 kB
    Shared_Clean:        372 kB
    Shared_Dirty:          0 kB
    Private_Clean:         0 kB
    Private_Dirty:         0 kB
    Referenced:          372 kB
    Anonymous:             0 kB
    AnonHugePages:         0 kB
    Swap:                  0 kB
    KernelPageSize:        4 kB
    MMUPageSize:           4 kB
    Locked:                0 kB
    VmFlags: rd ex mr mp me sd 
    ......

## 网络
### ss
可以通过该工具查看网络连接的信息

查看本地监听socket的信息，例如:

    $ ss -ntupl
    Netid  State      Recv-Q Send-Q          Local Address:Port           Peer Address:Port              
    udp    UNCONN     0      0               *:68                         *:*                  
    tcp    LISTEN     0      128             *:22                         *:*                  
    tcp    LISTEN     0      128             *:7737                       *:*                   users:(("echo",pid=15870,fd=3))
    tcp    LISTEN     0      80              :::3306                      :::*                  
    tcp    LISTEN     0      128             :::22                        :::* 
    
    其中参数
    -n 代表显示IP地址
    -t 代表显示tcp连接
    -u 代表显示udp连接
    -p 显示使用该socket的进程
    -l 显示监听socket
    
启动客户端，查看连接，例如:    
   
    $ ss -ntup
    Netid  State      Recv-Q Send-Q        Local Address:Port           Peer Address:Port                   
    tcp    ESTAB      370828 0             192.168.0.45:7737            192.168.0.115:42188        
    
    
显示更多tcp连接细节，例如:
    
    $ ss -ntupi
    Netid  State      Recv-Q Send-Q       Local Address:Port            Peer Address:Port              
    tcp    ESTAB      0      0            192.168.0.45:7737             192.168.0.115:42188         users:(("echo",pid=15870,fd=5))
         cubic wscale:7,7 rto:205 rtt:3/1.5 ato:40 mss:1448 rcvmss:1448 advmss:1448 cwnd:10 bytes_received:3008400 segs_out:1981 segs_in:40739 send 38.6Mbps lastsnd:4772 lastack:1677 pacing_rate 77.2Mbps rcv_rtt:5.75 rcv_space:29153                                       
    其中参数
    -i 代表显示更多tcp细节

其他详细参数请参考[man ss]()

### netstat
可以通过该工具查看网络连接的情况
  
    # netstat -tnap
    Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      3356/sshd           
    tcp        0      0 0.0.0.0:7737            0.0.0.0:*               LISTEN      16078/./echo        
    tcp6       0      0 :::3306                 :::*                    LISTEN      3526/mysqld         
    tcp6       0      0 :::22                   :::*                    LISTEN      3356/sshd       

    其中参数
    -t 代表显示tcp连接
    -n 代表显示ip而不进行解析
    -a 代表显示处于监听和非监听的socket信息
    -p 显示使用该socket的进程

启动客户端
    
    # netstat -tnap
    Active Internet connections (servers and established)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      3356/sshd           
    tcp        0      0 0.0.0.0:7737            0.0.0.0:*               LISTEN      16078/./echo        
    tcp     2680      0 192.168.0.45:7737       192.168.0.115:42190     ESTABLISHED 16078/./echo        
    tcp6       0      0 :::3306                 :::*                    LISTEN      3526/mysqld         
    tcp6       0      0 :::22                   :::*                    LISTEN      3356/sshd  
    
其他详细参数请参考[man netstat]()

## 系统
### lsof
可以通过该工具查看进程的文件描述符，输出的结果如:

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
     
     
    
查看指定用户的全部文件描述符信息，例子:
      
      lsof -u [USER]
      USER: 用户名
      
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
 
 
查看当前用户网络相关的文件描述符信息，例子:
    
     lsof -i 
    
    $ lsof -i
    COMMAND   PID     USER     FD         TYPE       DEVICE   SIZE/OFF    NODE      NAME
    echo      10506   dan      3u         IPv4       46129    0t0         TCP       *:7737 (LISTEN)   
    
    
查看当前用户符合条件的网络相关的文件描述符信息，例子:
    
    lsof -i [ip地址类型] [协议] [@ip] [:服务或者端口]
    ip地址类型: 4代表ipv4、 6代表ipv6
    协议: tcp、udp
    ip: ip地址
    服务或者端口: FTP、SSH、端口号    

    $ ./echo 
    $ lsof -i tcp:7737
    COMMAND   PID     USER     FD         TYPE       DEVICE   SIZE/OFF    NODE      NAME
    echo      10506   dan      3u         IPv4       46129    0t0         TCP       *:7737 (LISTEN)


查看当前用户指定进程的文件描述符信息，例子:
    
    lsof -p PID
    PID: 进程id

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
    
查看指定的文件描述符的信息，例子:
  
    lsof -d FD
    FD: 文件描述符
    
    $ lsof -d7
    COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF  NODE NAME
    echo    14323  dan    7u  IPv4  66035      0t0   TCP welugame.com:7737->welugame.com:59542 (ESTABLISHED)
    testpr  14559  dan    7u  IPv4  62438      0t0   TCP welugame.com:59544->welugame.com:7737 (ESTABLISHED)
    lsof    14568  dan    7w  FIFO    0,9      0t0 72855 pipe

查看指定进程名的信息，例子:
    
    lsof -c  COMMAND
    COMMAND: 进程名
    
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
    
定时刷新信息，例子:

    $ lsof -r T
    T: 刷新时间

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
    
### dstat
python实现的系统监测工具

查看 时间、cpu、磁盘、中断、上下文切换、网络包、网络字节、内存、页，例如:

    $ dstat -t --cpu --disk --sys --net-packet --net --mem --page
    ----system---- ----total-cpu-usage---- -dsk/total- ---system-- -pkt/total- -net/total- ------memory-usage----- ---paging--
         time     |usr sys idl wai hiq siq| read  writ| int   csw |#recv #send| recv  send| used  buff  cach  free|  in   out 
    14-05 12:52:16|  0   1  99   0   0   0|4447B 2902B| 923  2336 |   0     0 |   0     0 | 395M 2076k  435M 7173M|   0     0 
    14-05 12:52:17|  0   0 100   0   0   0|   0     0 |  79    82 |1.00  1.00 |  60B 1318B| 395M 2076k  435M 7173M|   0     0 
    14-05 12:52:18|  0   0 100   0   0   0|   0     0 |  41    58 |1.00  1.00 |  60B  550B| 395M 2076k  435M 7173M|   0     0 
    14-05 12:52:19|  0   0 100   0   0   0|   0     0 |  68    80 |2.00  2.00 | 120B  610B| 395M 2076k  435M 7173M|   0     0 
    14-05 12:52:20|  0   0 100   0   0   0|   0     0 |  47    57 |1.00  1.00 |  60B  550B| 395M 2076k  435M 7173M|   0     0 
    14-05 12:52:21|  0   0 100   0   0   0|   0     0 |  51    64 |1.00  1.00 |  60B  550B| 395M 2076k  435M 7173M|   0     0 
    14-05 12:52:22|  0   0 100   0   0   0|   0     0 |  42    55 |1.00  1.00 |  60B  550B| 395M 2076k  435M 7173M|   0     0 
    14-05 12:52:23|  0   0 100   0   0   0|   0     0 |  61    75 |1.00  1.00 |  60B  550B| 395M 2076k  435M 7173M|   0     0 
    14-05 12:52:24|  0   0 100   0   0   0|   0     0 |  48    61 |1.00  1.00 |  60B  550B| 395M 2076k  435M 7173M|   0     0 
    14-05 12:52:25|  0   0 100   0   0   0|   0     0 |  42    64 |1.00  1.00 |  60B  550B| 395M 2076k  435M 7173M|   0     0 
    14-05 12:52:26|  0   0 100   0   0   0|   0     0 |  41    53 |1.00  1.00 |  60B  550B| 395M 2076k  435M 7173M|   0     0 
    14-05 12:52:27|  0   0 100   0   0   0|   0     0 |  63    84 |1.00  1.00 |  60B  550B| 395M 2076k  435M 7173M|   0     0 
    
其中输出的结果中:      
      
      time 代表输出时间
      usr sys idl wai hiq siq 代表 用户进程占用cpu时间 系统进程占用 空闲 等待io 硬中断 软中断
      read  writ 代表磁盘读写 
      int   csw  代表系统中断和上下文切换
      #recv #send 代表收发包 
      recv  send 代表收发字节数
      used  buff  cach  free 代表内存占用 写缓存的量 读缓存的量 空闲的内存量
      in   out 代表页换入 页换出 
启动测试客户端和服务器:
    
    ----system---- ----total-cpu-usage---- -dsk/total- ---system-- -pkt/total- -net/total- ------memory-usage----- ---paging--
         time     |usr sys idl wai hiq siq| read  writ| int   csw |#recv #send| recv  send| used  buff  cach  free|  in   out 
    14-05 12:52:28|  0   0 100   0   0   0|   0     0 |1291  1345 | 804   644 |  53k   81k| 395M 2076k  435M 7173M|   0     0 
    14-05 12:52:29|  0   0 100   0   0   0|   0     0 |1022   988 | 572   456 |  38k   58k| 396M 2076k  435M 7172M|   0     0 
    14-05 12:52:30|  0   0  99   0   0   0|   0     0 |1282  1293 | 771   618 |  51k   78k| 397M 2076k  435M 7171M|   0     0 
    14-05 12:52:31|  0   0  98   0   0   1|   0     0 |1248  1559 |11.1k 11.0k| 854k  798k| 401M 2076k  435M 7167M|   0     0 
    14-05 12:52:32|  0   3  91   0   0   5|   0     0 |7755  1104 |20.2k 19.5k|1571k 1280k| 400M 2076k  435M 7168M|   0     0 
    14-05 12:52:33|  0   2  92   0   0   6|   0     0 |9155  1394 |19.4k 19.0k|1449k 1278k| 401M 2076k  435M 7167M|   0     0 
    14-05 12:52:34|  0   3  93   0   0   4|   0     0 |6139  1484 |18.9k 18.0k|1558k 1196k| 400M 2076k  435M 7168M|   0     0 
    14-05 12:52:35|  0   3  90   0   0   7|   0     0 |  11k  911 |21.7k 19.7k|1773k 1285k| 400M 2076k  435M 7168M|   0     0 
    14-05 12:52:36|  0   3  90   0   0   7|   0     0 |  11k  957 |20.8k 18.7k|1679k 1226k| 400M 2076k  435M 7168M|   0     0 
    14-05 12:52:37|  1   3  90   0   0   7|   0     0 |9499  1063 |20.1k 18.7k|1649k 1238k| 400M 2076k  435M 7168M|   0     0 
    14-05 12:52:38|  0   3  90   0   0   7|   0     0 |  10k 1018 |20.3k 18.6k|1671k 1215k| 400M 2076k  435M 7168M|   0     0 
    14-05 12:52:39|  0   4  89   0   0   7|   0     0 |  12k  875 |20.6k 18.0k|1718k 1168k| 398M 2076k  435M 7170M|   0     0 
    14-05 12:52:40|  0   3  91   0   0   7|   0     0 |8020  1014 |19.5k 18.4k|1592k 1229k| 399M 2076k  435M 7169M|   0     0 
    14-05 12:52:41|  0   2  90   0   0   8|   0     0 |  12k 1492 |19.1k 18.3k|1503k 1186k| 398M 2076k  435M 7170M|   0     0 
    14-05 12:52:42|  0   3  90   0   0   7|   0     0 |  10k 1145 |19.3k 17.9k|1587k 1173k| 398M 2076k  435M 7170M|   0     0 
    14-05 12:52:43|  0   3  89   0   0   8|   0     0 |  13k  918 |20.7k 18.2k|1709k 1176k| 398M 2076k  435M 7170M|   0     0 
    14-05 12:52:44|  0   3  90   0   0   8|   0     0 |  13k 1288 |20.1k 18.7k|1601k 1207k| 398M 2076k  435M 7170M|   0     0 
    14-05 12:52:45|  0   3  89   0   0   7|   0     0 |  11k 1326 |19.3k 17.7k|1570k 1194k| 398M 2076k  435M 7170M|   0     0 
    14-05 12:52:46|  0   1  98   0   0   1|   0     0 |1815   942 | 489   404 |  67k   39k| 398M 2076k  435M 7171M|   0     0 
    14-05 12:52:47|  0   1  98   0   0   0|   0     0 | 444   237 | 106  86.0 |  57k 7652B| 397M 2076k  435M 7171M|   0     0 
    14-05 12:52:48|  0   1  99   0   0   0|   0     0 | 294   151 |57.0  47.0 |  28k 4326B| 397M 2076k  435M 7171M|   0     0 
    14-05 12:52:49|  0   1  99   0   0   0|   0     0 | 300   170 |43.0  37.0 |  18k 3774B| 397M 2076k  435M 7171M|   0     0 
    14-05 12:52:50|  0   1  99   0   0   0|   0     0 | 252   148 |40.0  36.0 |  17k 3612B| 397M 2076k  435M 7172M|   0     0 
    14-05 12:52:51|  0   0  99   0   0   0|   0     0 | 236   151 |26.0  24.0 |  10k 2616B| 397M 2076k  435M 7172M|   0     0 
    14-05 12:52:52|  0   0 100   0   0   0|   0     0 |  85    70 |9.00  8.00 |4344B 1124B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:52:53|  0   1  99   0   0   0|   0     0 | 205   109 |28.0  24.0 |  13k 2516B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:52:54|  0   0  99   0   0   0|   0     0 | 195   119 |30.0  26.0 |  16k 2648B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:52:55|  0   0 100   0   0   0|   0     0 | 160   116 |25.0  21.0 |  12k 2206B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:52:56|  0   1  99   0   0   0|   0     0 | 240   119 |41.0  37.0 |  21k 3494B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:52:57|  0   1  99   0   0   0|   0     0 | 215   118 |47.0  40.0 |  26k 3668B| 396M 2076k  435M 7172M|   0     0 
断开客户端:
    
    ----system---- ----total-cpu-usage---- -dsk/total- ---system-- -pkt/total- -net/total- ------memory-usage----- ---paging--
         time     |usr sys idl wai hiq siq| read  writ| int   csw |#recv #send| recv  send| used  buff  cach  free|  in   out 
    14-05 12:52:58|  0   0 100   0   0   0|   0     0 |  40    58 |1.00  1.00 |  60B  550B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:52:59|  0   0 100   0   0   0|   0     0 |  53    72 |1.00  1.00 |  60B  550B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:53:00|  0   0 100   0   0   0|   0     0 |  50    58 |1.00  1.00 |  60B  550B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:53:01|  0   0 100   0   0   0|   0     0 |  49    55 |1.00  1.00 |  60B  550B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:53:02|  0   0 100   0   0   0|   0     0 |  46    57 |1.00  1.00 |  60B  550B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:53:03|  0   0 100   0   0   0|   0     0 |  59    67 |1.00  1.00 |  60B  550B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:53:04|  0   0 100   0   0   0|   0     0 |  56    60 |1.00  1.00 |  60B  550B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:53:05|  0   0 100   0   0   0|   0     0 |  42    55 |1.00  1.00 |  60B  550B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:53:06|  0   0 100   0   0   0|   0     0 |  41    55 |1.00  1.00 |  60B  550B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:53:07|  0   0 100   0   0   0|   0     0 |  77    80 |1.00  1.00 |  60B  550B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:53:08|  0   0 100   0   0   0|   0     0 |  50    56 |1.00  1.00 |  60B  550B| 396M 2076k  435M 7172M|   0     0 
    14-05 12:53:09|  0   0 100   0   0   0|   0     0 | 194   169 |31.0  1.00 |1860B  550B| 396M 2076k  435M 7172M|   0     0 

其他详细参数请参考[man dstat]()

### free
