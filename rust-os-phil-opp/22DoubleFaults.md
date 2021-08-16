# Double Faults

## 1. What is a Double Fault?
当 CPU 调用异常处理失败时，就会出现 Double Fault。例如触发 page fault 时，在 IDT 中没有 page fault 处理函数。

Double Fault 很重要，因为没有它，就会出现 triple fault，这将导致硬件重启。
### Triggering a Double Fault
## 2. A Double Fault Handler
## 3. Causes of Double Faults
### Kernel Stack Overflow
## 4. Switching Stacks
### The IST and TSS
### Creating a TSS
### The Global Descriptor Table
### The final Steps
## 5. A Stack Overflow Test
### Implementing _start
### The Test IDT
### The Double Fault Handler
## 6. Summart
