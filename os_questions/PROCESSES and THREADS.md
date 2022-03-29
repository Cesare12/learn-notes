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
