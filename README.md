# 性能分析一般步骤
应用程序的监控，可以分为指标监控和日志监控2大块
* 指标监控 主要是对一定时间段内的性能指标进行测量，然后再通过时间序列的方式，进行处理、存储和告警
* 日志监控 可以提供更详细的上下文信息，通常通过 ELK 技术栈，来进行收集、索引和图形化展示

## 系统资源瓶颈
 系统资源的瓶颈，可以通过 USE 法，即使用率、饱和度以及错误数这三类指标来衡量。系统的资源，可以分为硬件资源和软件资源两类
 * CPU,内存，磁盘文件系统，网络
 * 文件描述符，连接数，套接字缓冲区
### CPU性能分析
利用top，vmstat， pidstat， mpstat， strace， perf常用工具进行分析

<img src="https://github.com/lys861205/server_profile/blob/master/cpu_analysis.png" width="500" heigth="800">

工具top，pidstat，vmstat工具的CPU性能指标，都源自/proc文件系统（/proc/loadavg, /proc/stat, /proc/softirqs）
可以使用strace，查看进程的系统调用，使用perf工具，找出进程的热点函数，

### 内存性能分析
可以通过free, vmstat输出的性能指标，确认内存瓶颈，分析内存的使用，分配，泄露以及缓存

<img src="https://github.com/lys861205/server_profile/blob/master/memory_analysis.png" width="500" heigth="800">

内存性能指标,也来源于/proc文件系统(/proc/meminfo, /proc/slabinfo等）

### 磁盘和文件系统I/O性能分析
可以通过iostat 发现iO存在的性能瓶颈,IO使用率过高，响应时间过长或者等待队列长度突然增大等，
通过pidstat, vmstat等确认具体的I/O来源

<img src="https://github.com/lys861205/server_profile/blob/master/IO_analysis.png" width="500" heigth="800">

磁盘和文件系统的性能指标， 也来源于/proc 和 /sys 文件（/proc/diskstats, /sys/block/sda/stat等)

### 网络性能分析
网络性能分析要从linux的网络协议栈原理切入，通常包括应用层，套接字接口，传输层，网络层以及链路层

<img src="https://github.com/lys861205/server_profile/blob/master/net_analysis.png" width="500" heigth="800">

分析网络性能，要从几个协议层入手，通过使用率，饱和度，错误数性能指标，观察是否有问题
* 链路层 可以从网络接口的吞吐量、丢包、错误以及软中断和网络功能卸载等角度分析
* 网络层 可以从路由、分片、叠加网络等角度进行分析
* 传输层 可以从 TCP、UDP 的协议原理出发，从连接数、吞吐量、延迟、重传等角度进行分析
* 应用层 可以从应用层协议（如 HTTP 和 DNS）、请求数（QPS）、套接字缓存等角度进行分析

网络的性能指标也都来源内核，包括/proc 文件系统(/proc/net)

## 应用程序瓶颈
1. 依赖的服务瓶颈
诸如数据库，分布式缓存，中间件等应用程序，直接或者间接调用的服务出现了性能问题，从而导致应用程序的响应变慢，或者错误率升高
2. 程序自身的性能问题
多线程处理不当，死锁，业务算法的复杂度过高，观察关键环节的耗时和内部执行过程的耗时

还可以通过各类进程的分析工具，来进行分析定位
* 用strace， 观察系统的调用
* 使用perf和火焰图，分析热点函数

----
# 性能优化的一般步骤
