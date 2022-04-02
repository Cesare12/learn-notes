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