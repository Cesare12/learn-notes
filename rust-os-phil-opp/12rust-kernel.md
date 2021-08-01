# A Minimal Rust Kernel
# 最小化内核

## 1. BIOS(Basic Input Output System)启动
启动电脑时，主板ROM内存储的固件会：

    1.上电自检；可用RAM检测；CPU和其他硬件的预加载；
    2.寻找一个可引导的存储介质。电脑控制权移交给bootloader（通常情况下，引导程序都被切分为一段优先启动、长度不超过512字节、存储在介质开头的第一阶段引导程序，和一段随后由其加载的、长度可能较长、存储在其它位置的第二阶段引导程序。）
    3.bootloader将内核加载到内存；将CPU 从 16 位的实模式，先切换到 32 位的保护模式（protected mode），最终切换到 64 位的长模式（long mode）：此时，所有的 64 位寄存器和整个主内存（main memory）才能被访问；从 BIOS 查询特定的信息，并将其传递到内核；如查询和传递内存映射表（memory map）。

## 2.最小化内核
创建一个内核的磁盘映像，在启动时打印出“Hello World!”。

## 3.安装 Nightly Rust
在项目根目录添加一个名称为 rust-toolchain，内容为 nightly 的文件。可以运行 rustc --version：返回的版本号末尾应该包含-nightly。

## 4.目标配置清单

    {
        "llvm-target": "x86_64-unknown-none",
        "data-layout": "e-m:e-i64:64-f80:128-n8:16:32:64-S128",
        "arch": "x86_64",
        "target-endian": "little",
        "target-pointer-width": "64",
        "target-c-int-width": "32",
        "os": "none",
        "executables": true,
        "linker-flavor": "ld.lld",
        "linker": "rust-lld",
        "panic-strategy": "abort",
        "disable-redzone": true,
        "features": "-mmx,-sse,+soft-float"
    }

## 5.编译内核

    // src/main.rs

    #![no_std] // 不链接 Rust 标准库
    #![no_main] // 禁用所有 Rust 层级的入口点

    use core::panic::PanicInfo;

    /// 这个函数将在 panic 时被调用
    #[panic_handler]
    fn panic(_info: &PanicInfo) -> ! {
        loop {}
    }

    #[no_mangle] // 不重整函数名
    pub extern "C" fn _start() -> ! {
        // 因为编译器会寻找一个名为 `_start` 的函数，所以这个函数就是入口点
        // 默认命名为 `_start`
        loop {}
    }

### cargo xbuild
使用cargo xbuild工具自动交叉编译 core 库和一些编译器内建库（compiler built-in libraries）。

    rustup component add rust-src
    cargo install cargo-xbuild

### 设置默认目标
    # in .cargo/config

    [build]
    target = "x86_64-blog_os.json"

### 向屏幕打印字符
    tatic HELLO: &[u8] = b"Hello World!";

    #[no_mangle]
    pub extern "C" fn _start() -> ! {
        let vga_buffer = 0xb8000 as *mut u8;

        for (i, &byte) in HELLO.iter().enumerate() {
            unsafe {
                *vga_buffer.offset(i as isize * 2) = byte;
                *vga_buffer.offset(i as isize * 2 + 1) = 0xb;
            }
        }

        loop {}
    }
## 5.启动内核
首先，我们将编译完毕的内核与引导程序链接，来创建一个引导映像；这之后，我们可以在 QEMU 虚拟机中运行它。

### 创建引导映像
    # in Cargo.toml

    [dependencies]
    bootloader = "0.9.3"

    [package.metadata.bootimage]
    build-command = ["xbuild"]

使用 bootimage 工具，将会在内核编译完毕后，将它和引导程序组合在一起，最终创建一个能够引导的磁盘映像。

    cargo install bootimage
    > cargo bootimage

bootimage 工具执行了三个步骤：

    1.编译我们的内核为一个 ELF（Executable and Linkable Format）文件；
    2.编译引导程序为独立的可执行文件；
    3.将内核 ELF 文件按字节拼接（append by bytes）到引导程序的末端。

当机器启动时，引导程序将会读取并解析拼接在其后的 ELF 文件。这之后，它将把程序片段映射到分页表（page table）中的虚拟地址（virtual address），清零 BSS段（BSS segment），还将创建一个栈。最终它将读取入口点地址（entry point address）——我们程序中 _start 函数的位置——并跳转到这个位置。

### QEMU中启动内核

    > qemu-system-x86_64 -drive format=raw,file=bootimage-blog_os.bin

### 使用cargo xrun

    # in .cargo/config

    [target.'cfg(target_os = "none")']
    runner = "bootimage runner"