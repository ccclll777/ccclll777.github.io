---
title: 操作系统课程设计-nachos（java版） Project3:Caching and Virtual Memory
date: 2020-01-04 13:37:34
categories: 
- 课程设计
tags:
- 操作系统
- java
---
操作系统课程设计Project3（虚拟存储）系统设计与实现
<!-- more -->
## 项目地址
https://github.com/ccclll777/nachos_os_design
**如果觉得有用，请点个star**

## 有关Linux操作系统交换文件，交换分区
Linux将随机存储RAM称为内存页。交换技术就是将一页内存复制到预先设定的硬盘上的交换空间，来释放该页占用内存。物理内存和交换空间的和就是可提供的虚拟内存的总量。 
有两个原因证明交换技术是很重要的。首先，系统需要的内存量比物理内存更大时，系统内核可以把较少使用的内存页写到交换空间，把空闲出来的内存给当前的应用程序（进程）使用。其次，一个应用启动时使用的内存页，可能只是在初始化时使用，之后不会再用，操作系统就可以把这部分内存页写入交换空间，把空闲出来的内存给其他应用使用或作为磁盘高速缓存。 
Linux支持将一块硬盘或者某块硬盘的某块区域设定为交换分区，在缺少物理内存时，可以进行交换，比直接换入硬盘的普通区域效率要更高一些。

## Project3:Caching and Virtual Memory（虚拟存储）系统设计与实现
**【1】总体描述**
   NACHOS的第三个阶段是研究TLB、虚拟内存系统和文件系统之间的相互作用。为了帮助更好地组织代码，已经提供了一个新的package，nachos.vm，带有两个新类， VMKernel and VMProcess. 。VMKernel继承了UserKernel, VMProcess 继承UserProcess。VMKernel and VMProcess是您必须为这个项目阶段修改的唯一类。
   
设计者需要设计如何在TLB未命中时进行软件地址转换、如何表示交换分区、如何实现分页等。您应该根据所有可用的标准来评估您的设计：处理TLB缺失的速度、内存中的空间开销、最小化页面错误的数量、简单性等。

首先要考虑的设计方面是软件管理（TLB），在第二阶段使用页表来简化内存分配，并将故障从一个地址空间中隔离出来，以免影响其他程序，
在这个阶段，处理器对页表一无所知。相反，处理器只处理页面表条目的软件管理缓存，称为TLB。

给定一个内存地址（要获取的指令，或要加载或存储的数据），处理器首先在TLB中查找，以确定虚拟页到物理页的映射是否存在。如果映射在TLB中，处理器将直接使用它。如果映射不在TLB中（“TLB未命中”），处理器会导致操作系统陷入内核。然后，内核负责使用页表、段、反向页表或其他合适的机制将映射加载到TLB中。

这个项目的第二个设计方面是分页，它允许物理内存页被传送到磁盘，并提供磁盘（几乎）无限物理内存的幻象。TLB未命中可能需要从磁盘中导入一个页面以满足转换。也就是说，当发生TLB未命中错误时，内核应该检查自己的页表。如果页面不在内存中，则应该从磁盘中读取页面，将页面表条目设置为指向新页面，安装页面表条目，并恢复用户程序的执行。当然，内核必须首先在内存中找到输入页面的空间，如果修改了，可能会将其他页面写入磁盘。

这种机制的性能很大程度上取决于用来决定哪些页保存在内存中并存储在磁盘上的策略。在页面错误时，内核必须决定要替换哪个页面；理想情况下，它会扔出一个不会长期引用的页面，记住这些页面可能很快被引用。另一个注意事项是，如果替换的页面已被修改，则必须先将该页面保存到磁盘，然后才能将所需的页面引入。（当然，如果页面未被修改，则无需将其写回磁盘。

为了帮助您实现虚拟内存，每个TLB条目包含三个状态位： valid, used,和dirty. 如果设置了有效位（valid为true），则虚拟页在内存中，而TLB可以直接由处理器使用。 如果valid 为flase  或者如果在TLB中找不到页面，则处理器陷入内核，进行页面置换 。每当页面被引用时，处理器就在TLB条目中设置所使用的位，并在页面被修改时设置脏位。
每当页面被引用时，处理器就在TLB条目中设置use为true（used在读写时，首次加载时，表示此页正在使用，会将used设置为true
），并在页面被修改时设置脏位（dirty）。
**【2】task1:实现TLB的软件管理，通过反向页表进行页面置换**

