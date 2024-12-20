# Comment & Print



### 1.1 Comments

- Regular comments, which will be ignored by __rust-compiler__
  `// single line commens`
  `/* multiple lines comments */`

- Doc comments
  `/// generate doc for the folling line`
  `//! generate doc for the file`



For example:

```rust
//! describe the file

/// describe this fn
fn main() {
    // describe the folling line, and will be ignored
    println!("Hello world");
}
```



### 1.2 Formatted print

> Printing is handled by a series of `macros` defined in `std:fmt` 



1. `format!` : write formatted text to `String`

2. `print!` : write formatted text to `io::stdout`

3. `println!` : write formatted text to `io::stdout`, end with a `\n`

4. `eprint!` : write formatted text to `io::stderr`

5. `eprintln!` : write formatted text to `io::stderr`, end with a `\n`



```rust
use std::fmt::{Debug, Display};

fn main() {
    // print common string
    println!("Hello, world!");

    // print string, '{}' will be repplaced by argument
    print!("hello, {}. ", "fq");

    // argument 13 will be convert to '13'
    println!("I am {} years old", 13);

    // position arguments
    println!("second arg is {1}, and first arg is {}", "fq", 31);

    // named arguments, position args must before named args
    println!("c={0}, a={a}, b={b}", 3, a = 1, b = 2);

    // output in different base
    println!("base 10: {}", 100);
    println!("base 2: {:b}", 100);
    println!("base 8: {:o}", 100);
    println!("base 16 {:x}", 100);

    // right justify, by padding extra space
    println!("{0:>10}", 1);
    println!("{0:>10}", 1111);

    // You can pad arguments with extra char
    println!("{0:0>10}", 1);
    println!("{0:a>10}", 1111);

    // you can pad argumenets with extra char on the right
    println!("{0:0<10}", 1);
    println!("{0:a<10}", 1111);

    // print struct
    #[derive(Debug)]
    struct Person {
        name: String,
    }

    // struct data must implement Display trait
    impl Display for Person {
        fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
            // use {{ to display '{'
            write!(f, "{{ name: {name} }}", name = self.name)
        }
    }

    // use Display trait
    println!(
        "person: {}",
        Person {
            name: String::from("zhang")
        }
    );

    // use Debug trait
    println!(
        "person: {:?}",
        Person {
            name: String::from("zhang")
        }
    )
}

```



#### Precision

```rust
fn main() {
    // for non-numeric type, it will be truncated down
    println!("a = {:.5}", "12345678"); // a = 12345
    // for integral type, it will be ignored
    println!("a = {:.5}", 5); // a = 5

    /*
        for float-point type
        1. use `.N`, N means precision
        2. use `.N$`, N means the argument's position
        3. use `.*`, first val is precision, second val is target;
    */
    println!("a = {:.5}", 1.23); // a = 1.23000

    println!("a = {:.1$}", 1.23, 5); // a = 1.23000

    println!("Hello {} is {:.*}", "x", 5, 0.01); // Hello x is 0.01000
    println!("Hello {} is {2:.*}", "x", 4, 0.01); // Hello x is 0.0100
}

```



### Display or Debug



> `fmt::Debug` hardly looks compact and clean, but you need not implement trait
> 
> `fmt::Display` may be cleaner than `fmt::Debug` but you need implement Display trait In most cases



```rust
//! pretty printing for Vector

use std::fmt::Display;

struct List(Vec<i32>);

impl Display for List {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "[")?;

        for item in self.0.iter() {
            write!(f, "{},", item)?; // std::fmt::Result
        }

        writeln!(f, "]")
    }
}
fn main() {
    let a = List(vec![1, 2, 3, 4, 5, 6]);

    println!("a = {a}");
}

```



### Activity

```rust
use std::fmt::Display;

struct Color(u8, u8, u8);

impl Display for Color {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        write!(f, "RGB ({}, {}, {}) ", self.0, self.1, self.2)?;

        let r = format!("{:x}", self.0);
        let g = format!("{:x}", self.1);
        let b = format!("{:x}", self.2);

        // pad 0
        write!(f, "0x{r:0>2}{g:0>2}{b:0>2}")
    }
}

fn main() {
    let a = Color(128, 255, 90);
    let b = Color(0, 3, 254);
    let c = Color(0, 0, 0);
    println!("{a}"); // RGB (128, 255, 90) 0x80ff5a
    println!("{b}"); // RGB (0, 3, 254) 0x0003fe
    println!("{c}"); // RGB (0, 0, 0) 0x000000
}

```


