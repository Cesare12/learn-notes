# Double Faults

出现 fault 时，CPU 会先在 IDT (Interrupt Descriptor Table) 中查找异常处理函数，然后将 interrupt stack frame 压入栈，当被压入的栈是非法的时候，就可能出现 triple fault。为了保证被压入的栈是合法的，就需要在压入栈之前切换到合法的栈。

切换机制：用 Interrupt Stack Table (IST) 实现，IST是预先设置好的栈表，出现 fault 时，CPU 在 IDT 中找到处理函数后，在 IST 中再找要切换到对应的栈，这样就保证栈一直有效的。

解决步骤：
1. 重载 code segment register，以便指向一个不同的 GDT；
2. GDT 已经包含了一个 TSS selector，指明一下 TSS ；
3. 更新 IDT 入口：加载 TSS 后，CPU 已经可以访问有效的 IST。这样修改 IDT 入口后，CPU 可以使用对应的 fault 栈。

## 1. What is a Double Fault?
当 CPU 调用异常处理失败时，就会出现 Double Fault。例如触发 page fault 时，在 IDT 中没有 page fault 处理函数。

Double Fault 很重要，因为没有它，就会出现 triple fault，这将导致硬件重启。
### Triggering a Double Fault
往一个无效地址里写值

1. CPU 尝试往 0xdeadbeef 里写值，引起 page fault；
2. 在 IDT 里找相关条目，找不到 page fault，因此引起 Double Fault；
3. 在 IDT 里找 Double Fault 的处理函数，找不到，因此 triple fault；
4. Triple fault 致命的，因此 QEMU 重启。

## 2. A Double Fault Handler
和 page fault 处理类似。但是会有一些特殊的情况没法包含。

## 3. Causes of Double Faults

之前对 double fault 的定义：

    A double fault is a special exception that occurs when the CPU fails to invoke an exception handler.

1. 当一个断点异常出现时，对应的处理函数被 swapped out 会发生什么？
2. page fault 出现时，page fault 的处理函数被 swapped out 会发生什么？
3. 一个除以0的处理函数 引起断点异常，但是断点异常的处理被 swapped out 会发生什么？
4. 我们的内核溢出它的堆栈，然后保护页面被击中 会发生什么？

根据 AMD64 的手册定义：
    a double fault exception can occur when a second exception occurs during the handling of a prior (first) exception handler.

意思就是只有特殊的异常组合才会出现 double fault。

<table><thead><tr><th>First Exception</th><th>Second Exception</th></tr></thead><tbody>
<tr><td><a href="https://wiki.osdev.org/Exceptions#Divide-by-zero_Error">Divide-by-zero</a>,<br><a href="https://wiki.osdev.org/Exceptions#Invalid_TSS">Invalid TSS</a>,<br><a href="https://wiki.osdev.org/Exceptions#Segment_Not_Present">Segment Not Present</a>,<br><a href="https://wiki.osdev.org/Exceptions#Stack-Segment_Fault">Stack-Segment Fault</a>,<br><a href="https://wiki.osdev.org/Exceptions#General_Protection_Fault">General Protection Fault</a></td><td><a href="https://wiki.osdev.org/Exceptions#Invalid_TSS">Invalid TSS</a>,<br><a href="https://wiki.osdev.org/Exceptions#Segment_Not_Present">Segment Not Present</a>,<br><a href="https://wiki.osdev.org/Exceptions#Stack-Segment_Fault">Stack-Segment Fault</a>,<br><a href="https://wiki.osdev.org/Exceptions#General_Protection_Fault">General Protection Fault</a></td></tr>
<tr><td><a href="https://wiki.osdev.org/Exceptions#Page_Fault">Page Fault</a></td><td><a href="https://wiki.osdev.org/Exceptions#Page_Fault">Page Fault</a>,<br><a href="https://wiki.osdev.org/Exceptions#Invalid_TSS">Invalid TSS</a>,<br><a href="https://wiki.osdev.org/Exceptions#Segment_Not_Present">Segment Not Present</a>,<br><a href="https://wiki.osdev.org/Exceptions#Stack-Segment_Fault">Stack-Segment Fault</a>,<br><a href="https://wiki.osdev.org/Exceptions#General_Protection_Fault">General Protection Fault</a></td></tr>
</tbody></table>

