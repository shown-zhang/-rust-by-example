# 宏

虽然看起来很像， 但宏并不是函数，宏会展开成源代码，与其他部分一起编译。与c语言不同的是。rust 的宏被扩展成抽象语法树。而不是字符串预处理。

```rust
macro_rules! sayhello {
    // 无参数
    () => {
        // 将这里的代码展开
        println!("Hello boss");
        println!("Hello boss");
    };
}
fn main() {
    /*
        等价于：
        println!("Hello boss");
        println!("Hello boss");
    */
    sayhello!();
}

```



### 宏的语法

#### 指示符

宏的参数以美元符号$为前缀，并以**指示符**注释类型。

指示符包含以下几种：

1. `block`只匹配 block 表达式

2. `expr` 表达式

3. `ident` 函数名/变量名

4. `item` 一个项（item），如一个函数，结构体，模块等

5. `literal` 字面量

6. `pat` 一个模式（pattern），如match表达式的（pattern）

7. `path` 一个路径（path），如`foo`，`::std::mem::replace`，`transmute::<_, int>`，...。

8. `stmt` 一个语句（statement）

9. `tt` 词法树（token tree）

10. `ty` 一个类型（type），如i32,u32,String,Option等。

11. `vis` 一个可能为空的`Visibility`限定词，如`pub`

12. `meta` 一个元数据项；例如`#[...]`和`#![...]`属性，meta为[]内的值。



- `*`：用于匹配大于等于0个参数。
- `+`：用于匹配大于1参数。
- `?`：用于匹配0或1个参数。

```rust
macro_rules! create_function {
    // This macro takes an argument of designator `ident` and
    // creates a function named `$func_name`.
    // The `ident` designator is used for variable/function names.
    ($func_name:ident) => {
        fn $func_name() {
            // The `stringify!` macro converts an `ident` into a string.
            println!("You called {:?}()", stringify!($func_name));
        }
    };
}

// Create functions named `foo` and `bar` with the above macro.
create_function!(foo);
create_function!(bar);

macro_rules! print_result {
    // This macro takes an expression of type `expr` and prints
    // it as a string along with its result.
    // The `expr` designator is used for expressions.
    ($expression:expr) => {
        //stringify!($expression) 不会执行表达式，会把表达式原封不动的转成字符串
        println!("{:?} = {:?}", stringify!($expression), $expression);
    };
}

fn main() {
    foo();
    bar();

    print_result!(1u32 + 1);

    //复习一下 block块：{} 也是表达式，可以返回值
    print_result!({
        let x = 1u32;

        x * x + 2 * x - 1
    });
}

```

#### 重载

宏可以通过重载来处理不同参数组合。

```rust
// `test!` will compare `$left` and `$right`
// in different ways depending on how you invoke it:
macro_rules! test {
    // 参数需要使用 ',' 或者 ';' 分隔。
    ($left:expr; and $right:expr) => {
        println!(
            "{:?} and {:?} is {:?}",
            stringify!($left),
            stringify!($right),
            $left && $right
        )
    }; // 每个分支必须以分号结尾

    ($left:expr; or $right:expr) => {
        println!(
            "{:?} or {:?} is {:?}",
            stringify!($left),
            stringify!($right),
            $left || $right
        )
    };
}

fn main() {
    // 调用的时候需要和参数模版保持一致
    test!(1i32 + 1 == 2i32; and 2i32 * 2 == 4i32);
    test!(true; or false);
}

```



#### 递归

和正则表达式一样， `+` 表示匹配 **至少一次**，`*` 表示匹配0次或多次。其中`$(...),+` 用来表示剩余参数。

```rust
//! 返回表达式中的最大值

macro_rules! maxone {
    // 类似于递归
    // 这里是终止条件，只有一个值时，返回该值
    ($e:expr) => {
        $e
    };

    // 这里是循环结构
    // $($y: expr),+ 表示剩余参数，至少有一个
    // 剩余参数的类型都是表达式 $y: expr
    ($x: expr, $($y: expr),+) => {
        std::cmp::max($x, maxone!($($y),+))
    }
}

fn main() {
    println!("maxone: {}", maxone!(1));
    println!(
        "maxone: {}",
        maxone!(
            1,
            {
                let t = 2;
                t * 2
            },
            3
        )
    );
    println!("maxone: {}", maxone!(3, -1, 2));
}

```



