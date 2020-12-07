---
title: 操作系统课程设计-nachos（java版） Project1:Build a thread system
date: 2020-01-03 23:52:34
categories: 
- 课程设计
tags:
- 操作系统
- java
---
操作系统课程设计Project1，实现内核线程操作
<!-- more -->


## 项目地址
https://github.com/ccclll777/nachos_os_design
**如果觉得有用，请点个star**
# nachos初探
刚拿到nachos之后，都不知道从哪里开始下手，只能从目录结构开始，慢慢熟悉整个nachos。
**1.Nachos.ag包**
有关内核的加载启动类
**2.Nachos.machine包**
nachos操作系统的主要包
里面有用户程序的加载类coff，coffsection；
**3.配置文件的加载类config**
**4.有关操作系统中断模拟类interrupt**
**5.内核的抽象类kernel；**
**6.全局的工具类lib**
里面定义一些操作系统内部可用的断言，随机数的生成，文件的加载，字符类型的转化，有关类的检查（检查访问权限），有关类的实例的创建等等。
**7.Machine类**
机器启动类，里面定义了全局可以使用的每个类的实例，文件的加载，参数的加载，机器线程的启动等等。
**8.Openfile类，StubFileSystem类**
提供nachos的文件系统的模拟，底层使用java有关文件系统的类实现
**9.Stats类**
有关nachos运行状态的管理。
**10.Timer类**
有关nachos时钟的模拟类
**11.Processor类**
让nachos操作系统可以运行mips交叉编译后用户程序的类，通过对加载到内存的coffsection进行加载，解码，执行等等。然后还管理了nachos操作系统的内存memory，以及TLb。还初始化了nachos使用的内存空间，寄存器空间以及堆栈空间。
**12.TCB类**
有关nachos线程的管理，nachos中有许多线程，而每一个nachos线程都会又一个TCB来管理线程的状态，而nachos的线程又对应了一个java的线程，来使用java虚拟机运行nachos。TCB类中，实现了线程的启动，线程的上下文切换等操作。
还有一些有关nachos控制台的实现，通过多线程的控制，实现控制台的串行输入，输出。
**13.network包**
为了实验四准备，实现网络系统的包
security包：有关nachos操作系统的权限管理，在加载操作系统时，给内核的每一个类，加载了自己对应的权限，在越权之后，会产生报错，操作系统停止。nachos的securitymanager通过继承和重载java的security类实现。
**14.test包：**
里面是有关的用户程序，需要将c交叉编译为coff文件
**15.threads包：**
有关java的线程实现以及实验一使用的类。
**16.Alarm类：**
有关nachos操作系统中线程的抢占以及休眠
**17.KThread类：**
有关nachos的线程，每一个nachos的线程都与java的线程以及一个TCB相对应。nachos线程有几个对应的状态：从线程的new，ready，running，waitting，blocked。然后实现了线程的调度，
**系统的运行：**
nachos操作系统运行在java的虚拟机上，在执行时，首先加载配置文件，根据配置文件的内容选择加载的内核类，以及一些需要使用的类。然后加载安全配置，测试文件等等。
nachos很充分的体现了面向对象的思想，各个模块之间有序的连接，叹为观止！
## Project1:Build a thread system 内核线程操作的实现
**【1】总体描述**
nachos提供了不是很完善的线程系统，需要我们阅读代码之后，会nachos操作系统的运作有了一定程度的了解，然后进行下一步的工作。
**【2】task1:实现kthread.join（）**
**【2.1】要求分析**
如果主线程调用了a.JOIN 则主线程会被阻塞  a线程将执行 直到a线程返回  主线程才会继续执行。在一个KThread对象上只能调用一次join，且当前线程不能对自身调用join。
**【2.2】实现方式**
为每一个线程创建join队列，当a线程运行过程中调用b.join，则a线程被挂起，加入b线程的join队列，然后等待b线程执行完毕后，释放a线程，a线程才能在被调度然后执行。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426234458721.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

需要在线程finish时，将自身join队列上的线程释放，加入等待队列等待调度。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042623451243.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

