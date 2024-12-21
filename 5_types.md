# 类型

### 类型转换

Rust 基本类型之间没有隐式转换，但是可以使用关键字 `as` 进行显式转换。

1. 有符号整型 -> 无符号整型

2. 无符号整型 -> 有符号整型

3. 浮点型 -> 整型, 如果超出边界，那就是边界值

```rust
// Suppress all warnings from casts which overflow.
#![allow(overflowing_literals)]

fn main() {
    let decimal = 65.4321_f32;

    // 不能进行隐式转换， 使用as 显式转换
    let integer: u8 = decimal as u8;
    // FIXME ^ Comment out this line

    // 显式转换
    let integer = decimal as u8;
    let character = integer as char;

    // f32 不能直接转为 char。
    // f32 -> u8 -> char
    let character = decimal as u8 as char;
    // FIXME ^ Comment out this line

    println!("Casting: {} -> {} -> {}", decimal, integer, character);

    // 如果有符号整型转为无符号整型时。超出最大边界后，从0重新开始。
    // T::MAX + 1 = 0
    // T::MIN - 1 = T::MAX

    // 1000 already fits in a u16
    println!("1000 as a u16 is: {}", 1000 as u16);

    // 1000 - 256 - 256 - 256 = 232
    // 在底层，前8位最低有效位（LSB）被保留，
    // 而其余的最高有效位（MSB）被截断。
    // 1000的二进制：0000_0011_1110_1000
    // 232的二进制： 0000_0000_1110_1000
    // 对于正数， 就是求模
    println!("1000 as a u8 is : {}", 1000 as u8);
    // -1 + 256 = 255
    println!("  -1 as a u8 is : {}", (-1i8) as u8);

    // 对于正数， 就是求模
    println!("1000 mod 256 is : {}", 1000 % 256);

    // 如果无符号整型A的最大值小于有符号整型B。那就不用变
    println!(" 128 as a i16 is: {}", 128 as i16);

    // 否则，如果将无符号整型A转有符号整型B。
    // 先将无符号整型A转为无符号整型B。如果最高位是1，表示负数。
    // In boundary case 128 value in 8-bit two's complement representation is -128
    println!(" 128 as a i8 is : {}", 128 as i8);

    // repeating the example above
    // 1000 as u8 -> 232
    println!("1000 as a u8 is : {}", 1000 as u8);
    //  负数补码(取反 + 1) = ~|232| + 1 =  -24
    println!(" 232 as a i8 is : {}", 232 as i8);

    // 从Rust 1.45开始，‘ as ’关键字执行“饱和强制转换”
    // 当从float转换为int时。如果浮点值超过返回值的上界或小于下界，将等于交叉的边界。

    // 300.0 as u8 is 255
    println!(" 300.0 as u8 is : {}", 300.0_f32 as u8);
    // -100.0 as u8 is 0
    println!("-300.0 as i8 is : {}", -300.0_f32 as i8);
    // nan as u8 is 0
    println!("   nan as u8 is : {}", f32::NAN as u8);

    // This behavior incurs a small runtime cost and can be avoided
    // with unsafe methods, however the results might overflow and
    // return **unsound values**. Use these methods wisely:
    unsafe {
        // 300.0 as u8 is 44
        println!(" 300.0 as u8 is : {}", 300.0_f32.to_int_unchecked::<u8>());
        // -100.0 as u8 is 156
        println!(
            "-100.0 as u8 is : {}",
            (-100.0_f32).to_int_unchecked::<u8>()
        );
        // nan as u8 is 0
        println!("   nan as u8 is : {}", f32::NAN.to_int_unchecked::<u8>());
    }
}

```





### 字面量

数值类型字面量可以用后缀表示。如果不声明，整型默认是 `i32`, 浮点型默认是 `f64`

```rust
fn main() {
    // Suffixed literals, their types are known at initialization
    let x = 1u8;
    let y = 2u32;
    let z = 3f32;

    // Unsuffixed literals, their types depend on how they are used
    let i = 1; // i32
    let f = 1.0; // f64

    // `size_of_val` returns the size of a variable in bytes
    println!("size of `x` in bytes: {}", std::mem::size_of_val(&x)); // 1
    println!("size of `y` in bytes: {}", std::mem::size_of_val(&y)); // 4
    println!("size of `z` in bytes: {}", std::mem::size_of_val(&z)); // 4
    println!("size of `i` in bytes: {}", std::mem::size_of_val(&i)); // 4
    println!("size of `f` in bytes: {}", std::mem::size_of_val(&f)); // 8
}

```



### 类型推断

类型不仅可以在变量声明的时候获得。也可以通过上下文获得。

```rust
fn main() {
    let mut a = vec![];
    a.push(1); // 1 的类型是 i32， 所以a的类型是 Vec<i32>
}
```



### 类型别名

通过使用类型别名，可以避免类型名冲突； 简化代码；同时也可以将类型参数化，以便在后期改造。

```rust
// `NanoSecond`, `Inch`, and `U64` are new names for `u64`.
type NanoSecond = u64;
type Inch = u64;
type U64 = u64;

fn main() {
    // `NanoSecond` = `Inch` = `U64` = `u64`.
    let nanoseconds: NanoSecond = 5 as u64;
    let inches: Inch = 2 as U64;

    // Note that type aliases *don't* provide any extra type safety, because
    // aliases are *not* new types
    println!("{} nanoseconds + {} inches = {} unit?",
             nanoseconds,
             inches,
             nanoseconds + inches);
}
```


