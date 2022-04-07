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
32. 演示如何仅使用二进制信号量和普通机器指令来实现计数信号量（即，可以容纳任意值的信号量）。

33.  If a system has only two processes, does it make sense to use a barrier to synchronize them? Why or why not?
33. 如果一个系统只有两个进程，那么使用屏障来同步它们有意义吗？为什么？
没用

34. Can two threads in the same process synchronize using a kernel semaphore if the threads are implemented by the kernel? What if they are implemented in user space? Assume that no threads in any other processes have access to the semaphore. Discuss your answers.
34. 如果线程是由内核实现的，那么同一进程中的两个线程可以使用内核信号量进行同步吗？如果它们是在用户空间中实现的呢？假设任何其他进程中的线程都无法访问该信号量。讨论你的答案。

35. Synchronization within monitors uses condition variables and two special operations, wait and signal. A more general form of synchronization would be to have a single primitive, waituntil, that had an arbitrary Boolean predicate as parameter. Thus, one could say, for example,
waituntil x < 0 or y + z < n
The signal primitive would no longer be needed. This scheme is clearly more general than that of Hoare or Brinch Hansen, but it is not used. Why not? (Hint: Think about the implementation.)

It is very costly and difficult to implement. Every time for any variable which becomes visible in a predicate on which some other process is waiting for a change, then the run-time system has to evaluate the predicate again to look if it is possible to unblock the process.
With the help of Hoare and Brinch Hansen monitors, it is possible to awaken the process only on a signal primitive.

36. A fast-food restaurant has four kinds of employees: (1) order takers, who take custom- ers’ orders; (2) cooks, who prepare the food; (3) packaging specialists, who stuff the food into bags; and (4) cashiers, who give the bags to customers and take their money. Each employee can be regarded as a communicating sequential process. What form of interprocess communication do they use? Relate this model to processes in UNIX.
36. 快餐店有四种员工：（1）点菜员，他们接受顾客的点菜；（2） 厨师，准备食物；（3） 包装专家，他们把食物塞进袋子里；（4）收银员，他们把袋子交给顾客，然后拿走他们的钱。每个员工都可以被视为一个连续的沟通过程。他们使用什么形式的进程间通信？将此模型与UNIX中的进程关联。

37. Suppose that we have a message-passing system using mailboxes. When sending to a full mailbox or trying to receive from an empty one, a process does not block. Instead, it gets an error code back. The process responds to the error code by just trying again, over and over, until it succeeds. Does this scheme lead to race conditions?
37. 假设我们有一个使用邮箱的消息传递系统。当发送到已满邮箱或试图从空邮箱接收时，进程不会阻塞。相反，它会返回一个错误代码。该过程对错误代码的响应是，一次又一次地重试，直到成功。这个计划会导致比赛条件吗？

38. The CDC 6600 computers could handle up to 10 I/O processes simultaneously using an interesting form of round-robin scheduling called processor sharing. A process switch occurred after each instruction, so instruction 1 came from process 1, instruction 2 came from process 2, etc. The process switching was done by special hardware, and the overhead was zero. If a process needed T sec to complete in the absence of competition, how much time would it need if processor sharing was used with n processes?
38. CDC 6600计算机可以使用一种称为处理器共享的有趣的循环调度形式，同时处理多达10个I/O进程。每个指令之后都会发生进程切换，因此指令1来自进程1，指令2来自进程2，等等。进程切换由专用硬件完成，开销为零。如果一个进程在没有竞争的情况下需要T秒才能完成，那么如果处理器共享与n个进程一起使用，需要多少时间？

The time taken to complete will be nT seconds
Assume, time quantum=q seconds
Assume that the process runs for q seconds
so 1 cycle completion time =nq seconds

Let total time = T
Now, time for completion = nq + nq + nq + … =n( q + q + q + … ) = nT

39. Consider the following piece of C code:
void main( ) 
{ 
    fork( ); 
    fork( ); 
    exit( );
}
How many child processes are created upon execution of this program?

Total child process = 2^n -1  , where n is number of time fork will call
2^2-1 = 3

40. Round-robin schedulers normally maintain a list of all runnable processes, with each process occurring exactly once in the list. What would happen if a process occurred twice in the list? Can you think of any reason for allowing this?

If a process occurs twice in that list then it would be executed twice by the processor.
The reason for allowing this may be to increase priority of a process as it will be executed as many number of times as it is in the list.

41. Can a measure of whether a process is likely to be CPU bound or I/O bound be deter- mined by analyzing source code? How can this be determined at run time?


