1. Process has three  states. In theory, with three states, there could be six transitions, two out of each state. However, only four transitions are shown. Are there any circumstances in which either or both of the missing transitions might occur?

2. Suppose that you were to design an advanced computer architecture that did process switching in hardware, instead of having interrupts. What information would the CPU need? Describe how the hardware process switching might work.
2. 假设你在设计一个先进计算机架构，在硬件完成进程切换，而不是使用中断完成。CPU需要哪些信息？描述硬件进程切换怎样工作？

硬件切换上下文。

3. On all current computers, at least part of the interrupt handlers are written in assembly language. Why?
3. 为什么中断处理用汇编写的？

进中断的第一件事就是把当前寄存器的内容推到栈上，以便于系统处理完中断回复。汇编更容易操作寄存器。

4. When an interrupt or a system call transfers control to the operating system, a kernel stack area separate from the stack of the interrupted process is generally used. Why?
4. 当中断或系统调用 转换控制权给os时，通常使用 与中断进程栈 分离的 内核堆栈区域。为什么？

（1）如果内核在系统调用结束时 在用户程序的内存中留下了数据，恶意用户可能会使用。
（2）如果操作系统控制中断的话可能会崩溃，因为写的不好的用户程序可能没有留下足够的栈空间。

5. A computer system has enough room to hold five programs in its main memory. These programs are idle waiting for I/O half the time. What fraction of the CPU time is wasted?
5. 一个计算机系统在其主存中有足够的空间容纳5个程序。这些程序在等待I/O的时间中有一半是空闲的。浪费了多少CPU时间？

0.5^5 = 3.125%

6. A computer has 4 GB of RAM of which the operating system occupies 512 MB. The processes are all 256 MB (for simplicity) and have the same characteristics. If the goal is 99% CPU utilization, what is the maximum I/O wait that can be tolerated?
6. 一个电脑有4GB 的RAM，操作系统占了 512MB。所有的进程都有 256MB（为了简单起见）且有相同特性。如果目标是99%的CPU利用率，那么可以容忍的最大I/O等待是多久？

（4GB - 512MB）/256MB = 14
最多14个进程，即1%开14次方，即 71.97%

7. Multiple jobs can run in parallel and finish faster than if they had run sequentially. Suppose that two jobs, each needing 20 minutes of CPU time, start simultaneously. How long will the last one take to complete if they run sequentially? How long if they run in parallel? Assume 50% I/O wait.
7. 如果多任务可以并行执行，且比顺序执行的速度快。假设有两个任务，同时开始的话每个需要CPU20分钟。如果按顺序运行，最后一个需要多长时间才能完成？如果它们并行运行多久？假设50%的I/O等待。

顺序执行：20min执行 + 20min等待，两个任务40min * 2 = 80min
并行执行：CPU利用率：1-0.5^2 = 75%，一个进程占用 75%/2 = 37.5%，两个进程并行时间：20min / 37.5% = 53.3min

8. Consider a multiprogrammed system with degree of 6 (i.e., six programs in memory at the same time). Assume that each process spends 40% of its time waiting for I/O. What will be the CPU utilization?
8. 假设一个多程序系统同时能跑6个程序。每个程序有40%的时间在等待I/O，那么CPU利用率是多少？

1 - 40%^6 = 0.995904

9. Assume that you are trying to download a large 2-GB file from the Internet. The file is available from a set of mirror servers, each of which can deliver a subset of the file’s bytes; assume that a given request specifies the starting and ending bytes of the file. Explain how you might use threads to improve the download time.
9. 假设您正试图从Internet下载一个2 GB的大文件。文件可以从一组镜像服务器获得，每个镜像服务器都可以提供文件字节的子集；假设给定的请求指定了文件的起始字节和结束字节。解释如何使用线程来缩短下载时间。

并行下载

10. In the text it was stated that the model of Fig. 2-11(a) was not suited to a file server using a cache in memory. Why not? Could each process have its own cache?
10. 文中指出，图2-11（a）的模型不适用于 使用内存缓存的文件服务器。为什么不呢？每个进程都可以有自己的缓存吗？

Fig. 2-11(a)的模型是 每个进程都只有一个线程

11. If a multithreaded process forks, a problem occurs if the child gets copies of all the parent’s threads. Suppose that one of the original threads was waiting for keyboard input. Now two threads are waiting for keyboard input, one in each process. Does this problem ever occur in single-threaded processes?
11. 如果一个有多线程的进程分叉了，而且子进程复制了父进程的所有线程，那么就会有问题了。假设一个原始线程在等待键盘输入，那现在就有两个线程在等待键盘输入，一个进程一个。这个问题在单线程进程会出现么？

不会，如果单线程进程在等待键盘输入的时候被阻塞，那它就不会fork

12. In Fig. 2-8, a multithreaded Web server is shown. If the only way to read from a file is the normal blocking read system call, do you think user-level threads or kernel-level threads are being used for the Web server? Why?
12. 在图 2-8 中显示了一个多线程Web服务器。如果从文件读取的唯一方法是正常的阻塞读取系统调用，那么您认为Web服务器使用的是用户级线程还是内核级线程？为什么？


