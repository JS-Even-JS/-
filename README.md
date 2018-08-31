# -How-is-the-network-connected
# 网络是怎么连接的
## 第一章 浏览器生成消息

## 第二章 用电信号传输TCP/IP 数据 --探索协议栈和网卡
### 2.1 创建套接字
* 2.1.1 协议栈的内部结构<br/>
  **协议栈**是指网络中各层协议的总和，本质是**操作系统内部的网络控制软件**，也叫**协议驱动**或者**TCP/IP驱动**，其形象的反映了一个网络中文件传输的过程：由上层协议到底层协议，再由底层协议到上层协议。<br/>
  协议栈内部主要分为两层：**TCP协议和UDP协议**作为一层，其下面还有**IP协议**作为另一层；<br/>
  整个系统分层结构为:<br/>
  **应用程序**<br/>
  **Socket库(DNS解析器、socket组件、connect组件、write组件、read组件、close组件)**<br/>
  **协议栈(上层的TCP协议和UDP协议，下层的IP协议(ICMP、ARP协议))**<br/>
  **网卡驱动程序(控制网卡)**<br/>
  **网卡(负责对网线中的信号执行发送和接收的操作)**<br/>
  **收发数据的时候，上层会将收发数据的工作委派给下层**<br/>
  **ICMP**:用于告知网络包传送过程中产生的错误以及各种控制消息；<br/>
  **ARP**:用于根据IP地址查询相应的以太网MAC地址；<br/>
  
* 2.1.2 套接字的实体
  在协议栈的内部有一块**用于存放控制消息的内存空间**，其中控制消息包括，**源IP地址、源端口、目的IP地址、目的端口、连接状态、是否已经收到了响应、发送数据后经过了多长时间**等；<br/>
  所以我们通常把这个**用于存放控制信息的内存空间称为套接字的实体，协议栈会根据这些控制信息进行下一步操作**，这也就是套接字的作用；<br/>
  套接字其实是可以通过命令行的方式来查看的，可以通过**netstat -an -p TCP**命令来查看，-a表示**不仅显示正在通信状态(ESTABLISHED状态)的套接字，还显示处于尚未开始通信状态(LISTENING状态)的套接字**，-n表示**显示IP地址和端口号**<br/>
  ![](https://github.com/JS-Even-JS/-How-is-the-network-connected/blob/master/images/socket.png)
  图中的每一行就是一个socket套接字<br/>
  tcp46       0      0  *.8080                 *.*                    LISTEN     <br/>
  ip地址为*号*表示远程ip地址和本地ip地址都未确定，说明通信还没有开始，处于监听状态，在等待客户端的连接，并且是在8080端口上等待客户端连接；<br/>
  tcp4       0      0  127.0.0.1.8080         127.0.0.1.65213        ESTABLISHED <br/>
  这个是**服务器端**的socket <br/>
  tcp4       0      0  127.0.0.1.65213        127.0.0.1.8080         ESTABLISHED <br/>
  这个是**客户端**的socket <br/>
  因为一个socket是由客户端ip和端口、服务器端ip和端口共同组成，所以这也可以解释**为什么多个客户端可以连接到同一台服务器上的同一个端口上了**<br/>
  
* 2.1.3 调用socket组件时的创建操作
  应用程序和协议栈直接隔着一个Socket库，所以**应用程序必须通过Socket库来向协议栈发出一系列的委托请求**<br/>
  ①**创建套接字阶段**,应用程序首先会调用Socket库中的**socket组件**申请创建套接字，然后**协议栈根据应用程序的申请执行套接字的创建操作**,协议栈会向**内存管理模块**提出申请，请求划分一块内存空间出来，用于**存放创建的套接字**，其实这个给套接字的空间就是**用于给套接字存放控制信息的**，套接字创建好之后，**会给套接字写入一些初始的状态控制信息**；<br/>
  创建好的套接字会**存放于协议栈中**，确切的说是存放在了**协议栈中的TCP模块中**，因为只有TCP协议才会使用套接字进行通信；<br/>
  socket组件会返回创建好的socket的**描述符**，即 **<描述符> = socket(<使用IPv4>,<使用TCP协议>)**,返回的socket描述符会传递给应用程序，应用程序委托协议栈发送数据的时候**需要提供这个socket描述符**<br/>
  服务器程序启动的时候也会创建套接字，只不过这个套接字是处于LISTEN监听状态，在等待客户端的连接；<br/>
  
### 2.2 连接服务器
* 2.2.1 调用connect组件的连接操作<br/>
  所谓**连接**，就是**通信的双方互相交换控制信息**。套接字刚创建好之后，里面并没有存放任何数据，也不知道通信的对象是谁，但是浏览器从输入的url地址中可以指定通信对象的ip地址和端口号，所以**需要把服务器的ip地址和端口号告知协议栈**，应用程序委托协议栈发送数据的时候，协议栈才知道该给谁发送数据。在进行连接的过程中，由于**需要进行收发数据的操作，所以也会向内存管理模块申请一片内存空间，即所谓的缓冲区，用来临时存放要收发的数据**<br/>

* 2.2.2 负责保存控制信息的头部<br/>
  
