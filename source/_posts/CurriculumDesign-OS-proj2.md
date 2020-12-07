---
title: 操作系统课程设计-nachos（java版） Project2:Multiprogramming
date: 2020-01-04 09:37:34
categories: 
- 课程设计
tags:
- 操作系统
- java
---
操作系统课程设计Project2多道程序设计系统设计与实现
<!-- more -->
## 项目地址
https://github.com/ccclll777/nachos_os_design
**如果觉得有用，请点个star**

## nachos用户程序初探
nachos的用户程序需要使用mips交叉编译，虽然伯克利的管网给出了很多操作系统版本的mips编译器，但是经过一天的挣扎，最后还是在64为的centos上，对用户程序成功进行了编译。

编译时，需要修改makefile，将自己新加的程序加入makefile，然后在test文件夹下执行make，即可成功编译用户程序。

nachos提供了coff文件的加载类，nachos将coff文件加载之后，会将其分成若干个coffsection的段，然后对每一个段进行loadpage的操作，将其加载到物理内存中。而coff类和coffsection就是做用户程序加载工作的。

而nachos用户程序调用内核的系统调用，是通过test文件中的start.s进行链接的，此文件包含用于初始化进程的汇编语言代码。它还提供了允许调用系统调用的系统调用“stub code”。这利用了特殊的mips指令syscall，它捕获到nachos内核来调用系统调用。

对象文件与libnachos.a链接以生成与Nachos兼容的MIPS二进制文件，其扩展名为*.coff。

## Project2:Multiprogramming多道程序设计系统设计与实现
**【1】总体描述**
Nachos的第二阶段是支持多道程序设计。和第一个作业一样，我们给你一些你需要的代码；你的工作是完成系统并增强它。到目前为止，您为nachos编写的所有代码都是操作系统内核的一部分。在实际的操作系统中，内核不仅在内部使用其过程，而且允许用户级程序通过系统调用访问其一些例程。 
首先需要理解，Processor类模拟mips处理器，SerialConsole类模拟控制台，FileSystem是一个文件系统接口。要访问文件，请使用machine.stubfilesystem（）返回的文件系统。此文件系统访问测试目录中的文件。UserKernel类一个多道程序设计的内核，UserProcess类用户进程；管理空间地址，并将程序加载到虚拟内存中。UThread类能够执行用户mips代码的线程。SynchConsole类一个同步控制台；使得在多个线程之间共享机器的串行控制台成为可能。

**【2】task1:实现文件系统调用**
**【2.1】要求分析**
提供文件系统的七个系统调用，halt(停机), creat(创建并打开磁盘文件),open(打开磁盘文件),read(读取文件), write(写入文件), close(关闭), unlink(删除磁盘文件)。 我们需要为每一个用户程序提供一个进程号，用来区别他是不同的进程。
要确保如下几点: 
1)稳定性,不能因为一个进程的非法系统调用就使操作系统崩溃,而应该返回错误代码。 
2)halt()调用只能由第一个进程(root process)执行。 
3)系统调用需要读写内存时,通过 readVirturalMemory和writeVirtualMemory进行。 
4)文件名以null结尾,不超过256字符。 
5)如果系统调用出错,应返回-1。 
6)为每个打开的I0文件分配一个“文件描述符”,用整数表示每个进程最多可以拥有16个。其中0和1应分配给标准输入和标准输出(即屏幕),这由 
SynchConsole类管理。不同进程可以用相同的文件描述符处理不同的文件。 
7)Nachos已经提供了一个简单的文件系统 FileSystem(Machine包中),通过 ThreadedKernel. fileSystem访问 
8)系统不需要考虑文件访问的互斥等问题。 

**【2.2】实现方式**
（1）关闭文件halt（）在关闭之前先判断这个进程的进程号，看他是不是root进程，如果是则可以调用halt（）。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427094630455.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（2）首先定义文件描述符的类，属性为文件名filename以及操作文件系统的nachos的Openfile类。每个用户程序都会最多拥有16个文件描述符，然后根据要求将标准输入和输出分配对应的文件描述符。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427094642283.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427094647270.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（3）create（）系统调用，nachos在载入一个用户进程执行时，会把程序载入nachos的内存，在打开或者新建文件时  会返回一个文件描述符  读写文件要使用文件描述符来指定读写的文件若进程已打开相同的文件描述符，则不创建返回该文件的文件描述符。

