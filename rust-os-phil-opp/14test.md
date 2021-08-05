# Testing
# 内核测试

## 1.Rust中的测试

    > cargo xtest
    Compiling blog_os v0.1.0 (/…/blog_os)
    error[E0463]: can't find crate for `test`

## 2.自定义测试框架

解决在 #[no_std] 环境下可以用测试框架。

    // in src/main.rs

    #![feature(custom_test_frameworks)]
    #![test_runner(crate::test_runner)]

    #[cfg(test)]
    fn test_runner(tests: &[&dyn Fn()]) {
        println!("Running {} tests", tests.len());
        for test in tests {
            test();
        }
    }

    // in src/main.rs

    #![reexport_test_harness_main = "test_main"]

    #[no_mangle]
    pub extern "C" fn _start() -> ! {
        println!("Hello World{}", "!");

        #[cfg(test)]
        test_main();

        loop {}
    }

    // in src/main.rs

    #[test_case]
    fn trivial_assertion() {
        print!("trivial assertion... ");
        assert_eq!(1, 1);
        println!("[ok]");
    }

## 3.退出QEMU
    # in Cargo.toml

    [package.metadata.bootimage]
    test-args = ["-device", "isa-debug-exit,iobase=0xf4,iosize=0x04"]
### I/O端口
在x86平台上，CPU和外围硬件通信通常有两种方式，内存映射I/O和端口映射I/O。通过内存地址0xb8000访问 VGA文本缓冲区 就是 内存映射。isa-debug-exit设备使用的就是端口映射I/O。其中， iobase 参数指定了设备对应的端口地址（在x86中，0xf4是一个通常未被使用的端口），而iosize则指定了端口的大小（0x04代表4字节）。
### 使用退出设备
    # in Cargo.toml

    [dependencies]
    x86_64 = "0.11.0"

    // in src/main.rs

    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    #[repr(u32)]
    pub enum QemuExitCode {
        Success = 0x10,
        Failed = 0x11,
    }

    pub fn exit_qemu(exit_code: QemuExitCode) {
        use x86_64::instructions::port::Port;

        unsafe {
            let mut port = Port::new(0xf4);
            port.write(exit_code as u32);
        }
    }

    fn test_runner(tests: &[&dyn Fn()]) {
        println!("Running {} tests", tests.len());
        for test in tests {
            test();
        }
        /// new
        exit_qemu(QemuExitCode::Success);
    }

### 成功退出代码
    [package.metadata.bootimage]
    test-args = […]
    test-success-exit-code = 33         # (0x10 << 1) | 1
## 4.打印到控制台
### 串口
    # in Cargo.toml

    [dependencies]
    uart_16550 = "0.2.0"

使用uart_16550 crate来初始化UART，并通过串口来发送数据。

    // in src/main.rs

    mod serial;
    use uart_16550::SerialPort;
    use spin::Mutex;
    use lazy_static::lazy_static;

    lazy_static! {
        pub static ref SERIAL1: Mutex<SerialPort> = {
            let mut serial_port = unsafe { SerialPort::new(0x3F8) };
            serial_port.init();
            Mutex::new(serial_port)
        };
    }

    #[doc(hidden)]
    pub fn _print(args: ::core::fmt::Arguments) {
        use core::fmt::Write;
        SERIAL1.lock().write_fmt(args).expect("Printing to serial failed");
    }

    /// Prints to the host through the serial interface.
    #[macro_export]
    macro_rules! serial_print {
        ($($arg:tt)*) => {
            $crate::serial::_print(format_args!($($arg)*));
        };
    }

    /// Prints to the host through the serial interface, appending a newline.
    #[macro_export]
    macro_rules! serial_println {
        () => ($crate::serial_print!("\n"));
        ($fmt:expr) => ($crate::serial_print!(concat!($fmt, "\n")));
        ($fmt:expr, $($arg:tt)*) => ($crate::serial_print!(
            concat!($fmt, "\n"), $($arg)*));
    }

    #[cfg(test)]
    fn test_runner(tests: &[&dyn Fn()]) {
        serial_println!("Running {} tests", tests.len());
        […]
    }

    #[test_case]
    fn trivial_assertion() {
        serial_print!("trivial assertion... ");
        assert_eq!(1, 1);
        serial_println!("[ok]");
    }

### QEMU参数
    # in Cargo.toml

    [package.metadata.bootimage]
    test-args = [
        "-device", "isa-debug-exit,iobase=0xf4,iosize=0x04", "-serial", "stdio"
    ]
### 在panic时打印一个错误信息
在我们的测试panic处理中，我们用 serial_println来代替println 并使用失败代码来退出QEMU。
### 隐藏QEMU
    # in Cargo.toml

    [package.metadata.bootimage]
    test-args = [
        "-device", "isa-debug-exit,iobase=0xf4,iosize=0x04", "-serial", "stdio",
        "-display", "none"
    ]
