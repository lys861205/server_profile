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

<img src="https://github.com/lys861205/server_profile/blob/master/cpu_analysis.png" width="600" height="800">

工具top，pidstat，vmstat工具的CPU性能指标，都源自/proc文件系统（/proc/loadavg, /proc/stat, /proc/softirqs）
可以使用strace，查看进程的系统调用，使用perf工具，找出进程的热点函数，

### 内存性能分析
可以通过free, vmstat输出的性能指标，确认内存瓶颈，分析内存的使用，分配，泄露以及缓存

<img src="https://github.com/lys861205/server_profile/blob/master/memory_analysis.png" width="600" height="600">

内存性能指标,也来源于/proc文件系统(/proc/meminfo, /proc/slabinfo等）

### 磁盘和文件系统I/O性能分析
可以通过iostat 发现iO存在的性能瓶颈,IO使用率过高，响应时间过长或者等待队列长度突然增大等，
通过pidstat, vmstat等确认具体的I/O来源

<img src="https://github.com/lys861205/server_profile/blob/master/IO_analysis.png" width="600" height="400">

磁盘和文件系统的性能指标， 也来源于/proc 和 /sys 文件（/proc/diskstats, /sys/block/sda/stat等)

### 网络性能分析
网络性能分析要从linux的网络协议栈原理切入，通常包括应用层，套接字接口，传输层，网络层以及链路层

<img src="https://github.com/lys861205/server_profile/blob/master/net_analysis.png" width="600" height="400">

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
----
# 性能优化一般步骤
## 系统优化
USE 法可以用来分析系统软硬件资源的瓶颈，那么，相对应的优化方法，当然也是从这些资源瓶颈入手
### CPU优化
CPU 性能优化的核心，在于排除所有不必要的工作、充分利用 CPU 缓存并减少进程调度对性能的影响
* 第一种，把进程绑定一个或多个CPU上，充分利用CPU缓存本地性，减少进程间的影响
* 第二种，为多个处理程序启动多个CPU负载均衡，以便在发生大量中断时，可以充分利用多CPU的优势分摊负载
* 第三种，是用Cgroups等方法，为进程设置资源限制，避免个别进程消耗过多的CPU

### 内存优化
常见的一些内存问题，比如可用内存不足、内存泄漏、Swap 过多、缺页异常过多以及缓存过多
* 第一种，除非必要，Swap应该禁掉,避免swap的额外IO，带来访问内存变慢的问题
* 第二种，使用 Cgroups 等方法，为进程设置内存限制。这样就可以避免个别进程消耗过多内存，而影响了其他进程。
对于核心应用，还应该降低 oom_score，避免被 OOM 杀死
* 第三种，使用大页、内存池等方法，减少内存的动态分配，从而减少缺页异常

### 磁盘和文件系统I/O优化
* 第一种，优化文件系统和磁盘的缓存、缓冲区，比如优化脏页的刷新频率、脏页限额，
以及内核回收目录项缓存和索引节点缓存的倾向等等


### 网络优化
首先，从内核资源和网络协议的角度来说，我们可以对内核选项进行优化，比如
* 第一种，可以增大套接字缓冲区、连接跟踪表、最大半连接数、最大文件描述符数、本地端口范围等内核资源配额
* 第二种，也可以减少 TIMEOUT 超时时间、SYN+ACK 重传数、Keepalive 探测时间等异常处理参数
* 第三种，可以开启端口复用、反向地址校验，并调整 MTU 大小等降低内核的负担

## 应用程序优化
性能优化的最佳位置，还是应用程序，内部为什么这么说呢？我简单举两个例子你就明白了。
第一个例子，是系统 CPU 使用率（sys%）过高的问题。有时候出现问题，虽然表面现象是系统 CPU 使用率过高，
但待你分析过后，很可能会发现，应用程序的不合理系统调用才是罪魁祸首。这种情况下，
优化应用程序内部系统调用的逻辑，显然要比优化内核要简单也有用得多

再比如说，数据库的 CPU 使用率高、I/O 响应慢，也是最常见的一种性能问题。这种问题，一般来说，
并不是因为数据库本身性能不好，而是应用程序不合理的表结构或者 SQL 查询语句导致的。这时候，
优化应用程序中数据库表结构的逻辑或者 SQL 语句，显然要比优化数据库本身，能带来更大的收益

所以，在观察性能指标时，应该先查看程序的响应时间，吞吐量以及错误率等指标，
* 第一，从CPU使用角度来说，简化代码，优化算法，异步处理以及编译器优化等，
* 第二，从数据访问的角度来说，使用缓存，Copy on write, 增加IO大小，
* 第三，从内存管理角度来说，使用大页，内存池等方法，可以预先分配内存，减少内存的动态分配
* 第四，从网络角度来说，使用IO多路复用，连接池，长链接代替短链接
* 第五，从工程角度，使用异步处理，多线程，充分利用每个CPU的处理能力

----
----

# linux性能工具
Brendan Gregg整理的性能工具图谱

<img src="https://github.com/lys861205/server_profile/blob/master/linux_observability_tools.png" width="800" height="1000">

## CPU性能工具
cpu性能指标

<img src="https://github.com/lys861205/server_profile/blob/master/cpu_index.png" width="400" height="800">

cpu性能工具

<img src="https://github.com/lys861205/server_profile/blob/master/cpu_tool.png" width="400" height="800">

## 内存性能工具
内存性能指标

<img src="https://github.com/lys861205/server_profile/blob/master/mem_index.png" width="600" height="500">

内存性能工具

<img src="https://github.com/lys861205/server_profile/blob/master/mem_tool.png" width="500" height="600">

## 磁盘I/O性能工具
I/O性能指标

<img src="https://github.com/lys861205/server_profile/blob/master/io_index.png" width="600" height="500">

I/O性能工具

<img src="https://github.com/lys861205/server_profile/blob/master/io_tool.png" width="500" height="600">

## 网络性能工具
网络性能指标

<img src="https://github.com/lys861205/server_profile/blob/master/net_index.png" width="600" height="500">

网络性能工具

<img src="https://github.com/lys861205/server_profile/blob/master/net_tool.png" width="500" height="600">


## 基准测试工具

<img src="https://github.com/lys861205/server_profile/blob/master/linux_benchmarking_tools.png" width="800" height="1000">






