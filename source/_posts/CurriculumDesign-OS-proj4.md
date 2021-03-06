---
title: 操作系统课程设计-nachos（java版） Project4:Networks and Distributed Systems
date: 2020-01-05 15:37:34
categories: 
- 课程设计
tags:
- 操作系统
- java
---
操作系统课程设计Project3（网络和分布式系统）系统设计与实现
<!-- more -->
## 项目地址
https://github.com/ccclll777/nachos_os_design
**如果觉得有用，请点个star**

## 初探与网络系统有关的类

有关网络上数据传输的类为nachos.machine.Networklink和nachos.network.postoffice,在两者的配合下，进行数据包的发送与接收。
**【1】Networklink类**
同一个机器上运行的不同nachos实例可以使用networklink类通过网络相互通信。
其中有多个线程进行线程之间的同步，来进行网络消息的传输。
在接收数据包时：
如果已经收到消息时，会会进行wait（），直到允许接收数据包时（某个变量为null），会进行notify（）。
而底层数据包的收发是经过java的DatagramSocket类进行的，将接收到的消息通过DatagramPacket类拷贝到内存中的某个地方。
在发送数据包时：
会构造一个DatagramPacket的对象，然后使用DatagramSocket进行数据包的发送。
数据包的发送和接受是通过多线程，以及中断驱动的，并且在发送消息和接收消息的过程中，通过synchronized保证临界区的互斥。在通过DatagramSocket收到消息后，某个线程得到了自己需要的数据，就开始构造数据包，然后将其存到某个变量中，当postoffice接收消息时，会将这个变量的内容置为空，然后启用某个线程，继续接收数据包。
在发送数据时，如果有从postoffice传来的数据，会存到某个变量中，然后如果这个变量不为空，某个线程就会将数据通过DatagramSocket发送出去，然后初始化变量为null，等待下一个消息。
**【2】PostOffice类**
PostOffice类使用Networklink类中的方法进行消息的收发。
它给每一个端口都准备一个消息队列，在收到消息时，会将添加到对应端口的队列中，然后可以唤醒等待读取消息的进程。
在接收消息时，会指定端口，获取到它队列中的消息，如果队列此时为空，线程将会休眠，直到接收到消息。（经过修改，在接收消息时，如果对应的消息队列为空，不会将线程阻塞，而是返回空，等待下一次遍历）
当一个包被发送后，会使用信号量，将线程停止，直到此数据包被接收或者被丢弃。
当接收消息时，如果没有消息，则会使用信号量将线程停止，直到数据包到达，在让线程继续执行。

之后会在此基础上构建网络系统的传输工作
## Project4:Networks and Distributed Systems（网络和分布式系统）系统设计与实现
**【1】总体描述**
我们将为您提供一些低级的网络通信设施；您将在这些设施之上构建一个更好的抽象，然后将该抽象用于构建分布式聊天程序。 
网络中的每个节点都将作为nachos的实例来实现。因此，您将为每个网络节点运行一个jvm；这些jvm将在同一台实际机器上运行，即使我们假装它们分布在网络上。 

每个nachos“节点”都有一个到网络的连接，该连接是使用UDP socket实现的。 nachos.machine.networklink类为您提供此功能。 

machine/NetworkLink.java  物理网络硬件的仿真，网络接口与控制台类似，只是传输单元是一个包而不是字符。网络在节点之间提供有序、不可靠的有限大小数据包传输。此类提供send（）和receive（）消息以在网络上发送和接收数据包。由于此类模拟网络设备驱动程序，因此必须提供保护对网络链接的访问并确保其正确使用的机制。网络链路在接收数据包和传输数据包后生成中断；您将使用这些中断在低级网络链路接口上实现高级通信机制。

