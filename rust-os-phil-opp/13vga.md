# VGA Text Mode
# VGA字符模式

## 1.VGA字符缓冲区（text buffer）

在VGA字符模式下，可以向硬件提供的VGA字符缓冲区里写入字符，buffer 内容会实时渲染到屏幕。通常情况此buffer是 25 x 80 的二维数组，数组元素是 2bytes 的字符单元。

Bit(s)	|Value
---     |---
0-7	    |ASCII code point
8-11	|Foreground color
12-14	|Background color
15	    |Blink

要修改 VGA 字符缓冲区，我们可以通过存储器映射输入输出（memory-mapped I/O）的方式，读取或写入地址 0xb8000；这意味着，我们可以像操作普通的内存区域一样操作这个地址。

## 2.包装Rust模块

    // in src/main.rs
    mod vga_buffer;

### 颜色

    // in src/vga_buffer.rs

    #[allow(dead_code)]
    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    #[repr(u8)]
    pub enum Color {
        Black = 0,
        Blue = 1,
        Green = 2,
        Cyan = 3,
        Red = 4,
        Magenta = 5,
        Brown = 6,
        LightGray = 7,
        DarkGray = 8,
        LightBlue = 9,
        LightGreen = 10,
        LightCyan = 11,
        LightRed = 12,
        Pink = 13,
        Yellow = 14,
        White = 15,
    }

    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    #[repr(transparent)]
    struct ColorCode(u8);

    impl ColorCode {
        fn new(foreground: Color, background: Color) -> ColorCode {
            ColorCode((background as u8) << 4 | (foreground as u8))
        }
    }

### 字符缓冲区

    // in src/vga_buffer.rs

    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    #[repr(C)]
    struct ScreenChar {
        ascii_character: u8,
        color_code: ColorCode,
    }

    const BUFFER_HEIGHT: usize = 25;
    const BUFFER_WIDTH: usize = 80;

    #[repr(transparent)]
    struct Buffer {
        chars: [[ScreenChar; BUFFER_WIDTH]; BUFFER_HEIGHT],
    }

    pub struct Writer {
        column_position: usize,
        color_code: ColorCode,
        buffer: &'static mut Buffer,
    }

### 打印字符

    // in src/vga_buffer.rs

    impl Writer {
        pub fn write_byte(&mut self, byte: u8) {
            match byte {
                b'\n' => self.new_line(),
                byte => {
                    if self.column_position >= BUFFER_WIDTH {
                        self.new_line();
                    }

                    let row = BUFFER_HEIGHT - 1;
                    let col = self.column_position;

                    let color_code = self.color_code;
                    self.buffer.chars[row][col] = ScreenChar {
                        ascii_character: byte,
                        color_code,
                    };
                    self.column_position += 1;
                }
            }
        }

        fn new_line(&mut self) {/* TODO */}

        ub fn write_string(&mut self, s: &str) {
            for byte in s.bytes() {
                match byte {
                    // 可以是能打印的 ASCII 码字节，也可以是换行符
                    0x20...0x7e | b'\n' => self.write_byte(byte),
                    // 不包含在上述范围之内的字节
                    _ => self.write_byte(0xfe),
                }
            }
        }
    }
    pub fn print_something() {
        let mut writer = Writer {
            column_position: 0,
            color_code: ColorCode::new(Color::Yellow, Color::Black),
            buffer: unsafe { &mut *(0xb8000 as *mut Buffer) },
        };

        writer.write_byte(b'H');
        writer.write_string("ello ");
        writer.write_string("Wörld!");
    }

首先，我们把整数 0xb8000 强制转换为一个可变的裸指针（raw pointer）；之后，通过运算符*，我们将这个裸指针解引用；最后，我们再通过 &mut，再次获得它的可变借用。这些转换需要 unsafe 语句块（unsafe block），因为编译器并不能保证这个裸指针是有效的。

    // in src/main.rs
    #[no_mangle]
    pub extern "C" fn _start() -> ! {
        vga_buffer::print_something();
        loop {}
    }