**【2.1】要求分析**

我们需要实现一个全局的反向页表，在每个用户程序加载时，会在反向页表中分配一项，里面记录进程的逻辑页，但是不分配物理页（valid为false），物理页的分配在出现页错误时在进行。

  在出现页错误时，会陷入内核，在给对应的逻辑页分配物理页，然后将此页（TranslationEntry）的信息加入TLB中，程序可以继续执行。
需要在上下文切换时正确的设置TLB的状态，需要在保存上下文时保存此用户程序当前TLB的状态，然后在恢复上下文时恢复之前TLB的状态（或者只恢复valid为false的页）

**【2.2】实现方式**
（1）需要设计一个反向页表类（InvertPageTable），里面有一个全局的反向页表（GlobalInvertPageTable）（java的静态变量），已经对于反向页表进行操作的一些静态方法。
反向页表使用HashMap实现，用进程的id和虚拟页号为key，然后页表项TranslationEntry为value。
需要维护一个全局的帧表，为每一页如果分配了物理内存，则会加入帧表。
需要实现对反向页表的插入，删除，更新，设置，获取对应的页的方法，并且在缺页时，通过二次机会算法选择需要牺牲的页。
在用户程序加载时，先为每一个虚拟页在反向页表中分配空间，在发生页错误后会陷入内核，分配相应的物理页，然后更新反向页表以及帧表的内容。

