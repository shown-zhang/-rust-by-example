# 变量绑定

### 可变性

声明的变量默认是不可变的，无法修改对应的值。加上`mut` 关键字可以使其值可变。

```rust
fn main() {
    let mut a = 1;
    a += 1;
    println!("{a}");
}
```



### 作用域和遮蔽

所有的变量被强制放置在一个作用域内。作用域就是被 `{}` 包裹的一组语句。

```rust
fn main() {
    // This binding lives in the main function
    let long_lived_binding = 1;

    // This is a block, and has a smaller scope than the main function
    {
        // This binding only exists in this block
        let short_lived_binding = 2;

        println!("inner short: {}", short_lived_binding);
    }
    // End of the block

    // Error! `short_lived_binding` doesn't exist in this scope
    println!("outer short: {}", short_lived_binding);
    // FIXME ^ Comment out this line

    println!("outer long: {}", long_lived_binding);
}
```



Rust 允许变量遮蔽

```rust
fn main() {
    let shadowed_binding = 1;

    {
        println!("before being shadowed: {}", shadowed_binding);

        // This binding *shadows* the outer one
        let shadowed_binding = "abc";

        println!("shadowed in inner block: {}", shadowed_binding);
    }
    println!("outside inner block: {}", shadowed_binding);

    // This binding *shadows* the previous binding
    let shadowed_binding = 2;
    println!("shadowed in outer block: {}", shadowed_binding);
}

```

```rust
fn main() {
    // a 为 可变
    let mut a = 1;
    {
        let a = a;
        // Error, 此时 a 为 不可变
        a+=1    
    }

    // a 为可变
    a+=1;
}
```



### 先声明后初始化

变量可以先声明，只要在其使用前初始化即可。很少这样用。

```rust
fn main() {
    // Declare a variable binding
    let a_binding;

    {
        let x = 2;

        // Initialize the binding
        a_binding = x * x;
    }

    println!("a binding: {}", a_binding);

    let another_binding;

    // Error! Use of uninitialized binding
    println!("another binding: {}", another_binding);
    // FIXME ^ Comment out this line

    another_binding = 1;

    println!("another binding: {}", another_binding);
}
```




