# CPU Exceptions
# CPU 异常

## 1. Overview
x86上大概有20种不同类型的异常，最重要的有：
- Page Fault：操作非法内存，例如从没有映射的页读数据 或者 让只读的页写数据。
- Invalid Opcode：使用当前CPU不支持的指令。
- General Protection Fault：违规访问，例如在用户层级执行root指令或者在配置寄存器中写入保留字段。
- Double Fault：两个不同的异常同时想调用同一个异常处理函数。
- Triple Fault：CPU想调用处理Double Fault的函数时，又出现了异常。以情况下一般直接重启。

### The Interrupt Descriptor Table 中断描述表
一个 16-byte 的结构体

1. Push some registers on the stack, including the instruction pointer and the RFLAGS register. 
2. Read the corresponding entry from the Interrupt Descriptor Table (IDT). For example, the CPU reads the 14-th entry when a page fault occurs.
3. Check if the entry is present. Raise a double fault if not.
4. Disable hardware interrupts if the entry is an interrupt gate (bit 40 not set).
5. Load the specified GDT selector into the CS segment.
6. Jump to the specified handler function.
## 2. An IDT Type
    #[repr(C)]
    pub struct InterruptDescriptorTable {
        pub divide_by_zero: Entry<HandlerFunc>,
        pub debug: Entry<HandlerFunc>,
        pub non_maskable_interrupt: Entry<HandlerFunc>,
        pub breakpoint: Entry<HandlerFunc>,
        pub overflow: Entry<HandlerFunc>,
        pub bound_range_exceeded: Entry<HandlerFunc>,
        pub invalid_opcode: Entry<HandlerFunc>,
        pub device_not_available: Entry<HandlerFunc>,
        pub double_fault: Entry<HandlerFuncWithErrCode>,
        pub invalid_tss: Entry<HandlerFuncWithErrCode>,
        pub segment_not_present: Entry<HandlerFuncWithErrCode>,
        pub stack_segment_fault: Entry<HandlerFuncWithErrCode>,
        pub general_protection_fault: Entry<HandlerFuncWithErrCode>,
        pub page_fault: Entry<PageFaultHandlerFunc>,
        pub x87_floating_point: Entry<HandlerFunc>,
        pub alignment_check: Entry<HandlerFuncWithErrCode>,
        pub machine_check: Entry<HandlerFunc>,
        pub simd_floating_point: Entry<HandlerFunc>,
        pub virtualization: Entry<HandlerFunc>,
        pub security_exception: Entry<HandlerFuncWithErrCode>,
        // some fields omitted
    }

x86-interrupt calling convention

    type HandlerFunc = extern "x86-interrupt" fn(_: InterruptStackFrame);

## 3. The interrupt Calling Convention 中断调用约定
一次函数调用是自愿地被编译器调用插入到call指令，然而异常却可能出现在任何地方。
rust中用 extern "C" fn 声明的函数遵循以下规则（x86_64 Linux System V ABI 二进制接口）：
- 前6个整形参数被传入到 rdi, rsi, rdx, rcx, r8, r9 寄存器
- 剩余的参数在栈上传递
- 结果返回到 rax 和 rdx

### Preserver and Scratch Registers 保留寄存器和可擦除寄存器
调用约定把寄存器分成两部分：Preserver Registers 和 Scratch Registers

Callee is only allowed to overwrite these registers if it restores their original values before returning. 

Called is allowed to overwrite scratch registers without restrictions.

preserved registers	|   scratch registers
---                 |   ---
rbp, rbx, rsp, r12, r13, r14, r15|  rax, rcx, rdx, rsi, rdi, r8, r9, r10, r11
callee-saved        |   caller-saved

### Preserving all Registers
大部分情况下，我们都无法知道什么时候会引起异常。因为我们不知道什么时候出现异常，所以我们不能对寄存器做备份。也就是不能用 caller-saved 寄存器处理异常。x86-interrupt 调用协议就可以保证 所有的寄存器的值在函数 return 时被存储为原始值。