**（2）反向页表的插入（insertEntry）**
在用户程序加载时，会在反向页表中插入页，如果此页的valid标记为true，表示此页应该在物理帧中，则在帧表中分配一块空间。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100028752.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（3）反向页表的删除（deleteEntry）**
在用户程序执行完毕释放资源时，需要调用此方法释放反向页表中有关的条目。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100040596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（4）设置反向页表的状态（setEntry）**
在某一页发生更新时，需要在反向页表中更新相应的状态
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100307480.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（5）获取某进程某页对应的页表项（getEntry）**![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042710031733.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（6）二级机会算法，来选择牺牲页。**
遍历物理页表，寻找使用位为false的页，如果第一次遍历到use标记为true，则更新标记为false（给他第二次机会），最后返回一个标记为false的页。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100333933.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**（7）用户程序在执行过程中，有关反向页表的操作**
在用户程序加载时，会为程序在反向页表中分配对应的页。
读取虚拟内存时，需要判断此页是否以及分配物理页，以及更新反向页表中有关此页的状态
写入虚拟内存时需要更新反向页表中某一页的状态（dirty和used）
交换文件时，需要在反向页表中回去对应页的状态来进行不同的操作。
在TLB未命中时，需要获取页表状态，看是否需要分配物理内存。
**（8）用户程序在执行过程中，有关TLB的操作**
在保存上下文时，需要保存此时TLB的状态，在恢复上下文时，可以恢复此时TLB的状态。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100351213.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100357312.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
在发生TLB未命中时，需要选择一个TLB然后装入需要的页。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100406524.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
**【3】task2:实现虚拟内存的按需调页**
**【3.1】要求分析**
使用nachos的filesystem作为交换空间，将页从磁盘移动到内存，从内存移动到磁盘。
为了找到未引用的页面以排除页面上的错误，您需要跟踪系统中当前正在使用的所有页面。
反向页表只能包含实际位于物理内存中的页的条目。您需要维护一个单独的数据结构来定位交换中的页面。
页面替换策略不应将任何未修改的页面写入交换文件（即未设置脏位的页面）。因此，即使页面已被移动到物理内存中，也需要将其保持在交换状态。
现在页面正在内存和磁盘之间来回移动，您需要确保一个进程不会在另一个进程对页面执行其他操作
我们建议您使用由所有进程共享的单个全局交换文件。您可以为此文件使用您希望的任何格式，但只要跟踪文件中存储不同虚拟页的位置，就应该非常简单。您可以假设将交换文件扩展到任意大小是安全的；也就是说，您不必担心此文件的磁盘空间不足。

**【3.2】思考**
有两个原因证明交换技术是很重要的。首先，系统需要的内存量比物理内存更大时，系统内核可以把较少使用的内存页写到交换空间，把空闲出来的内存给当前的应用程序（进程）使用。其次，一个应用启动时使用的内存页，可能只是在初始化时使用，之后不会再用，操作系统就可以把这部分内存页写入交换空间，把空闲出来的内存给其他应用使用或作为磁盘高速缓存。 

**【3.3】实现方式**
需要维护一个全局的交换文件类（SwapperController）来维护交换空间，这个交换空间用nachos的文件系统实现。
然后维护三个数据结构，表示已经分配交换空间的项，还未分配交换空间的项，已经交换空间剩余的位置。
交换文件使用的是nachos的文件类openfile，而nachos的文件类依赖于java的文件类。所有交换文件是磁盘上的一块区域（假设这块区域是无限大的）
需要实现各种操作，如在交换空间分配位置，写入交换文件，读取交换文件，删除交换文件中相应的位置，根据进程的虚拟页寻找物理页在交换文件中存在的位置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100437658.png)
（1）用户程序加载时，需要为其的虚拟页在未分配列表分配对应的位置，表示此页存在但是还没有分配，以便在之后为其在交换空间分配相应的位置。（insertUnallocatedPage）
![在这里插入图片描述](https://img-blog.csdnimg.cn/202004271004467.png)
（2）为用户程序的某一页在交换空间分配位置（allocatePosition）
先查看未分配列表中有没有此页，没有则为其分配一个位置，有则从已分配列表寻找对应的位置继续使用。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100457460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（3）交换空间的读写，都是通过偏移量决定读写的位置，根据进程的pid和虚拟页号，在已分配交换空间的列表中寻找对应的位置号position，表示此页在交换空间所占的位置。当需要读写时，读写的起始位置都是（位置号position）*（对应每一页的大小pagesize），然后就是读取或写入的位置，而读取或者写入的大小，则是一页的大小pagesize。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100508212.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（4）用户程序使用交换空间的相关操作
用户程序开始执行时，需要将其有关的页加入交换空间的为分配列表
当发生缺页时（缺少物理页），则选择一个不常使用的页，将其在交换空间中分配位置，写入交换空间。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042710052087.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
当发生TLB错误时，可能需要用到之前存入交换空间的页，然后找到对应的页在交换到物理内存中继续使用。
当用户程序执行结束后，需要释放交换空间中为此用户程序分配的位置。
测试结果：
交换文件，在test文件夹下，是一个二进制文件：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042710053364.png)
**【4】task3:实现从用户程序延迟加载代码和数据页**
**【4.1】需求分析**
不是在启动时从用户程序中读取所有的代码和数据，而是在反向页表中设置页表条目，这样当出现页面错误时，从可执行文件中的页面将按需读取到内存中。
必须确保对可执行文件的内存映像的更改不会写入可执行文件。代码页总是只读的，因此在执行过程中，用户进程可以不修改它们。然而，来自可执行文件（代表全局变量等）的数据页可以由该过程修改。确保这些更改不会写回可执行文件。
除了从可执行文件中延迟加载页面外，还应根据需要分配堆栈页面。因此，当初始化时，不应该将完整的堆栈页分配给进程，而是在堆栈页上出现页错误时，应在那时分配新的堆栈页。仔细考虑堆栈页应该如何分配和初始化，特别是涉及到的安全问题
**【4.2】实现方式**
（1）为了延迟加载用户程序的数据和代码，我们需要在loadSection时不真正加载用户程序，而是将加载用户程序所需要的地址（coffSectionAddress）和对应的虚拟页号（virtualPageNum）存起来。
当程序发生缺页时，如果未加载的列表包含此虚拟页号，则再为它分配相应的物理页，然后加载对应的coffsection，实现用户程序代码和数据的懒加载。
（2）在load时分配反向页表的条目，交换空间未分配列表
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100551298.png)
（3）在loadsection时现将每一个coffsection对应的逻辑页和coff文件的地址存起来，等待加载。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100559658.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（4）当发生页错误时，再去分配物理页，将对应的coffsection加载到物理页中，实现懒加载
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200427100606937.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