创建一个文件时，首先根据传入的虚拟内存中的文件地址，读入虚拟内存中存储的文件名。
然后根据文件名，使用nachos的文件系统打开文件（没有则会创建对应的文件），之后为文件分配对应的文件描述符，并将文件描述符和打开的文件存在对应的数据结构中，最后返回文件描述符。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427094719239.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（4）open（）系统调用
首先根据传入的虚拟内存中的文件地址，读入虚拟内存中存储的文件名。
然后根据文件名，使用nachos的文件系统打开文件，之后为文件分配对应的文件描述符，并将文件描述符和打开的文件存在对应的数据结构中，最后返回文件描述符。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427094735126.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（5）read（）系统调用
read系统调用需要读取nachos文件系统的信息，然后将其写入用户程序的虚拟内存（主存中），传入的参数是，打开文件的文件描述符，内存的地址以及大小。首先在文件描述符数组中，用文件描述符获取到对应的nachos文件系统的对象，然后调用文件系统的read方法读取相应的内容，通过writeVirturalMemory（）写入主存中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042709475137.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（6）close（）系统调用，将之前用户程序创建的文件描述符移除，需要传入对应的文件描述符，找到对应的nachos文件系统对象，现将文件通过OpenFile关闭，然后将此文件描述符从文件描述符列表中移除。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427094801668.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（7）unlink系统调用
需传入对应的文件描述符，然后从内存中读取文件名，然后通过文件名找到对应的文件描述符，通过OpenFile类的remove操纵将此文件从磁盘删除，最后清除此文件描述符。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427094810270.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**【2.3】测试及其结果**
我们测试的文件的创建，打开，写入，读出，关闭，并且测试了如果创建超过16个文件描述符，是否可以停止分配，返回-1.
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095040793.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**【3】task2:完成多道程序设计的支持**
**【3.1】要求分析**
完成对多道程序的支持。已给出的代码只能一次运行一个进程,而你要使它 
支持多道程序。 从分配物理内存入手,不同的进程在在内存使用上不会重叠。用户程序不会使用 malloc()或free(),既用户没有动态内存分配的需求。故在装入程序时就要为每个进程一次性分配固定的物理内存,在进程结 束时收回它们。还需要实现简单的虚拟内存方案:每个进程的地址空间是连续的虚拟内存(以页面为单位,1页=1024字节),但这些连续的虚拟页面在物理内存中却不一定是连续的。 这个方案很简单,虚拟空间的总容量和物理空间的总容量相等,映射机制只是从虚拟内存到物理内存的一一映射。
**【3.2】实现方式**
首先我们需要使用页表，每一个页表项都是TranslationEntry类的对象，我们还需要一个全局的存放空闲页表的队列freePages，并且实现对物理页的分配和释放。在分配物理页时，为了避免同一页被分配给两个用户程序，所以需要加锁来保证分配页时的原子性。
在加载用户程序时，coff文件分为若干的段，每一段都是一个coffsection类型的对象，每一个coffsection又包括若干个页。当加载用户程序时，需要给他从空闲页中分配若干页，然后加载到页表中。
**（1）对空闲页表的分配和释放getFreePage（）addFreePage（）**