**【2.3】测试以及结果**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426234524443.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426234533684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)


说明：在主线程上调用了t1.join，在t1线程上调用了t2.join，所以有如下的运行结果。

****【3】task2:使用开关中断提供原子性，实现条件变量****
**【3.1】要求分析**
已经提供了一个使用信号量的示例实现；需要提供一个等效的实现，而不直接使用信号量。（可以使用锁，虽然它间接的使用了信号量）
条件变量使我们可以睡眠等待某种条件出现,是利用线程间共享的全 局变量进行同步的一种机制,主要包括两个动作:一个线程等待”条件变量的条件成立"而挂起;另一个线程使”条件成立”(给出条件成立信号)为了防止竞争, 条件变量的使用总是和一个互斥锁结合在一起。 
**【3.2】实现方式**
使用锁threads.lock以保证互斥。在临界代码区的两端执行 Lock. acquire()和Lock. release()即可保证同时只有一个线程访问临界代码区。条件变量建立在锁之上,由 threads. Condition实现,用来保证同步。每一个条件变量拥有一个锁变量。 
将调用sleep的线程加入到等待队列，然后释放锁并阻塞，以便其他线程唤醒它，,需要在函数的开始和结尾处关或允许中断保证原子性。当前线程必须拥有相关锁,阻塞进程在 sleep(函数返回之前线程将会自动再次拥有锁。即调用这个方法的线程将自己挂起到等待队列,阻塞自己的同时将自己拥有的锁交出去,之后等待锁的分配。       
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042623463844.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
  
wake()唤醒条件变量队列中第一个线程,在试验中利用从线程队列中取出线程用 KThread. ready()实现唤醒,加入就绪队列。(同样要在wake函数利用 中断保证原子性) 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426234644774.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
    wakeall()唤醒在此条件变量上睡眠的所有线程。当前线程必须持有关联的锁
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042623465283.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)


**【3.3】测试以及结果**
条件变量在之后的实现中会使用到，所以没有单独写测试代码


**【4】task3:实现alarm类的waituntil（long x）方法**
**【4.1】要求分析**
线程调用waituntil（long x）来暂停自己的执行，直到时间至少提前到现在+x为止。线程唤醒之后，并不一定立马进入运行状态，只要将其放入就绪队列即可。
**【4.2】实现方案**
先构造一个队列，用来存放调用waituntil的线程，需要存储对应的线程以及wait的时间

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426234743493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

waitUntil()方法使用了一个队列可以存放线程以及唤醒时间,这个队列以时间为序的有序队列。每次调用时,把当前线程和唤醒时间加入队列,等 待唤醒。在调用 waitUnt il(x)函数时,首先得到关于该线程的信息(线程:当 
前线程,唤醒时间:当前时间+x),该线程设定唤醒时间并阻塞,并调用 sleep操作使当前线程挂起,放入队列中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426234758126.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

在时钟回调函数中(大约每500个时钟间隔调用一次)有一次中断,中断时则依次检查队列中的每个对象,将队列中此时应 当唤醒的进程加入就绪队列。如果唤醒时间大于当前时间,则将该对象移出队列并执行wake操作将相应线程唤醒。 
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042623480593.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

【4.3】测试代码及测试结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426234815253.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426234824760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)


说明：启动了20个线程分别wait随机的一段时间，然后会发现线程唤醒的时间一定在它能被唤醒的要求时间之后。
**【5】task4:用条件变量实现消息的收取speak和发送listen**
**【5.1】要求分析**
使用条件变量（不要使用信号量）实现单字长消息的同步发送和接收使用操作void speak（int word）和int listen（）实现Communicator类。
speak（）原子性地等待，直到在同一通信器对象上调用listen（），然后将消息传递给listen（）。一但消息发送完成，双方都可以返回。类似地，listen（）会一直等到调用speak（）时才进行传输，然后两者都可以返回。即使同一个通信器有多个speaker和listener，您的解决方案也应该有效（注意：这相当于零长度有界缓冲区；由于缓冲区没有空间，生产者和消费者必须直接交互，要求他们彼此等待）