比如 divide-by-zero fault 接 page fault 没有问题，divide-by-zero fault 接 general-protection fault 就会引起 double fault。

可以回答上面的三个问题：
1. 如果断点异常出现，对应的处理函数 swapped out，那么 page fault 就会出现，page fault 处理被调用；
2. 如果page fault 出现时，page fault 的处理函数被 swapped out，那么 double fault 就会出现，double fault 处理被调用；
3. 如果除以0的处理函数 引起断点异常，CPU 会尝试调用断点处理。如果断点处理被 swapped out，page fault 就会出现，page fault 处理被调用。

### Kernel Stack Overflow
第四个问题：
    我们的内核溢出它的堆栈，然后保护页面被击中 会发生什么？

Guard page 是一个特殊的内存页，放在栈底，以检测栈溢出。这个 page 不会 map 任何物理帧，所以访问它会引起 page fault。bootloader 给内核设置了一个保护页，因此栈溢出会导致 page fault。

Page fault 出现时，CPU 会查找 IDT 然后将 interrupt stack frame 压入栈。但是，当前栈指针仍然指在 guard page，因此第二个 page fault 出现，引起 double fault。

CPU 现在想调用 double fault 处理函数，但是 CPU 在 double fault 下仍尝试将 interrupt stack frame 压入栈。但是，当前栈指针仍然指在 guard page，因此 triple fault 出现。

所以当前的 double fault 处理不能避免出现 triple fault。

因为压入异常栈帧是 CPU 自己做的，所以不能省略。那么我们就要 保证在 double fault 出现时，栈是有效的。这里 x86_64 已经有了解决方案。

## 4. Switching Stacks
x86_64 架构可以在异常出现时切换到一个预先定义好的栈，这发生在硬件层面，所以栈切换可以在 CPU 压入异常栈帧之前完成。

切换机制 用 Interrupt Stack Table (IST) 实现， IST 由7个指向已知能用的栈 的指针组成。

对于每次异常，都可以根据 IDT 的栈指针字段 从 IST 中选一个栈。例如，对于 double fault 的处理可以在 IST 中选第一个栈，然后无论什么时候发生 double fault， CPU 都会切换到这个栈。这个切换会发生在任何东西压栈前，所以 triple fault 不会发生。

### The IST and TSS
Interrupt Stack Table (IST) 是 Task State Segment (TSS) 的一部分。

64-bit TSS 包含下面几个表：

<table><thead><tr><th>Field</th><th>Type</th></tr></thead><tbody>
<tr><td><span style="opacity: 0.5">(reserved)</span></td><td><code>u32</code></td></tr>
<tr><td>Privilege Stack Table</td><td><code>[u64; 3]</code></td></tr>
<tr><td><span style="opacity: 0.5">(reserved)</span></td><td><code>u64</code></td></tr>
<tr><td>Interrupt Stack Table</td><td><code>[u64; 7]</code></td></tr>
<tr><td><span style="opacity: 0.5">(reserved)</span></td><td><code>u64</code></td></tr>
<tr><td><span style="opacity: 0.5">(reserved)</span></td><td><code>u16</code></td></tr>
<tr><td>I/O Map Base Address</td><td><code>u16</code></td></tr>
</tbody></table>

### Creating a TSS
x86_64 crate 自带 TaskStateSegment 结构体。

新建一个 gdt 模块 Global Descriptor Table (GDT)。
### The Global Descriptor Table
曾经用来内存分区，但是已经被 page standard 替代，目前的作用：

1. 在内核空间和用户空间切换；
2. 加载 TSS 结构体。
### The final Steps
1. 重载代码段寄存器：改变 GDT 后，要重载 代码段寄存器。因为旧的段选择器不能指向一个不同的 GDT。
2. 加载 TSS
3. 更新 IDT
## 5. A Stack Overflow Test
### Implementing _start
### The Test IDT
### The Double Fault Handler
## 6. Summary
In this post we learned what a double fault is and under which conditions it occurs. We added a basic double fault handler that prints an error message and added an integration test for it.

We also enabled the hardware supported stack switching on double fault exceptions so that it also works on stack overflow. While implementing it, we learned about the task state segment (TSS), the contained interrupt stack table (IST), and the global descriptor table (GDT), which was used for segmentation on older architectures.
