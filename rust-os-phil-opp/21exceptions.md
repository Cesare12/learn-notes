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

### Preserver and Scratch Registers
### Preserving all Registers
### The Interrupt Stack Frame
### Behind the Scenes
## 4. Implementation
### Loading the IDT
### Running it
### Adding a Test
## 5. Too much Magic?