**【5.2】实现方式**
对于speaker：需要先获得锁，然后进行判断，如果没有listener等待，则会把speaker挂起，如果有听者等待，则唤醒一个listeneer，然后传递消息最后释放锁。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020042623485163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

对于listener：要先获得锁，然后尝试唤醒speaker，如果没有speaker等待，则把listener放入队列然后睡眠。如果有speaker等待，就要唤醒一个speaker，将自己挂起等待speaker准备好数据将自己唤醒，然后传递消息，最后释放锁。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426234900534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

**【5.3】测试代码及结果**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426234907556.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426234922348.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)


说明：每个listener收到消息后，下一个speaker才会发出消息，两者一一对应。
【6】通过完成priorityscheduler类在nachos中实现优先级调度
【6.1】要求分析
优先级调度的传统算法如下:每个线程拥有一个优先级(在 Nach中,优先级是一个0到7之间的整数,默认为1)。在线程调度时,调度程序选择一个拥有最高优先级的处于就绪状态的线程运行。这种算法的问题是可能出现“饥饿”现象。为解决上述优先级反转问题,需要实现一种“让出”优先级的机制(Priority Donation)。 get EffectivePriority(就是为了解决优先级翻转而设的,也就是说当出现一个优先级高的线程等待低优先级线程时,若存在另外的高优先级线程而导致低优先级线程永远不能执行的情况,该低优先级线程调用该函数返回它持有的锁上等待的所有线程中优先级中最高的一个给该优先级线程,以确保它执行。 
【6.2】实现方式
使用一个优先级队列PriorityQueue和保存进程优先级，调整进程优先级的ThreadState类共同实现进程的优先级调度。
对于每个KThread都会保存等待它持有资源的队列，以及他自己所处的等待队列。
对于每个优先级队列PriorityQueue，都会保存等待此队列拥有锁的其他进程，我们用一个大小为8的TreeSet数组保存，每个线程根据优先级放入相应的TreeSet中，这样，每一个TreeSet中的进程都是按照相应的规则排序的。这样在优先级的捐赠时，实现将会更加的简单。
getEffectivePriority()：获取进程的有效优先级时，需要检查等待它资源的优先队列上所有的进程，找到其中优先级最大的那一个，然后获取他的优先级，然后重新计算此线程正在等待的线程，修改自己所处等待队列中自己的优先级。
waitForAccess（）：在某个进程等待某个优先级队列上的资源时，需要将它加入此优先级队列的等待资源的数据结构中。
Acquire（PriorityQueue waitQueue）：表示某个进程已经获得waitQueue持有的锁，需要将waitQueue加入  此进程的 等待资源的队列中。
在PriorityQueue的方法中，需要修改的是pickNextThread()和nextThread()。其中pickNextThread()方法是从link链表里拿到一个ThreadState对象。nextThread()方法是比较从link链表里拿到的ThreadState的有效优先级，找到有效优先级最大的的线程的ThreadState，并返回该ThreadState的thread对象，同时将该ThreadState从link链表里删除。

```java
public class PriorityScheduler extends Scheduler {
  
    public PriorityScheduler() {
    }

    //构造⼀一个优先级队列列 如果transferPriority为true则表示可以传输优先级
    public ThreadQueue newThreadQueue(boolean transferPriority) {
        return new PriorityQueue(transferPriority);
    }

    public int getPriority(KThread thread) {
        Lib.assertTrue(Machine.interrupt().disabled());

        return getThreadState(thread).getPriority();
    }

    public int getEffectivePriority(KThread thread) {
        Lib.assertTrue(Machine.interrupt().disabled());

        return getThreadState(thread).getEffectivePriority();
    }

    public void setPriority(KThread thread, int priority) {
        Lib.assertTrue(Machine.interrupt().disabled());

        Lib.assertTrue(priority >= priorityMinimum &&
                priority <= priorityMaximum);

        getThreadState(thread).setPriority(priority);
    }

    public boolean increasePriority() {
        boolean intStatus = Machine.interrupt().disable();

        KThread thread = KThread.currentThread();

        int priority = getPriority(thread);
        if (priority == priorityMaximum)
            return false;

        setPriority(thread, priority + 1);

        Machine.interrupt().restore(intStatus);
        return true;
    }

    public boolean decreasePriority() {
        boolean intStatus = Machine.interrupt().disable();

        KThread thread = KThread.currentThread();

        int priority = getPriority(thread);
        if (priority == priorityMinimum)
            return false;

        setPriority(thread, priority - 1);

        Machine.interrupt().restore(intStatus);
        return true;
    }

    public static final int priorityDefault = 1;
 
    public static final int priorityMinimum = 0;
  
    public static final int priorityMaximum = 7;


    // 返回线程的调度状态
    protected ThreadState getThreadState(KThread thread) {
        if (thread.schedulingState == null)
            thread.schedulingState = new ThreadState(thread);

        return (ThreadState) thread.schedulingState;
    }

 
    //按优先级对线程排序的线程队列列。
    protected class PriorityQueue extends ThreadQueue {
        PriorityQueue(boolean transferPriority) {
            cnt = 0;
            this.transferPriority = transferPriority;
            wait = new TreeSet[priorityMaximum + 1];
            for (int i = 0; i <= priorityMaximum; i++)
                wait[i] = new TreeSet<ThreadState>();

        }

        //将线程加入  等待队列中
        public void waitForAccess(KThread thread) {
            Lib.assertTrue(Machine.interrupt().disabled());
            //表示此线程 在等待  此队列上的锁
            getThreadState(thread).waitForAccess(this);
        }


        //表示此线程  已经获得锁 可以开始执行
        public void acquire(KThread thread) {
            Lib.assertTrue(Machine.interrupt().disabled());
            getThreadState(thread).acquire(this);
            if (transferPriority)
                lockholder = getThreadState(thread);
        }


        public KThread nextThread() {
            Lib.assertTrue(Machine.interrupt().disabled());
            ThreadState res = pickNextThread();

            return res == null ? null : res.thread;
        }


        //返回下一个<tt>nextThread（）</tt>将返回的线程，而不修改此队列的状态。
        protected ThreadState pickNextThread() {
            //取出⼀一个优先级最    高的线程;
            ThreadState res = NextThread();

            if (lockholder != null) {
                //将此队列 移除  刚才 拥有此队列锁的线程  的WaitResourceQueues  因为要更换 一个新的拥有锁的线程
                lockholder.holdQueues.remove(this);
                lockholder.getEffectivePriority();
                //此队列现在拥有锁的线程 应该是当前执行的线程
                lockholder = res;
            }
            if (res != null) res.waitQueue = null;
            return res;
        }


        //执行队列上下一个 优先级最高的线程
        protected ThreadState NextThread() {
            ThreadState res = null;

            for (int i = priorityMaximum; i >= priorityMinimum; i--)
                if ((res = wait[i].pollFirst()) != null) break;

            return res;
        }


        public void print() {
            Lib.assertTrue(Machine.interrupt().disabled());
            // implement me (if you want)
        }

 
        //添加 等待(此线程拥有资源) 的线程
        public void add(ThreadState thread) {
            wait[thread.effectivePriority].add(thread);
        }

        public boolean isEmpty() { //
            for (int i = 0; i <= priorityMaximum; i++)
                if (!wait[i].isEmpty()) return false;
            return true;
        }

        //<tt>true</tt>如果此队列列应将优先级从等待线程传输到所属线程
        protected long cnt = 0;
        public boolean transferPriority;

        protected TreeSet<ThreadState>[] wait; //等待此队列锁的其他线程
        protected ThreadState lockholder = null;//此队列中 拥有锁的线程

    }

    protected class ThreadState implements Comparable<ThreadState> {

        public ThreadState(KThread thread) {
            this.thread = thread;
            holdQueues = new LinkedList<PriorityQueue>();
            setPriority(priorityDefault);
            getEffectivePriority();//获取有效优先级
        }

        public int getPriority() {
            return priority;
        }
        public int getEffectivePriority() {
            int res = priority;
            //遍历等待它 资源的 线程队列 （这些线程可以让出自己的优先级）
            if (!holdQueues.isEmpty()) {
                Iterator it = holdQueues.iterator();
                while (it.hasNext()) {
                    PriorityQueue holdQueue = (PriorityQueue) it.next();
                    //如果等待它拥有的资源的线程中 有优先级⽐比较⼤大的线程
                    for (int i = priorityMaximum; i > res; i--)
                        if (!holdQueue.wait[i].isEmpty()) {
                            res = i;
                            break;
                        }
                }
            }
            //重新计算此线程正在等待的线程 修改自己所处等待队列中 自己的优先级
            if (waitQueue != null && res != effectivePriority) {
                ((PriorityQueue) waitQueue).wait[effectivePriority].remove(this);
                ((PriorityQueue) waitQueue).wait[res].add(this);
            }
            effectivePriority = res;
            if (lockholder != null)
                //让拥有锁的线程  获取自己的有效优先级 看能不能提前执行
                lockholder.getEffectivePriority();
            return res;
        }



        public void setPriority(int priority) {
            if (this.priority == priority)
                return;

            this.priority = priority;
            // implement me
            this.getEffectivePriority();

        }
        //在指定的优先级队列列上调⽤用waitforaccess(thread)</tt>时调⽤用(其中<tt>thread</tt>是关联的线 程)。
        // 因此，关联的线程正在等待对由waitqueue保护的资源的访问。仅当关联的线程⽆无法⽴立即获得访问权限时才调⽤用此⽅方法。
        //此线程队列列指定的线程正在等待调⽤用 将需要等待获得资源de 线程j加⼊入等待队列列等待调度
        public void waitForAccess(PriorityQueue waitQueue) {
            Lib.assertTrue(Machine.interrupt().disabled());
            //waitQueue拥有资源 等待调⽤ 优先级可能不够高
            time = ++waitQueue.cnt;

            this.waitQueue = waitQueue;
            //将此线程 加入 等待队列的   等待资源的线程的 数据结构中
            waitQueue.add(this);
            //此线程需要的资源  被等待队列中那个 拥有锁的线程 拿着
            lockholder = waitQueue.lockholder;
            //获取自己的有效优先权
            getEffectivePriority();//进⾏优先权的交换
        }

        /**
         * Called when the associated thread has acquired access to whatever is
         * guarded by <tt>waitQueue</tt>. This can occur either as a result of
         * <tt>acquire(thread)</tt> being invoked on <tt>waitQueue</tt> (where
         * <tt>thread</tt> is the associated thread), or as a result of
         * <tt>nextThread()</tt> being invoked on <tt>waitQueue</tt>.
         *
         * @see ThreadQueue#acquire
         * @see ThreadQueue#nextThread
         */
        //当前线程已经获得到waitQueue上的锁
        public void acquire(PriorityQueue waitQueue) {
            Lib.assertTrue(Machine.interrupt().disabled());

            //所以将waitQueue加入  此线程的 等待资源的队列中
            if (waitQueue.transferPriority) holdQueues.add(waitQueue);
            Lib.assertTrue(waitQueue.isEmpty());
        }


        /**
         * The thread with which this object is associated.
         */
        //关联线程
        protected KThread thread;
     
        //关联线程的优先级
        protected int priority;

        //有效优先级
        protected int effectivePriority;

        protected Long time;
        ;//等待时间
        protected ThreadQueue waitQueue = null; //表示 所处的等待队列
        protected LinkedList holdQueues;// 表示 ⼦子孙线程 等待它拥有资源的线程队列  等待队列
        protected ThreadState lockholder = null; //该线程等待队列中 拥有锁的线程

        //实现Comparator接⼝口，并重写compare()⽅方法， @Override
        public int compareTo(ThreadState target) {
            if (time == target.time) return 0;
            return time > target.time ? 1 : -1;
        }

    }
}
```

【6.3】测试代码及结果

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426235037167.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)


