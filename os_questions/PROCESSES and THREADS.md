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