编译器只备份被函数覆盖的寄存器。
### The Interrupt Stack Frame
对于正常的函数调用，CPU先把 return 地址push 到栈，然后再跳转到目标函数。函数return 时，CPU 再pop 这个地址然后跳转到那。

当中断出现时，CPU 表现如下：
1. Aligning the stack pointer: An interrupt can occur at any instructions, so the stack pointer can have any value, too. However, some CPU instructions (e.g. some SSE instructions) require that the stack pointer is aligned on a 16 byte boundary, therefore the CPU performs such an alignment right after the interrupt.
1. 栈指针对齐：中断出现时，栈指针可以是任何值，但是一些CPU 指令要求16字节对齐，所以CPU 在中断后会执行这样的对齐。

2. Switching stacks (in some cases): A stack switch occurs when the CPU privilege level changes, for example when a CPU exception occurs in a user mode program. It is also possible to configure stack switches for specific interrupts using the so-called Interrupt Stack Table (described in the next post).
2. 切换栈：当CPU 权限改变时，就会出现栈切换

3. Pushing the old stack pointer: The CPU pushes the values of the stack pointer (rsp) and the stack segment (ss) registers at the time when the interrupt occurred (before the alignment). This makes it possible to restore the original stack pointer when returning from an interrupt handler.
3. 压入旧栈指针：在中断发生时，CPU 把栈指针和栈段寄存器压入。这样当中断处理结束后，栈指针可以恢复原始值。

4. Pushing and updating the RFLAGS register: The RFLAGS register contains various control and status bits. On interrupt entry, the CPU changes some bits and pushes the old value.
4. 压入并更新RFLAGS 寄存器：RFLAGS 寄存器含有各种控制和状态位。在中断入口，CPU 改变一些位然后压入旧值。

5. Pushing the instruction pointer: Before jumping to the interrupt handler function, the CPU pushes the instruction pointer (rip) and the code segment (cs). This is comparable to the return address push of a normal function call.
5. 压入指令指针：在跳到中断执行前，CPU 压入指令指针和代码段。这个常规的函数调用类似。

6. Pushing an error code (for some exceptions): For some specific exceptions such as page faults, the CPU pushes an error code, which describes the cause of the exception.
6. 压入错误码：一些中断有相应的错误代码。

7. Invoking the interrupt handler: The CPU reads the address and the segment descriptor of the interrupt handler function from the corresponding field in the IDT. It then invokes this handler by loading the values into the rip and cs registers.
7. 调用中断处理：CPU 从IDT 中读到中断地址等信息，然后通过将值加载到指令指针和代码段寄存器 执行这个处理。

### Behind the Scenes 幕后
x86-interrupt 中断调用协议：
- Retrieving the arguments: Most calling conventions expect that the arguments are passed in registers. This is not possible for exception handlers, since we must not overwrite any register values before backing them up on the stack. Instead, the x86-interrupt calling convention is aware that the arguments already lie on the stack at a specific offset.
- 索引的参数

- Returning using iretq: Since the interrupt stack frame completely differs from stack frames of normal function calls, we can't return from handlers functions through the normal ret instruction. Instead, the iretq instruction must be used.
- 使用iretq 指令

- Handling the error code: The error code, which is pushed for some exceptions, makes things much more complex. It changes the stack alignment (see the next point) and needs to be popped off the stack before returning. The x86-interrupt calling convention handles all that complexity. However, it doesn't know which handler function is used for which exception, so it needs to deduce that information from the number of function arguments. That means that the programmer is still responsible to use the correct function type for each exception. Luckily, the InterruptDescriptorTable type defined by the x86_64 crate ensures that the correct function types are used.
- 处理错误码

- Aligning the stack: There are some instructions (especially SSE instructions) that require a 16-byte stack alignment. The CPU ensures this alignment whenever an exception occurs, but for some exceptions it destroys it again later when it pushes an error code. The x86-interrupt calling convention takes care of this by realigning the stack in this case.
- 栈对齐

## 4. Implementation
### Loading the IDT
### Running it
### Adding a Test
## 5. Too much Magic?