说明：一共15个线程，1-5不需要锁，6-10需要a锁，11-15需要b锁，每个进程都有着不同的优先级，可以看出有明显的优先级捐赠的痕迹
【7】task6:用条件变量实现过河问题
【7.1】要求分析
利用前面所学同步方法,以及条件变量解决过河问题。一群夏威夷人想要从瓦胡岛(Oahu)到摩洛凯岛(Molokai),只有一艘船,该船可载最多2个小孩或一个成人,且必须有一个开船的人(默认每个人都会开船)。设计一个解决方案,使得每个人都能从Oahu到 Molokai。假设至少有两个小孩,每一个人都会划船。要为每个大人和孩子创建一个线程,这个线程就相当于一个人,他们能自己判断(思考),什么时候可以过河,而不是通过调度程序的调度。在解决问题的过程中不能出现忙等待,而且问题最终一定要解决。不符合逻辑的事情不能发生,比如在这个岛的人能知道对面岛的情况。
【7.2】实现方式
对于孩子：由于要让孩子划船，所以我们应该经可能保证现将孩子运送到M岛，来保证永远有孩子能将船从M岛划到O岛。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426235053270.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

如果船在O岛，则尽可能让两个孩子都上船，让两个孩子一起到M岛，并且在到达M岛后睡眠;如果此时O岛上没有其他孩子则放弃此次对船的占有,尝试唤醒O岛上的大人。 
如果船在M岛，则让孩子把船开到O岛，然后唤醒O岛的所有大人。
如果孩子线程和船都在O岛,且船上已经有一个孩子作为乘客,则该线程作为乘客获取船驶向M岛,并且做问题解决检查,如果O岛上在自己离开后没有人了的话则问题解决, 
如果孩子线程和船都在O岛,但船上已经满载,则放弃船。