In simple cases it may be possible to see if I/O will be limiting by looking at source code. For instance a program that reads all its input files into buffers at the start will probably not be I/O bound, but a problem that reads and writes incrementally to a number of different files (such as a compiler) is likely to be I/O bound. If the operating system provides a facility such as the UNIX ps command that can tell you the amount of CPU time used by a program, you can compare this with the total time to complete execution of the program. This is, of course, most meaningful on a system where you are the only user.

Reference: https://met.guc.edu.eg/Download.ashx?id=29695&file=OS_PA_3_Summer_19_29695.pdf

42. Explain how time quantum value and context switching time affect each other, in a round-robin scheduling algorithm.

There is an inverse relation between time quantum and context switching in a round robin scheduling as very high time quantum may lead to starvation , i.e. , other processes have to wait to more. If time quantum is small then there will be more context switching and more process will get time to be executed for lesser time period .

43. Measurements of a certain system have shown that the average process runs for a time T before blocking on I/O. A process switch requires a time S, which is effectively wasted (overhead). For round-robin scheduling with quantum Q, give a formula for the CPU efficiency for each of the following:
(a) Q = ∞
(b) Q > T
(c) S < Q < T (d) Q = S
(e) Q nearly 0

Answer:

CPU efficiency = useful CPU time / total CPU time

44. Five jobs are waiting to be run. Their expected run times are 9, 6, 3, 5, and X. In what order should they be run to minimize average response time? (Your answer will depend on X.)

Answer:

For minimizing the average response time, the processes have to be executed according to the Shortest Job First.
0<𝐗<=3:𝐗,3,5,6,9
3<𝐗<=5:3,𝐗,5,6,9
5<𝐗<=6:3,5,𝐗,6,9
6<𝐗<=9:3,5,6,𝐗,9
𝐗>9:3,5,6,9,𝐗

45. Five batch jobs. A through E, arrive at a computer center at almost the same time. They have estimated running times of 10, 6, 2, 4, and 8 minutes. Their (externally de- termined) priorities are 3, 5, 2, 1, and 4, respectively, with 5 being the highest priority. For each of the following scheduling algorithms, determine the mean process turnaround time. Ignore process switching overhead.
(a) Round robin.
(b) Priority scheduling.
(c) First-come, first-served (run in order 10, 6, 2, 4, 8). (d) Shortest job first.
For (a), assume that the system is multiprogrammed, and that each job gets its fair share of the CPU. For (b) through (d), assume that only one job at a time runs, until it finishes. All jobs are completely CPU bound.

46. A process running on CTSS needs 30 quanta to complete. How many times must it be swapped in, including the very first time (before it has run at all)?

Answer:
The first time it receives 1 quantum.
In the following execution, it receives 2,4,8,and15. Hence, it should be swapped 5 times.
2^nd quanta of time: 1+2+4+8+15=30

47. Consider a real-time system with two voice calls of periodicity 5 msec each with CPU time per call of 1 msec, and one video stream of periodicity 33 ms with CPU time per call of 11 msec. Is this system schedulable?
48. 
yes 
because totel time will be 774msec when we calculate
https://stackoverflow.com/questions/22260913/is-a-real-time-system-sdhedulable

48. For the above problem, can another video stream be added and have the system still be schedulable?


49. The aging algorithm with a = 1/2 is being used to predict run times. The previous four runs, from oldest to most recent, are 40, 20, 40, and 15 msec. What is the prediction of the next time?

Answer: 
On taking all the 4 previous run times, the prediction

