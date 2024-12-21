# 数据类型

### 基本类型

- 有符号整型 `i8`, `i16`, `i32`, `i64`, `i128` and `isize` (指针大小)
- 无符号整型: `u8`, `u16`, `u32`, `u64`, `u128` and `usize` (指针大小)
- 浮点型: `f32`, `f64`
- 字符: 完整的Unicode 码， `'a'`, `'α'` and `'∞'` (4 字节)
- 布尔: `true` or `false`
- 单元:  `()`, 只有一个值 `()`

尽管单元类型是一个元组，但并不意味着它是复合类型，因为它不包含任何数据。

### 复合类型

- 数组 `[1, 2, 3]`
- 元组 `(1, true)`

你可以主动为变量声明类型。数值还可以通过`添加后缀`或者`设置默认值`来声明类型，整型默认是 `i32`, 浮点型默认是 `f64`。Rust 也可以从上下文推导变量的类型。

```rust
fn main() {
    // 类型声明
    let logical: bool = true;

    let a_float: f64 = 1.0; // 常规声明
    let an_integer = 5i32; // 通过后缀声明

    // 通过设置默认值
    let default_float = 3.0; // `f64`
    let default_integer = 7; // `i32`

    // 通过上下文推断.
    let mut inferred_type = 12; // 正常这里是i32
    inferred_type = 4294967296i64; // 但是这里通过后缀声明了i64

    // 可变变量可以被修改，但是必须修改成相同类型
    let mut mutable = 12; // Mutable `i32`
    mutable = 21;
    mutable = true; // Error expected integer, found `bool`

    // 遮蔽, 可以改变变量的类型
    let mutable = true;

    /* 复合类型 - Array and Tuple */

    // Array 类型签名 包含元素类型 T 和 数组长度length， [T; length].
    let my_array: [i32; 5] = [1, 2, 3, 4, 5];

    // Tuple 是一组包含不同类型数据的集合
    // 通过()来构造元组，元组内部元素类型需要严格对应
    let my_tuple = (5u32, 1u8, true, -5.04f32);
}
```

### 字面量和操作符

1. 上面罗列的基本类型都可以用作字面量。

2. 数值通过添加前缀（0b / 0o / 0x）, 来表示 二进制/八进制/16进制

3. 数值可以通过增加下划线来提高可读性。例如 `1_000_000` 、`1.000_01` , 下划线可以随便添加。

4. Rust 支持科学计数。 `1e4` = `10000` , `1.2e-4` = `0.00012`。数据类型是 `f64`

> Rust 的操作符及其优先级和C语言类似

```rust
fn main() {
    // Integer addition
    println!("1 + 2 = {}", 1u32 + 2);

    // Integer subtraction
    println!("1 - 2 = {}", 1i32 - 2);
    // TODO ^ Try changing `1i32` to `1u32` to see why the type is important

    // Scientific notation
    println!("1e4 is {}, -2.5e-3 is {}", 1e4, -2.5e-3);

    // Short-circuiting boolean logic
    println!("true AND false is {}", true && false);
    println!("true OR false is {}", true || false);
    println!("NOT true is {}", !true);

    // Bitwise operations
    println!("0011 AND 0101 is {:04b}", 0b0011u32 & 0b0101);
    println!("0011 OR 0101 is {:04b}", 0b0011u32 | 0b0101);
    println!("0011 XOR 0101 is {:04b}", 0b0011u32 ^ 0b0101);
    println!("1 << 5 is {}", 1u32 << 5);
    println!("0x80 >> 2 is 0x{:x}", 0x80u32 >> 2);

    // Use underscores to improve readability!
    println!("One million is written as {}", 1_000_000u32);
}
```

### 元组

元组是不同类型数据的集合，使用`()` 构造。元组的签名由元组每个位置的数据类型组成。例如：`let a: (i32, bool, f64) = (1,false,1.2)` 。元组可以作为函数的返回值，来返回多个数据。