对于大人：只需在O岛sleep 等待唤醒  如果能上船则上船不能上船则sleep。到达M岛后 唤醒一个孩子  让孩子划船岛O岛。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426235142760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

```java
static void ChildItinerary() {
    bg.initializeChild();

    int position = Oahu;
    while (true) {
        conditionLock.acquire();
        if (position == Oahu) {
            while ((boatPosition != Oahu) || (peopleOnBoat > 1)) {
                OahuChildCondition.sleep();
            }
            if (peopleOnBoat == 0)//第一个上船的小孩
            {
                peopleOnBoat += 1;
                OahuChildCondition.wakeAll();//唤醒O岛的小孩
                numOfChildrenOnOahu -= 1;
                OahuChildCondition.wakeAll();
                boatCondition.sleep();//如果在O岛 之上一个孩子 那船先不能开
                position = Molokai;//一个孩子到了M岛
                numOfChildrenOnMolokai += 1;
                MolokaiChildCondition.sleep();
            } else//第二个小孩上船
            {
                boatCondition.wake();
                numOfChildrenOnOahu -= 1;
                bg.ChildRowToMolokai();//一个小孩驾船
                bg.ChildRideToMolokai();//一个小孩坐船
                OahuChildCondition.wakeAll();
                boatPosition = Molokai;
                peopleOnBoat = 0;
                position = Molokai;//两个孩子都到了M岛
                numOfChildrenOnMolokai++;
                if (numOfChildrenOnOahu == 0 && numOfAdultsOnOahu == 0) {
                    s1.V();
                    MolokaiChildCondition.sleep();
                } else {
                    numOfChildrenOnMolokai--;
                    bg.ChildRowToOahu();
                    position = Oahu;
                    boatPosition = Oahu;
                    numOfChildrenOnOahu += 1;
                    OahuAdultCondition.wake();//叫醒一个O岛的大人
                    OahuChildCondition.sleep();
                }
            }
        } else//如果船在M岛  小孩怎么做
        {

            //小孩要驾驶船去M岛
            numOfChildrenOnOahu += 1;
            numOfChildrenOnMolokai -= 1;
            position = Oahu;
            bg.ChildRowToOahu();
            boatPosition = Oahu;
            //然后将O岛的所有小孩唤醒
            OahuChildCondition.wakeAll();
        }
        conditionLock.release();
    }

}

```

【7.3】测试代码及实验结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200426235147582.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

说明：可以看到一直是先运送小孩，在运送大人的，直到所有人都到达O岛。

