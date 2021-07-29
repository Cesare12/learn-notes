# A Freestanding Rust Binary
# 独立可执行文件

目标：创建一个不链接标准库的rust可执行文件

## 1. 禁用标准库

    #![no_std]
    fn main() {
    println!("Hello, world!");
    }

## 2. 添加panic处理函数，在发生panic时调用

    use core::panic::PanicInfo;

    #[panic_handler]
    fn panic(_info: &PanicInfo) -> ! {
        loop {}
    }

## 3.eh_personality语言项

标记某些函数用于实现[栈展开](https://www.bogotobogo.com/cplusplus/stackunwinding.php)。当发生panic时，rust使用栈展开，确保所有使用的内存被释放。目前盏展开特性不是迫切需求。

### 禁用盏展开

    /// Cargo.toml
    [profile.dev]
    panic = "abort"

    [profile.release]
    panic = "abort"

## 4.start语言项

程序运行时，并不是从main开始，而是从crt0 (C runtime zero) 运行时库开始，它建立一个适合运行 C 语言程序的环境，包含创建栈和传入可执行程序的参数。然后，这个运行时库调用[rust运行时入口点](https://github.com/rust-lang/rust/blob/bb4d1491466d8239a7a5fd68bd605e3276e97afb/src/libstd/rt.rs#L32-L73)，这个入口点就是start语言项。

所以需要重写整个 crt0 库和它定义的入口点。

### 重写入口点

    #![no_std] // 不链接 Rust 标准库
    #![no_main] // 告诉编译器不使用预定义的入口点

    use core::panic::PanicInfo;

    #[no_mangle] // 不重整函数名
    pub extern "C" fn _start() -> ! {
        // 因为编译器会寻找一个名为 `_start` 的函数，所以这个函数就是入口点，默认命名为 `_start`
        loop {}
    }

    /// 这个函数将在 panic 时被调用
    #[panic_handler]
    fn panic(_info: &PanicInfo) -> ! {
        loop {}
    }

首先移除了main函数，因为运行时不会调用它。

添加[no_mangle](https://en.wikipedia.org/wiki/Name_mangling)不重整函数名属性，让编译器输出名为_start的函数。

函数标记[extern "C"](https://en.wikipedia.org/wiki/Calling_convention)告诉编译器使用C语言的调用约定。

## 5.链接

rust会根据当前主机环境编译成对应的可执行程序，例如windows -> .exe。所以我们要选择一个底层没有操作系统的运行环境。

    rustup target add thumbv7em-none-eabihf
    cargo build --target thumbv7em-none-eabihf

我们传递了 --target 参数，来为裸机目标系统交叉编译（cross compile）我们的程序。我们的目标并不包括操作系统，所以链接器不会试着链接 C 语言运行环境，因此构建过程成功会完成，不