13. In the text, we described a multithreaded Web server, showing why it is better than a single-threaded server and a finite-state machine server. Are there any circumstances in which a single-threaded server might be better? Give an example.
13. 在书中，我们描述了一个多线程Web服务器，说明了为什么它比单线程服务器和有限状态机服务器更好。在什么情况下，单线程服务器可能更好？举个例子。

CPU不太行的时候，单线程反而会增加复杂度。

14. In Fig. 2-12 the register set is listed as a per-thread rather than a per-process item. Why? After all, the machine has only one set of registers.
14. 寄存器被列在了线程项而不是进程项，为什么？每个计算机只有一个寄存器。

每个线程都要在寄存器存放自己的值。

15. Why would a thread ever voluntarily give up the CPU by calling thread_yield? After all, since there is no periodic clock interrupt, it may never get the CPU back.
15. 为什么线程会通过调用thread_yield自动放弃CPU？毕竟，由于没有周期性的时钟中断，它可能永远无法再得到CPU。

进程中的线程相互协作,有时候 yield 可以改进应用程序。
通常一个进程中的所有线程使用的同一个程序。
操作系统中没有类似的线程调度程序。因此，线程不能失控，永远占用进程的CPU时间。
为了防止资源死锁，线程可能需要让自己 yield，才能成功地继续进程。

16. Can a thread ever be preempted by a clock interrupt? If so, under what circumstances? If not, why not?
16. 线程可以被时钟中断抢占吗？如果可以，在什么情况下？如果不可以，为什么没有？

在整个进程的量被使用之前，不能抢占用户级线程。
可以抢占内核级线程，在内核级线程运行太久的时候可以抢占，中断执行之后，内核可以选择这个进程的任意线程执行。

17. In this problem you are to compare reading a file using a single-threaded file server and a multithreaded server. It takes 12 msec to get a request for work, dispatch it, and do the rest of the necessary processing, assuming that the data needed are in the block cache. If a disk operation is needed, as is the case one-third of the time, an additional 75 msec is required, during which time the thread sleeps. How many requests/sec can the server handle if it is single threaded? If it is multithreaded?
17. 这道题是比较单线程文件服务器和多线程服务器 读文件的区别。假设需要的数据都在缓存中，获取请求，分发，还有处理其他必要的操作需要12ms。磁盘操作需要再加75ms，此例是三分之一的时间在磁盘操作，磁盘操作期间线程睡眠。如果是单线程，服务器每秒可以处理多少个请求？如果是多线程呢？

12 * 2/3 + （12 + 75）* 1/3 = 37ms
1000 / 37 = 27 次

非抢占的调度器，多线程等待磁盘操作都是重叠的，所以总共12ms
1000 / 12 = 83 次

18. What is the biggest advantage of implementing threads in user space? What is the biggest disadvantage?
18. 在用户空间使用多线程，最大的优势和劣势分别是什么？

优势：提高效率
劣势：一个线程阻塞会导致整个进程阻塞

19. In Fig. 2-15 the thread creations and messages printed by the threads are interleaved at random. Is there a way to force the order to be strictly thread 1 created, thread 1 prints message, thread 1 exits, thread 2 created, thread 2 prints message, thread 2 exists, and so on? If so, how? If not, why not?
19. 在图2-15中，线程创建和线程打印的消息是随机交错的。有没有办法强制订单严格按照线程1创建、线程1打印消息、线程1退出、线程2创建、线程2打印消息、线程2存在等方式执行？如果是，怎么做？如果没有，为什么没有？

20. In the discussion on global variables in threads, we used a procedure create global to allocate storage for a pointer to the variable, rather than the variable itself. Is this essential, or could the procedures work with the values themselves just as well?
20. 在讨论线程中的全局变量时，我们使用了一个create global操作为指向该变量的指针分配存储，而不是变量本身。这是必须的吗，还是说程序与变量本身也能一起工作？】


21. Consider a system in which threads are implemented entirely in user space, with the run-time system getting a clock interrupt once a second. Suppose that a clock interrupt occurs while some thread is executing in the run-time system. What problem might occur? Can you suggest a way to solve it?
21. 假设一个系统其中线程完全在用户空间应用，系统运行时每秒获得一个时钟中断。如果某个线程在运行时突然来了一个时钟中断。可能会出现什么问题？有什么解决办法？

执行线程的程序在临界区附近，解锁或者上锁，这时候容易造成线程状态不一致。
解决方法就是，等执行完线程，延后执行中断程序。

22. Suppose that an operating system does not have anything like the select system call to see in advance if it is safe to read from a file, pipe, or device, but it does allow alarm clocks to be set that interrupt blocked system calls. Is it possible to implement a threads package in user space under these conditions? Discuss.
22. 假设操作系统没有类似 select 系统调用的功能，可以提前查看从文件、管道或设备读取数据是否安全，但它允许设置闹钟来中断被阻止的系统调用。在这些条件下，有可能在用户空间中实现线程包吗？讨论