### 易失操作
因为只向 buffer 写入数据，却不从它读出数据，编译器可能会把写入操作优化掉。
    # in Cargo.toml

    [dependencies]
    volatile = "0.2.6"

修改 buffer 类型定义：

    // in src/vga_buffer.rs

    use volatile::Volatile;

    struct Buffer {
        chars: [[Volatile<ScreenChar>; BUFFER_WIDTH]; BUFFER_HEIGHT],
    }
修改 write_byte：

    // in src/vga_buffer.rs

    impl Writer {
        pub fn write_byte(&mut self, byte: u8) {
            match byte {
                b'\n' => self.new_line(),
                byte => {
                    ...

                    self.buffer.chars[row][col].write(ScreenChar {
                        ascii_character: byte,
                        color_code: color_code,
                    });
                    ...
                }
            }
        }
        ...
    }

### 格式化宏

    // in src/vga_buffer.rs

    use core::fmt;

    impl fmt::Write for Writer {
        fn write_str(&mut self, s: &str) -> fmt::Result {
            self.write_string(s);
            Ok(())
        }
    }
    ...
    pub fn print_something() {
        use core::fmt::Write;
        let mut writer = Writer {
            column_position: 0,
            color_code: ColorCode::new(Color::Yellow, Color::Black),
            buffer: unsafe { &mut *(0xb8000 as *mut Buffer) },
        };

        writer.write_byte(b'H');
        writer.write_string("ello! ");
        write!(writer, "The numbers are {} and {}", 42, 1.0/3.0).unwrap();
    }

### 换行

    // in src/vga_buffer.rs

    impl Writer {
        fn new_line(&mut self) {
            for row in 1..BUFFER_HEIGHT {
                for col in 0..BUFFER_WIDTH {
                    let character = self.buffer.chars[row][col].read();
                    self.buffer.chars[row - 1][col].write(character);
                }
            }
            self.clear_row(BUFFER_HEIGHT - 1);
            self.column_position = 0;
        }

        fn clear_row(&mut self, row: usize) {
            let blank = ScreenChar {
                ascii_character: b' ',
                color_code: self.color_code,
            };
            for col in 0..BUFFER_WIDTH {
                self.buffer.chars[row][col].write(blank);
            }
        }
    }

## 3.全局接口

    // in src/vga_buffer.rs

    pub static WRITER: Writer = Writer {
        column_position: 0,
        color_code: ColorCode::new(Color::Yellow, Color::Black),
        buffer: unsafe { &mut *(0xb8000 as *mut Buffer) },
    };

一般的变量在运行时初始化，而静态变量在编译时初始化。

### 延迟初始化

    # in Cargo.toml

    [dependencies.lazy_static]
    version = "1.0"
    features = ["spin_no_std"]



    // in src/vga_buffer.rs

    use lazy_static::lazy_static;

    lazy_static! {
        pub static ref WRITER: Writer = Writer {
            column_position: 0,
            color_code: ColorCode::new(Color::Yellow, Color::Black),
            buffer: unsafe { &mut *(0xb8000 as *mut Buffer) },
        };
    }

### 自旋锁
    # in Cargo.toml
    [dependencies]
    spin = "0.4.9"

    // in src/vga_buffer.rs
    use spin::Mutex;
    ...
    lazy_static! {
        pub static ref WRITER: Mutex<Writer> = Mutex::new(Writer {
            column_position: 0,
            color_code: ColorCode::new(Color::Yellow, Color::Black),
            buffer: unsafe { &mut *(0xb8000 as *mut Buffer) },
        });
    }

    // in src/main.rs
    #[no_mangle]
    pub extern "C" fn _start() -> ! {
        use core::fmt::Write;
        vga_buffer::WRITER.lock().write_str("Hello again").unwrap();
        write!(vga_buffer::WRITER.lock(), ", some numbers: {} {}", 42, 1.337).unwrap();

        loop {}
    }

### print!宏

    // in src/vga_buffer.rs

    #[macro_export]
    macro_rules! print {
        ($($arg:tt)*) => ($crate::vga_buffer::_print(format_args!($($arg)*)));
    }

    #[macro_export]
    macro_rules! println {
        () => ($crate::print!("\n"));
        ($($arg:tt)*) => ($crate::print!("{}\n", format_args!($($arg)*)));
    }

    #[doc(hidden)]
    pub fn _print(args: fmt::Arguments) {
        use core::fmt::Write;
        WRITER.lock().write_fmt(args).unwrap();
    }

