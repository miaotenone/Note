### SControlNetWork（公网服务器）
整体代码量比较小，其实只有五个文件。
![[远控项目文件.png]]

#### 整体结构图

#### main函数
解析：就是初始化网络然后执行。
![[远控公网服务器main函数.png]]

#### UDPPassNetWork类
##### 整体定义
先看声明，有两个部分--UDP和TCP，那么就有各自的地址、socket和连接用户哈希以及运行状态。

![[远控公网服务器SControlNetWork类.png]]

##### **构造函数**
构造函数就是初始化TCP和UDP地址，注意的是这个时候线程池已经初始化了，提前初始化。
![[远控公网服务器SControlNetWork类构造函数.png]]

##### **Invoke()函数**
TCP服务端和UDP初始化一条龙，参见[[socket编程]] 初始化流程，然后是线程池分配任务并激活。
![[远控公网服务器SControlNetWork类Invoke函数.png]]

##### **`int ThreadTcpProc()`函数**
接受连接并使用线程池分配任务执行处理程序
![[远控公网服务器SControlNetWork类TCP线程函数.png]]

##### **`int ThreadTcpClnt(void* arg)`函数**
整个函数就是不断读取1kb的数据然后解析的过程。
**注意：** 1、在这里看一下`ThreadTcpProc()`函数进行线程池分配函数，将int类型的`socket`转成`void*`类型，在x64系统中，指针大小是64位（8字节），到这个函数需要将`void*`类型转成int类型的`socket`，就需要先用long这个8字节的存储，再转换成4字节。
2、recv函数失败的处理过程是   关闭socket--> 用户哈希删除 --> 广播发送至剩余用户


![[远控公网服务器SControlNetWork类TCP客户端处理线程函数.png]]

##### **`int ThreadUdpProc()`函数**
UDP连接没有接受的过程，所以直接处理数据就可以。
![[远控公网服务器SControlNetWork类UDP线程函数.png]]

##### `int SendAddrs()`函数
遍历剩余用户哈希，发送102命令，表示
![[远控公网服务器SControlNetWork类SendAddrs函数.png]]