### 超时
- bootloader加载内核失败，导致系统不停重启；
- BIOS/UEFI固件加载bootloader失败，同样会导致无限重启；
- CPU在某些函数结束时进入一个loop {}语句，例如因为QEMU的exit设备无法正常工作而导致死循环；
- 硬件触发了系统重置，例如未捕获CPU异常时（后续的文章将会详细解释）。
  

    # in Cargo.toml
    [package.metadata.bootimage]
    test-timeout = 300          # (in seconds)

## 5.测试VGA缓冲区
测试打印函数。还可以测试：当打印一个很长的且包装正确的行时是否会发生panic的函数，或是一个用于测试换行符、不可打印字符、非unicode字符是否能被正确处理的函数。
## 6.集成测试
basic_boot
### 创建一个库
创建lib.rs
### 完成集成测试
tests/basic_boot.rs中写测试用例
### 未来的测试
- CPU异常：当代码执行无效操作（例如除以零）时，CPU就会抛出异常。内核会为这些异常注册处理函数。集成测试可以验证在CPU异常时是否调用了正确的异常处理程序，或者在可解析的异常之后程序是否能正确执行；
- 页表：页表定义了哪些内存区域是有效且可访问的。通过修改页表，可以重新分配新的内存区域，例如，当你启动一个软件的时候。我们可以在集成测试中调整_start函数中的一些页表项，并确认这些改动是否会对#[test_case]的函数产生影响；
- 用户空间程序：用户空间程序是只能访问有限的系统资源的程序。例如，他们无法访问内核数据结构或是其他应用程序的内存。集成测试可以启动执行禁止操作的用户空间程序验证认内核是否会将这些操作全都阻止。
### 那些应该panic的测试
### 无约束的测试
## 7.代码

lib.rs

    // lib.rs
    #![no_std]
    #![cfg_attr(test, no_main)]
    #![feature(custom_test_frameworks)]
    #![test_runner(crate::test_runner)]
    #![reexport_test_harness_main = "test_main"]
    pub mod serial;
    pub mod vga_buffer;
    use core::panic::PanicInfo;

    pub fn test_runner(tests: &[&dyn Fn()]) {
        serial_println!("Running {} tests", tests.len());
        for test in tests {
            test();
        }
        exit_qemu(QemuExitCode::Success);
    }

    pub fn test_panic_handler(info: &PanicInfo) -> ! {
        serial_println!("[failed]\n");
        serial_println!("Error: {}\n", info);
        exit_qemu(QemuExitCode::Failed);
        loop {}
    }

    /// Entry point for `cargo xtest`
    #[cfg(test)]
    #[no_mangle]
    pub extern "C" fn _start() -> ! {
        test_main();
        loop {}
    }

    #[cfg(test)]
    #[panic_handler]
    fn panic(info: &PanicInfo) -> ! {
        test_panic_handler(info)
    }

    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    #[repr(u32)]
    pub enum QemuExitCode {
        Success = 0x10,
        Failed = 0x11,
    }

    pub fn exit_qemu(exit_code: QemuExitCode) {
        use x86_64::instructions::port::Port;

        unsafe {
            let mut port = Port::new(0xf4);
            port.write(exit_code as u32);
        }
    }

serial.rs

    //serial.rs
    use lazy_static::lazy_static;
    use spin::Mutex;
    use uart_16550::SerialPort;

    lazy_static! {
        pub static ref SERIAL1: Mutex<SerialPort> = {
            let mut serial_port = unsafe { SerialPort::new(0x3F8) };
            serial_port.init();
            Mutex::new(serial_port)
        };
    }

    #[doc(hidden)]
    pub fn _print(args: ::core::fmt::Arguments) {
        use core::fmt::Write;
        use x86_64::instructions::interrupts;

        interrupts::without_interrupts(|| {
            SERIAL1
                .lock()
                .write_fmt(args)
                .expect("Printing to serial failed");
        });
    }

    /// Prints to the host through the serial interface.
    #[macro_export]
    macro_rules! serial_print {
        ($($arg:tt)*) => {
            $crate::serial::_print(format_args!($($arg)*));
        };
    }

    /// Prints to the host through the serial interface, appending a newline.
    #[macro_export]
    macro_rules! serial_println {
        () => ($crate::serial_print!("\n"));
        ($fmt:expr) => ($crate::serial_print!(concat!($fmt, "\n")));
        ($fmt:expr, $($arg:tt)*) => ($crate::serial_print!(
            concat!($fmt, "\n"), $($arg)*));
    }

main.rs

    #![no_std]
    #![no_main]
    #![feature(custom_test_frameworks)]
    #![test_runner(blog_os::test_runner)]
    #![reexport_test_harness_main = "test_main"]

    use core::panic::PanicInfo;
    use blog_os::println;

    #[no_mangle]
    pub extern "C" fn _start() -> ! {
        println!("Hello World{}", "!");

        #[cfg(test)]
        test_main();

        loop {}
    }

    /// This function is called on panic.
    #[cfg(not(test))]
    #[panic_handler]
    fn panic(info: &PanicInfo) -> ! {
        println!("{}", info);
        loop {}
    }

    #[cfg(test)]
    #[panic_handler]
    fn panic(info: &PanicInfo) -> ! {
        blog_os::test_panic_handler(info)
    }