### 简化代码

```rust
use std::ops::{Add, Mul, Sub};

macro_rules! assert_equal_len {
    // The `tt` (token tree) designator is used for
    // operators and tokens.
    ($a:expr, $b:expr, $func:ident, $op:tt) => {
        assert!(
            $a.len() == $b.len(),
            "{:?}: dimension mismatch: {:?} {:?} {:?}",
            stringify!($func),
            ($a.len(),),
            stringify!($op),
            ($b.len(),)
        );
    };
}

macro_rules! op {
    ($func:ident, $bound:ident, $op:tt, $method:ident) => {
        fn $func<T: $bound<T, Output = T> + Copy>(xs: &mut Vec<T>, ys: &Vec<T>) {
            assert_equal_len!(xs, ys, $func, $op);

            for (x, y) in xs.iter_mut().zip(ys.iter()) {
                // *x = $bound::$method(*x, *y);
                *x = x.$method(*y);
            }
        }
    };
}

// Implement `add_assign`, `mul_assign`, and `sub_assign` functions.
// 不需要重复写几乎相同的代码
op!(add_assign, Add, +=, add);
op!(mul_assign, Mul, *=, mul);
op!(sub_assign, Sub, -=, sub);

#[cfg(test)]
mod test {
    use std::iter;
    macro_rules! test {
        ($func:ident, $x:expr, $y:expr, $z:expr) => {
            #[test]
            fn $func() {
                for size in 0usize..10 {
                    let mut x: Vec<_> = iter::repeat($x).take(size).collect();
                    let y: Vec<_> = iter::repeat($y).take(size).collect();
                    let z: Vec<_> = iter::repeat($z).take(size).collect();

                    super::$func(&mut x, &y);

                    assert_eq!(x, z);
                }
            }
        };
    }

    // Test `add_assign`, `mul_assign`, and `sub_assign`.
    test!(add_assign, 1u32, 2u32, 3u32);
    test!(mul_assign, 2u32, 3u32, 6u32);
    test!(sub_assign, 3u32, 2u32, 1u32);
}

fn main() {
    let mut x = vec![1, 2, 3];
    let y = vec![2, 3, 4];

    add_assign(&mut x, &y);

    println!("{:?}", x);
}

```



### 嵌入在rust特定作用域中的迷你语言

比如我想执行一段代码，类似javascript中的eval语句。

```rust
macro_rules! eval {
    // 需要注意的是，这里永远是处理一个参数的场景
    (eval $e: expr;) => {
        println!("eval {} is {}", stringify!($e), $e);
    };

    // 这里递归处理多个参数
    (eval $e: expr; $(eval $other: expr); *;) => {
        eval!(eval $e;);
        eval!($(eval $other); *;);
    };
}

macro_rules! js_scope {
    // 注意{{}}, 这里会展开成一个代码块。
    // 代码块里实现了eval，这个rust里没有的关键字
    (eval $e: expr; $(eval $other: expr); *;) => {{
        eval!(eval $e; $(eval $other);*;);
    }};
}

fn main() {
    // 哇！ 我们在一个作用域里实现了新语法
    js_scope! {
        eval 1+1;

        eval 1+13;

        eval 1+11;
    };
}

```



> 上面宏的递归写法可以简化

```rust
macro_rules! js_scope {
    // 注意{{}}, 这里会展开成一个代码块。
    // 代码块里实现了eval，这个rust里没有的关键字
    ($(eval $e: expr); *) => {{
        // 这里对匹配逐个进行处理
        $(
            println!("eval {} is {}", stringify!($e), $e);
        )*

        println!("finished");
    }};
}

fn main() {
    // 哇！ 我们在一个作用域里实现了新语法
    js_scope! {
        eval 1+1;

        eval 1+13;

        eval 1+11
    };
}

```



### 宏的展开不会污染上下文

```rust
macro_rules! create_var {
    () => {
        let a = 1;
    };
}
fn main() {
    let a = 2;
    // 宏在上下文工作不受上下文的影响，也不会影响上下文。
    create_var!();
    println!("{}", a);
}

/*
宏展开后：
fn main() {
    let a = 2;
    let a = 1; // 宏展开不会影响上下文，所以 a = 2
    {
        ::std::io::_print(format_args!("{0}\n", a));
    };
}
*/
```