```rust
use std::fmt::Display;

// 元组可以被用作函数的返回值
fn reverse(pair: (i32, bool)) -> (bool, i32) {
    // 可以通过解构赋值
    let (int_param, bool_param) = pair;

    (bool_param, int_param)
}

// The following struct is for the activity.
#[derive(Debug, PartialEq)]
struct Matrix(f32, f32, f32, f32);

impl Matrix {
    fn transpose(m: &mut Matrix) -> &mut Matrix {
        (m.1, m.2) = (m.2, m.1);
        m
    }
}

impl Display for Matrix {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        writeln!(f, "({}, {})", self.0, self.1)?;
        writeln!(f, "({}, {})", self.2, self.3)
    }
}

fn main() {
    // A tuple with a bunch of different types.
    let long_tuple = (
        1u8, 2u16, 3u32, 4u64, -1i8, -2i16, -3i32, -4i64, 0.1f32, 0.2f64, 'a', true,
    );

    // 可以通过下标访问元组的元素
    println!("Long tuple first value: {}", long_tuple.0);
    println!("Long tuple second value: {}", long_tuple.1);

    // 元组可以作为另一个元组的元素
    let tuple_of_tuples = ((1u8, 2u16, 2u32), (4u64, -1i8), -2i16);

    // 打印元组
    println!("tuple of tuples: {:?}", tuple_of_tuples);

    // 元组超过12个元素后无法打印。Debug只实现了12个元素内的元组
    // let too_long_tuple = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13);
    // println!("Too long tuple: {:?}", too_long_tuple);
    // TODO ^ Uncomment the above 2 lines to see the compiler error

    let pair = (1, true);
    println!("Pair is {:?}", pair);

    println!("The reversed pair is {:?}", reverse(pair));

    // 只有一个元素的元组需要加一个','。 否则不会被当成元组。
    println!("One element tuple: {:?}", (5u32,));
    println!("Just an integer: {:?}", (5u32));

    // 元组支持解构赋值.
    let tuple = (1, "hello", 4.5, true);

    let (a, b, c, d) = tuple;
    println!("{:?}, {:?}, {:?}, {:?}", a, b, c, d);

    let matrix = &mut Matrix(1.1, 1.2, 2.1, 2.2);
    println!("{}", matrix);
    println!("{}", Matrix::transpose(matrix));
}
```

### 数组和切片

1. 数组，具有相同数据类型的集合，存储在一段连续的内存中。编译时类型和长度固定。
   `let a: [i32; 5] = [1,2,3,4,5];`

2. 切片，与数组类似，但在编译时长度不定。
   `let b: &[i32] = &a[0..2];`

```rust
use std::mem;

// 数组可以传入切片, 可变借用可以更改数组元素
fn analyze_slice(slice: &mut [i32]) {
    slice[0] = 11;
    println!("First element of the slice: {}", slice[0]);
    println!("The slice has {} elements", slice.len());
}

fn main() {
    // 定长数组，类型可以推导，声明是多余的。
    let mut xs: [i32; 5] = [1, 2, 3, 4, 5];

    // 数组初始化
    let mut ys: [i32; 500] = [0; 500];

    // 通过索引访问元素
    println!("First element of the array: {}", xs[0]);
    println!("Second element of the array: {}", xs[1]);

    // `len` 返回数组长度
    println!("Number of elements in array: {}", xs.len());

    // 数组是栈分配的
    // size_of_val: Returns the size of the pointed-to value in bytes.
    println!("Array occupies {} bytes", mem::size_of_val(&xs));

    // 数组可以自动转为切片
    println!("Borrow the whole array as a slice.");
    analyze_slice(&mut xs);

    // 切片截取数组的一部分[starting_index..ending_index)
    // `starting_index` 切片的开始位置.
    // `ending_index` 切片结束的下一个位置（不包含）.
    println!("Borrow a section of the array as a slice.");
    analyze_slice(&mut ys[1..4]);

    // Example of empty slice `&[]`:
    let empty_array: [u32; 0] = [];
    assert_eq!(&empty_array, &[]);
    assert_eq!(&empty_array, &[][..]); // Same but more verbose

    // Arrays can be safely accessed using `.get`, which returns an
    // `Option`. This can be matched as shown below, or used with
    // `.expect()` if you would like the program to exit with a nice
    // message instead of happily continue.
    for i in 0..xs.len() + 1 {
        // Oops, one element too far!
        match xs.get(i) {
            Some(xval) => println!("{}: {}", i, xval),
            None => println!("Slow down! {} is too far!", i),
        }
    }

    // Out of bound indexing on array causes compile time error.
    //println!("{}", xs[5]);
    // Out of bound indexing on slice causes runtime error.
    //println!("{}", xs[..][5]);
}

```