23. Does the busy waiting solution using the turn variable (Fig. 2-23) work when the two processes are running on a shared-memory multiprocessor, that is, two CPUs sharing a common memory?
23. 当两个进程跑在一个共享内存的多进程处理器上时，即两个 CPU 共享一个公共区，使用turn 变量忙等是不是可行？

24. Does Peterson’s solution to the mutual-exclusion problem shown in Fig. 2-24 work when process scheduling is preemptive? How about when it is nonpreemptive?
24. Peterson 关于互斥的解决方案对于抢占式进程调度是否有效？对于非抢占式的呢？

‘’‘c
#define FALSE   0 
#define TRUE    1
#define N       2

int turn;
int interested[N];

void enter_region(int process);
{
    int other;
    other = 1 - process;
    interested[process] = TRUE;
    turn = process;
    while( turn == process && interested[other] == TRUE )
}

void leave_region(int process)
{
    interested[process] = FALSE;
}
’‘’
此解决方案就是为了抢占式设计的。对于非抢占式调度，一个进程会一直运行，永远不会释放 CPU。


25. Can the priority inversion problem discussed in Sec. 2.3.4 happen with user-level threads? Why or why not?
25. 优先级反转问题会出现在用户级线程吗？

不会，因为用户级线程没有抢占。内核级线程可能出现。

26. In Sec. 2.3.4, a situation with a high-priority process, H, and a low-priority process, L, was described, which led to H looping forever. Does the same problem occur if round-robin scheduling is used instead of priority scheduling? Discuss.
27. 在2.3.4节，描述了一个高优先级进程H和低优先级进程L的情况，这导致了H死循环。如果使用循环调度而不是优先级调度，是否会出现同样的问题？讨论

28. In a system with threads, is there one stack per thread or one stack per process when user-level threads are used? What about when kernel-level threads are used? Explain.
28. 在一个有线程的系统中，当使用用户级线程时，是每个线程有一个堆栈还是每个进程有一个堆栈？当使用内核级线程时呢？解释

每个线程都有自己的堆栈。

29. The producer-consumer problem can be extended to a system with multiple producers and consumers that write (or read) to (from) one shared buffer. Assume that each producer and consumer runs in its own thread. Will the solution presented in Fig. 2-28, using semaphores, work for this system?
29. 生产者-消费者问题可以延伸为 一个拥有多个消费者和生产者对一块共享内存的读写 的系统。假设每个生产者和消费者都跑在自己的线程中。使用信号量在此系统可以工作吗？

可以

30. Consider the following solution to the mutual-exclusion problem involving two processes P0 and P1. Assume that the variable turn is initialized to 0. Process P0’s code is presented below.

For process P1, replace 0 by 1 in above code. Determine if the solution meets all the required conditions for a correct mutual-exclusion solution.
30.   考虑两个进程P0和P1的互斥问题的下列解决方案。假设变量turn初始化为0。

P0:
/* Other code */
while (turn != 0) { } /* Do nothing and wait. */ 
Critical Section /* . . . */
turn = 0;
/* Other code */

P1:
/* Other code */
while (turn != 1) { } /* Do nothing and wait. */ 
Critical Section /* . . . */
turn = 1;
/* Other code */

如果turn只有0 1，只有 Process 0 和 Process 1 能对turn更改，那么此方案可以解决互斥问题。

31. How could an operating system that can disable interrupts implement semaphores?
31. 一个可以关中断的系统是怎么应用信号量的？

先关掉 能对信号量操作 的中断。然后读信号量，如果是 down 且信号量为 0，则它会将调用进程保留在包含与该信号量连接的所有阻塞进程的列表中。
如果信号量正在执行up操作，它必须检查信号量上是否有任何进程被阻塞。其中一个进程从包含被阻止进程的列表中删除，如果一个或多个进程被阻止，则使其可运行。

32. Show how counting semaphores (i.e., semaphores that can hold an arbitrary value) can be implemented using only binary semaphores and ordinary machine instructions.


33. If a system has only two processes, does it make sense to use a barrier to synchronize them? Why or why not?
34. Can two threads in the same process synchronize using a kernel semaphore if the threads are implemented by the kernel? What if they are implemented in user space? Assume that no threads in any other processes have access to the semaphore. Discuss your answers.
35. Synchronization within monitors uses condition variables and two special operations, wait and signal. A more general form of synchronization would be to have a single primitive, waituntil, that had an arbitrary Boolean predicate as parameter. Thus, one could say, for example,
waituntil x < 0 or y + z < n
The signal primitive would no longer be needed. This scheme is clearly more general than that of Hoare or Brinch Hansen, but it is not used. Why not? (Hint: Think about the implementation.)
36. A fast-food restaurant has four kinds of employees: (1) order takers, who take custom- ers’ orders; (2) cooks, who prepare the food; (3) packaging specialists, who stuff the food into bags; and (4) cashiers, who give the bags to customers and take their money. Each employee can be regarded as a communicating sequential process. What form of interprocess communication do they use? Relate this model to processes in UNIX.

37. Suppose that we have a message-passing system using mailboxes. When sending to a full mailbox or trying to receive from an empty one, a process does not block. Instead, it gets an error code back. The process responds to the error code by just trying again, over and over, until it succeeds. Does this scheme lead to race conditions?