=(((40+20)/2+40)/2+15)/2
=((30+40/2+15)/2
=(35+15)/2
=25
On taking only 2 previous executions, the prediction=（40+15）/ 2=27.5

50. A soft real-time system has four periodic events with periods of 50, 100, 200, and 250 msec each. Suppose that the four events require 35, 20, 10, and x msec of CPU time, respectively. What is the largest value of x for which the system is schedulable?

Answer:
We require 35/50+20/100+10/200+x/250≤1
So, the largest possible value of x=0.7+0.2+0.005+x/250=1 ⇒ x=12.5
The answer is x=12.5

51.  In the dining philosophers problem, let the following protocol be used: An even-num- bered philosopher always picks up his left fork before picking up his right fork; an odd-numbered philosopher always picks up his right fork before picking up his left fork. Will this protocol guarantee deadlock-free operation?

52. A real-time system needs to handle two voice calls that each run every 6 msec and con- sume 1 msec of CPU time per burst, plus one video at 25 frames/sec, with each frame requiring 20 msec of CPU time. Is this system schedulable?

The required timeslices are quite a bit smaller than 1 second, so one can try to see if the different tasks can complete their work within 1 second.
Voice runs every 5 ms. 1/0.005 = 200 times/second.
Video runs at 25 frames/second. = 25 times/second
Voice needs 1 ms each time it's run = 200 ms/second.
Video needs 20 ms each time it's run = 25*0.020 = 500 ms
2 voice tasks + 1 video task = 200ms*2 + 500ms = 900ms

How one wants an RTOS to schedule such tasks depends among other things on how much jitter you can live with for the different tasks . e.g. the 2 voice tasks could have equal priority, but higher than than the video task - allowing the voice tasks to run whenever they need in a fifo order. (meaning one voice task might need to wait at most 1ms to be scheduled), and the video task gets the remaining CPU time

53. Consider a system in which it is desired to separate policy and mechanism for the scheduling of kernel threads. Propose a means of achieving this goal.


54. In the solution to the dining philosophers problem (Fig. 2-47), why is the state variable set to HUNGRY in the procedure take forks?


55. Consider the procedure put forks in Fig. 2-47. Suppose that the variable state[i] was set to THINKING after the two calls to test, rather than before. How would this change affect the solution?


56. The readers and writers problem can be formulated in several ways with regard to which category of processes can be started when. Carefully describe three different variations of the problem, each one favoring (or not favoring) some category of proc- esses. For each variation, specify what happens when a reader or a writer becomes ready to access the database, and what happens when a process is finished.


57. Write a shell script that produces a file of sequential numbers by reading the last number in the file, adding 1 to it, and then appending it to the file. Run one instance of the script in the background and one in the foreground, each accessing the same file. How long does it take before a race condition manifests itself? What is the critical region? Modify the script to prevent the race.
(Hint: use ln file file.lock to lock the data file.)

58. Assume that you have an operating system that provides semaphores. Implement a message system. Write the procedures for sending and receiving messages.


59. Solve the dining philosophers problem using monitors instead of semaphores.

https://www.geeksforgeeks.org/dining-philosophers-solution-using-monitors/

60. Suppose that a university wants to show off how politically correct it is by applying the U.S. Supreme Court’s ‘‘Separate but equal is inherently unequal’’ doctrine to gender as well as race, ending its long-standing practice of gender-segregated bathrooms on campus. However, as a concession to tradition, it decrees that when a woman is in a bathroom, other women may enter, but no men, and vice versa. A sign with a sliding marker on the door of each bathroom indicates which of three possible states it is currently in:
Empty
Women present 
Men present
In some programming language you like, write the following procedures: woman_wants_to_enter, man_wants_to_enter, woman_leaves, man_leaves. You may use whatever counters and synchronization techniques you like. 

61. Rewrite the program of Fig. 2-23 to handle more than two processes.


62. Write a producer-consumer problem that uses threads and shares a common buffer. However, do not use semaphores or any other synchronization primitives to guard the shared data structures. Just let each thread access them when it wants to. Use sleep and wakeup to handle the full and empty conditions. See how long it takes for a fatal race condition to occur. For example, you might have the producer print a number once in a while. Do not print more than one number every minute because the I/O could affect the race conditions.


63. A process can be put into a round-robin queue more than once to give it a higher priority. Running multiple instances of a program each working on a different part of a data pool can have the same effect. First write a program that tests a list of numbers for primality. Then devise a method to allow multiple instances of the program to run at once in such a way that no two instances of the program will work on the same number. Can you in fact get through the list faster by running multiple copies of the program? Note that your results will depend upon what else your computer is doing; on a personal computer running only instances of this program you would not expect an improvement, but on a system with other processes, you should be able to grab a bigger share of the CPU this way.


64. The objective of this exercise is to implement a multithreaded solution to find if a given number is a perfect number. 𝑁 is a perfect number if the sum of all its factors, excluding itself, is 𝑁; examples are 6 and 28. The input is an integer, 𝑁. The output is true if the number is a perfect number and false otherwise. The main program will read the numbers 𝑁 and 𝑃 from the command line. The main process will spawn a set of 𝑃 threads. The numbers from 1 to 𝑁 will be partitioned among these threads so that two threads do not work on the name number. For each number in this set, the thread will determine if the number is a factor of 𝑁. If it is, it adds the number to a shared buffer that stores factors of 𝑁. The parent process waits till all the threads complete. Use the appropriate synchronization primitive here. The parent will then determine if the input number is perfect, that is, if 𝑁 is a sum of all its factors and then report accordingly. (Note: You can make the computation faster by restricting the numbers searched from 1 to the square root of 𝑁.)


65. Implement a program to count the frequency of words in a text file. The text file is partitioned into 𝑁 segments. Each segment is processed by a separate thread that outputs the intermediate frequency count for its segment. The main process waits until all the threads complete; then it computes the consolidated word-frequency data based on the individual threads’ output.