空闲页标记的队列使用链表来实现，在加载用户程序是，会使用getFreePage（）来分配页表项，在释放时会使用addFreePage（）来回收页表项。并且使用信号量来保证分配页表时的原子性。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095106369.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（2）loadSection（）加载用户程序**
在用户程序启动时，使用load将用户程序从磁盘装入内存，然后装入coffSection，在这时为每一个虚拟页分配一个物理页，在得到可用的物理页号之后，就装入所有的页，为段中每一个虚拟页在pageTable中创建一个TranslationEntry的对象，并为其添加信息（虚拟页号，物理页号等）然后调用coffsection的loadpage方法将每一页的内容读入。读入时，进行了物理地址和逻辑地址的关联，将程序的每一块按照顺序对应于物理地址导入到主存当中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095143806.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095146379.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（3）unloadSection（）在用户程序执行结束释放资源时，需要情况pageTable的对应页，然后释放对应freePages中的物理页。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095155987.png)
（4）readVirturalMemory（）在读取虚拟内存时，需要根据地址提取出虚拟页号以及页偏移，然后从页表pageTable中找到对应的物理页，然后将对应的内容复制到数组中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095301594.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（5）writeVirturalMemory（）写入虚拟内存时，也是需要将虚拟地址转化为虚拟页，然后寻找出对应的物理页，根据是否是只读文件，判断是否能写入。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095309309.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**【4】task3:实现系统调用**
**【4.1】要求分析**
实现调度用户程序的三个系统调用exec，join和exit。
exec为启动一个新进程；join与内核线程之间的join操作类似，exit将立即结束当前进程。
子进程的状态是完全私有的，父子进程之间不直接共享存储器或文件描符。注意两个过程当然可以打开相同的文件;例如,所有的进程应该有的0和1映射到系统控制台的文件描述符。 
不像 KThread.join(),只有一个进程的父进程可以调用join()。举例来说, 
如果A创建B并且B创建C,A不允许join()到C中,但B被允许join（）到C 中。
加入需要一个进程ID作为参数,用于唯一标识其父进程希望加入的子进程。 
进程ID应该是全局唯一的正整数,在每个进程创建时分配进程ID。解决这个问题的最简单的方法就是保持一个表明接下来的进程ID分配的静态计数器。
当进程调用exit(),它的线程应立即终止,而这个过程应该清理与它相关的任何状态(也就是释放内存,关闭打开的文件等)。如果一个进程异常退出执行相同的清理。
**【4.2】实现方式**
在进程初始化时，需要为其分配一个pid，分配pid时需要使用信号量保证他能原子的执行。由于我们要实现子进程的exec，join等操作，所以我们需要保存子进程的pid，以便在之后调度时使用。
**（1）exec（）系统调用**
参数为依次为文件名地址，参数个数，参数列表地址。
我们需要做的是：在虚拟内存中读出文件名；处理参数，用参数列表地址作为虚拟内存地址得到参数列表数组的首地址，然后使用readVirturalMemoryString依次读出每一个参数；之后使用newUserProcess创建子进程，将文件和参数列表加载到子进程；最后使用execute方法执行子进程，同时将这个子进程的父进程置为这个进程。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095354441.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（2）join系统调用**
参数依次为子进程编号，保存子进程状态的地址。
需要做的工作是：利用进程号确定join的是那个进程，检查childProcesses，遍历子进程列表，确定join的进程是子进程。如果子进程编号不存在，则出错；之后就讲父进程休眠，子进程执行结束，唤醒父进程，加入就绪队列等待调度。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095411265.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（3）exit（）系统调用**
1.首先关闭coff  将所有的打开文件关闭  将退出状态置入，
2.如果该进程有父进程   看是否执行了join方法 如果执行了就将其唤醒  同时将此进程从子进程链表中删除
3.调用unloadsections释放内存，调用kthread。finish结束线程
4.如果是最后一个线程 则停机
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095454308.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**【4.3】测试及其结果**
先执行sh.coff，之后在nachos的控制台上执行其他的用户程序，sh.coff里面执行了父进程与子进程之间的调度关系。并且可以测试task2的支持多道程序设计的实现也是没有问题的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042709550671.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**【5】task4:实现彩票程序调度**
**【5.1】要求分析**
实现彩票调度这个类扩展自PriorityScheduler类，只是做了一点修改：（1）“优先级”的概念变成了“彩票”（表示该线程下次被允许的几率）
（2）在调度过程中，并不是选择彩票数最多的进程执行，而是随机抽取一张彩票，让彩票的主人运行，这样，彩票越多，下次执行的机会就越大。
**【5.2】实现方式**
getEffectivePriority中将选取最大优先级的过程改成将所有等待锁的线程的彩票加到拥有锁的线程上，然后修改nextThread：遍历一遍队列，计算出当前队列中所有进程的彩票总数（考虑过彩票的捐赠），然后生成一个随机数x代表抽取的彩票，最后在遍历一遍队列，用累加器t计算彩票的总数，当t的值大于x的值时，说明当前循环到的线程正好落在目标区间内，然后选出该线程。
![在这里插入图片描述](https://img-blog.csdnimg.cn/202004270955328.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095536131.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**【5.3]测试以及结果**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095549906.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427095557285.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
新建了5个线程，分别给他们10，15，20，25，30张彩票，然后将他们以不同的顺序放到队列中，等待调度，然后计算不同进程的优先级。