machine/Packet.java 一个网络传输单元  一个包的最大大小由 machine.Packet.maxPacketLength给出，它恰好是32字节。 每个包包括4byte的头文件，每个包都包含一个4字节的头（详细信息请参见packet.java）和一个28字节的payload。使用常量machine.packet.maxcontentslength来表示代码中有效payload的大小（即，不要使用值为28字节的硬代码）。 

post office提供了比原始网络链接更方便的通信抽象。 它在网络链路的节点到节点通信之上提供用户到用户的通信，但不提供可靠的消息传递。在这个项目中，您必须在 PostOffice和网络链接抽象之上实现其他层。
**【2】task1:实现两个网络系统调用accept（）和connect（）**
**【2.1】要求分析**
它们在连接端点之间提供可靠的、面向连接的字节流通信。连接端点是与端口号组合在一起的链接地址。用户程序通过使用远程网络链接ID（也称为主机ID）和端口号调用connect（）来连接到远程节点，远程节点必须调用accept（）系统调用来接受连接；系统调用采用可进行连接的本地端口号。

在sockets上实现read（）和write（）必须提供无消息大小限制的全双工、可靠的字节流通信。这意味着数据包可以同时在连接上双向流动。
需要使用滑动窗口协议。

因为连接是全双工的，所以每个通信方向都需要一个单独的滑动窗口。此外，每个连接都应保持自己的滑动窗口。这意味着，如果到同一端口有多个连接，则一次可以在每个连接（每个方向）上发送多达16个未确认的数据包。

连接上的每个单向数据流都需要发送方的一个不同的发送缓冲区和接收方的一个不同的接收缓冲区。每个发送缓冲区可以任意增大，但每个接收缓冲区可以容纳不超过16个包。

读/写系统调用不应等待来自远程主机的数据包。如果端口上没有可读取的数据，则read立即返回。写入将端口上的数据排队，然后返回而不等待确认。（但是，在建立连接之前，连接将被阻塞。）
nachos传输协议规范：

一个端点首先调用connect（）系统调用。此端点称为活动端点。另一个端点（被动式端点）上的进程稍后必须调用accept（）系统调用。当调用这些系统调用时，底层协议执行双向握手。当用户进程调用connect（）系统调用时，活动端点发送同步数据包（syn）。这将导致创建挂起的连接。在被动端点的用户进程调用accept（）系统调用之前，此挂起连接的状态必须存储在接收器上。调用accept（）时，被动端向主动端发送一个syn确认数据包（syn/ack），并建立连接。 

一旦建立了连接，双方就可以发送数据包。每个包都有一个唯一的32位序列号。在接收到数据packet时，接收器将确认packet（ack）发回给发送者。与tcp一样，nachos传输协议使用滑动窗口。但是，与tcp不同的是，窗口的大小总是固定在16个数据包。发送方有责任确保其发送的未确认数据包不超过16个。为了保证最终接收到所有数据包，发送方必须定期重新传输已发送但尚未确认的任何数据包。重传过程的频率应为每20000个时钟周期一次。 

**【2.2】实现方式**
由于我们要实现网络协议，所以我们需要研究数据包的构成，构造一个packet类，构造对应的包的内容，nachos的传输协议给出了数据包的格式。

我们还需要一个特殊的文件类，这个文件类记录了发送方和接受方的地址信息，以便我们在网络传输中，可以根据不同的发送方来定位不同的文件，实现传输工作，并且在读写时，不是简单的内存操作，需要使用对应的方法接受指定端口的数据，或者向对应的端口发送数据。
然后需要实现端口的分发，为每一个连接分发不同的端口。而消息的接受和发送，我们只需要将消息传到发送消息的线程就能执行消息的发送，接收消息时从消息队列接收消息即可。