## 4.代码

    use core::fmt;
    use lazy_static::lazy_static;
    use spin::Mutex;
    use volatile::Volatile;

    lazy_static! {
        pub static ref WRITER: Mutex<Writer> = Mutex::new(Writer {
            column_position: 0,
            color_code: ColorCode::new(Color::Yellow, Color::Black),
            buffer: unsafe { &mut *(0xb8000 as *mut Buffer) },
        }); 
    }

    #[allow(dead_code)]
    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    #[repr(u8)]
    pub enum Color {
        Black = 0,
        Blue = 1,
        Green = 2,
        Cyan = 3,
        Red = 4,
        Magenta = 5,
        Brown = 6,
        LightGray = 7,
        DarkGray = 8,
        LightBlue = 9,
        LightGreen = 10,
        LightCyan = 11,
        LightRed = 12,
        Pink = 13,
        Yellow = 14,
        White = 15,
    }

    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    #[repr(transparent)]
    struct ColorCode(u8);

    impl ColorCode {
        fn new(foreground: Color, background: Color) -> ColorCode {
            ColorCode((background as u8) << 4 | (foreground as u8))
        }
    }

    #[derive(Debug, Clone, Copy, PartialEq, Eq)]
    #[repr(C)]
    struct ScreenChar {
        ascii_character: u8,
        color_code: ColorCode,
    }

    const BUFFER_HEIGHT: usize = 25;
    const BUFFER_WIDTH: usize = 80;

    struct Buffer {
        chars: [[Volatile<ScreenChar>; BUFFER_WIDTH]; BUFFER_HEIGHT],
    }


    pub struct Writer {
        column_position: usize,
        color_code: ColorCode,
        buffer: &'static mut Buffer,
    }

    impl Writer {
        pub fn write_byte(&mut self, byte: u8) {
            match byte {
                b'\n' => self.new_line(),
                byte => {
                    if self.column_position >= BUFFER_WIDTH {
                        self.new_line();
                    }

                    let row = BUFFER_HEIGHT - 1;
                    let col = self.column_position;

                    let color_code = self.color_code;
                    self.buffer.chars[row][col].write(ScreenChar {
                        ascii_character: byte,
                        color_code,
                    });
                    self.column_position += 1;
                }
            }
        }

        fn write_string(&mut self, s: &str) {
            for byte in s.bytes() {
                match byte {
                    0x20..=0x7e | b'\n' => self.write_byte(byte),

                    _ => self.write_byte(0xfe),
                }
            }
        }

        fn new_line(&mut self) {
            for row in 1..BUFFER_HEIGHT {
                for col in 0..BUFFER_WIDTH {
                    let character = self.buffer.chars[row][col].read();
                    self.buffer.chars[row - 1][col].write(character);
                }
            }
            self.clear_row(BUFFER_HEIGHT - 1);
            self.column_position = 0;
        }

        fn clear_row(&mut self, row: usize) {
            let blank = ScreenChar {
                ascii_character: b' ',
                color_code: self.color_code,
            };
            for col in 0..BUFFER_WIDTH {
                self.buffer.chars[row][col].write(blank);
            }
        }
    }

    impl fmt::Write for Writer {
        fn write_str(&mut self, s: &str) -> fmt::Result {
            self.write_string(s);
            Ok(())
        }
    }
    #[macro_export]
    macro_rules! print {
        ($($arg:tt)*) => ($crate::vga_buffer::_print(format_args!($($arg)*)));
    }

    #[macro_export]
    macro_rules! println {
        () => ($crate::print!("\n"));
        ($($arg:tt)*) => ($crate::print!("{}\n", format_args!($($arg)*)));
    }

    #[doc(hidden)]
    pub fn _print(args: fmt::Arguments) {
        use core::fmt::Write;
        WRITER.lock().write_fmt(args).unwrap();
    }
