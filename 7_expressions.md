# 表达式

Rust 程序由一组语句构成， 其中语句一般由 __变量绑定__ 和 __以分号结尾的表达式__ 构成。

```rust
fn main() {
    // variable binding
    let x = 5;

    // expression;
    x;
    x + 1;
    15;
}
```



一个由作用域块也是一个表达式。因此 块 也可以作为值赋值给变量。作用域块中的最后一个表达式（不含分号）就是这个块的返回值。如果最后一个表达式以分号结尾， 那么返回值是().

```rust
fn main() {
    let x = 5u32;

    let y = {
        let x_squared = x * x;
        let x_cube = x_squared * x;

        // This expression will be assigned to `y`
        x_cube + x_squared + x
    };

    let z = {
        // The semicolon suppresses this expression and `()` is assigned to `z`
        2 * x;
    };

    println!("x is {:?}", x);
    println!("y is {:?}", y);
    println!("z is {:?}", z);
}
```