**（1）实现固定的数据包格式Packet类**
nachos已经提供了基本的数据包类型，我们需要根据nachos的传输协议规范构造相对应的数据包。
我们需要记录每个数据包的源地址和目标地址，并且由于要实现滑动窗口协议，我们需要给每一个数据包一个序列号，以及不同的状态。我们需要根据传输协议定义不同的状态位。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427101446992.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（2）connection（）实现网络所使用的文件系统，传输和接受对应的网络文件。**
我们需要继承Openfile类，然后重载文件的read和write操作。
read操作需要获取某个指定端口的数据，获取数据时，如果对应的端口没有数据传回，则会阻塞（修改之后变为，会返回空，然后客户端对标记进行处理）。在接收到数据之后，会将数据复制到目标数组，由于是滑动窗口协议，所以还要更新序列号。
write操作需要读取内存中的某一段数据，然后将其构造成数据包，发送到对应的目的地
注：底层数据包的接受和发送依赖于java的DatagramSocket类。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427101503727.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427101507723.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（3）系统调用connect（）和accept（）**
根据nachos操作系统的传输协议实现，底层协议需要执行双向握手。
当用户程序A要主动连接用户程序B时，需要调用connect的系统调用，指定B的ip地址和端口，内核应该给A分配对应的文件描述符，文件类型为connection类型（为网络创建的文件类型），然后构造对应的数据包（状态为syn），发送给用户程序B，用户程序在accept（）到指定端口的syn数据包时，会将此包的端口信息添加，然后分配一个文件描述符，文件类型也为connection类，然后确实对方是否要连接自己，如果是正确的连接，则B会构造一个synack的数据包，发送给A作为回应，A在收到synack的回复后，会向B发送确认收到的ack数据包，之后建立连接，返回文件描述符socket，B在收到ack确认后，也会返回对应的文件描述符，表示连接已经建立。到此，A与B正式建立连接，可以相互发送数据包。
（在使用postoffice发送和接受数据包时，通过信号量控制，数据包的收发，当收到数据包时，执行P，则继续执行networkLink的接收。当发送数据包时，直到此数据包被丢弃或者接受才会发送下一个数据包）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427101524445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427101528197.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（4）有关出错重传的设计**
在packet的发送类中，定义一个数据结构，存储未收到确认的消息，在接收消息时，会根据收到数据包的序列号，以及状态码，判断是否数据包，还是确认收到数据的数据包。如果是确认收到数据的数据包，则会将刚刚发送的数据，从未收到消息确认的队列中删除，如果是其他需要进行确认的数据包，则向来源发送确认收到数据包的消息。
会在packet的发送类中，单独开启一个线程，将未收到确认的数据包重新发送。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427101541101.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427101544195.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
而在消息包的收发时，需要根据数据包的序列号，以及状态判断它是数据包，或者确认收到数据的数据包，如果是确认收到数据的数据包，则将那数据从为发送列表中删除，如果是其他数据包，则发送确认收到数据包的消息。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427101555562.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**【3】task2:实现网络聊天应用程序**
**【3.1】要求分析**
通过使用消息传递的系统调用在多个（至少3个）用户之间实现“网络聊天”应用程序，演示您的消息传递系统调用的工作。

你应该编写两个程序：一个聊天服务器（chat server.c）和一个聊天客户端（chat.c）。服务器在一个节点上运行，接受来自聊天客户端的连接。聊天客户端连接到聊天服务器并接受来自控制台的用户输入。用户键入一行文本后，该行将传输到聊天服务器。然后，服务器将消息广播给与之连接的所有客户端。然后，聊天客户端接收到的每条消息都应显示在控制台上。用户可以动态连接和断开与聊天室的连接。
**【3.2】实现方式**
对于chatserver，需要循环进行accept，看是否有客户端请求建立连接，如果有连接，则建立连接，并且保存连接的标记。
每次循环都会判读是否收到数据，如果收到的数据，会将收到的数据进行广播。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427101615163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
客户端程序chat.c，先请求服务器连接，然后返回连接的socket，之后不断的读取控制台的输出，如果读到了，就将其发送给服务端，然后进行广播。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427101628219.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
