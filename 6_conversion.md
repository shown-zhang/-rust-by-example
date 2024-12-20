# 复杂类型转换



### From & Into

From 和 Into 是一个意思。 如果A类型调用`B::From`可以转化为B类型，那么A类型也可以调用`A.into()`转化为B类型。因此只需要实现From即可，Into会自动实现。

> 注意，如果实现了Into， From不会自动实现

```rust
#[derive(Debug)]
struct Position {
    x: i32,
    y: i32,
}

// 实现From，可以将其他类型转为当前类型
impl From<&str> for Position {
    fn from(value: &str) -> Self {
        let distance = value.trim().parse().expect("解析失败");
        Position {
            x: distance,
            y: distance,
        }
    }
}

// 实现From<T>，可以将T类型转为当前类型Position
impl From<i32> for Position {
    fn from(value: i32) -> Self {
        Position { x: value, y: value }
    }
}

// 实现Into<T>，可以讲当前类型Position转为T
impl Into<i32> for Position {
    fn into(self) -> i32 {
        self.x
    }
}

fn main() {
    let a = String::from("   3 ");
    let b = 4;

    println!("{:?}", Position::from(&a[..]));
    println!("{:?}", Position::from(b));

    let c: Position = "   31 ".into();
    println!("{:?}", c);
}

```



### TryFrom & TryInto

和 From&Into类似，TryFrom & TryInto 是一般性的类型转换方法。不同的是，它的返回值是`Result<T, Error>`  。

```rust
use std::num::ParseIntError;

#[derive(Debug)]
struct EvenNumber {
    val: i32,
}

// 实现From，可以将其他类型转为当前类型
impl TryFrom<&str> for EvenNumber {
    type Error = ParseIntError;
    fn try_from(value: &str) -> Result<Self, Self::Error> {
        let val: i32 = value.trim().parse().unwrap();

        Ok(EvenNumber { val })
    }
}

impl TryFrom<i32> for EvenNumber {
    type Error = ();
    fn try_from(value: i32) -> Result<Self, Self::Error> {
        if value % 2 == 0 {
            Ok(EvenNumber { val: value })
        } else {
            Err(())
        }
    }
}

fn main() {
    println!("{:?}", EvenNumber::try_from("   32 "));

    match EvenNumber::try_from(12) {
        Ok(v) => println!("{:?}", v),
        Err(_) => println!("解析失败"),
    }

    let c = 11;
    // try_into 需要指定类型
    let c: Result<EvenNumber, _> = c.try_into();

    match c {
        Ok(v) => println!("{:?}", v),
        Err(_) => println!("解析失败"),
    };
}
```



### To and From Strings



#### Converting to string

想要将复杂类型转化为String。需要实现 `ToString` trait。 但是一般不这么做。我么可以实现`Display` trait, 它会自动实现`ToString` trait，并且可以支持格式化输出。

```rust
use std::fmt::Display;

struct Position {
    x: f32,
    y: f32,
}

impl Display for Position {
    fn fmt(&self, f: &mut std::fmt::Formatter<'_>) -> std::fmt::Result {
        writeln!(f, "{{ x: {}, y: {} }}", self.x, self.y)
    }
}

fn main() {
    let pos = Position { x: 12., y: -2.8 };

    println!("pos.to_string(): {}", pos.to_string());
    println!("pos: {}", pos);
}

```



#### Parsing a String

我们只要在类型<T>上实现 `FromStr` trait, 就可以使用 `parse()` 方法可以将字符串解析为我们想要的类型<T>。一般情况下，较为常用的是将字符串转为数值类型。标准库中有很多类型实现了`FromStr` trait

```rust
use std::{num::ParseFloatError, str::FromStr};

#[derive(Debug)]
struct Position {
    x: f32,
    y: f32,
}

impl FromStr for Position {
    type Err = ParseFloatError;
    fn from_str(s: &str) -> Result<Self, Self::Err> {
        match s.trim().parse() {
            Ok(v) => Ok(Position { x: v, y: v }),
            Err(v) => Err(v),
        }
    }
}

fn main() {
    let a = "1";
    // 指定类型，然后解析
    let b: i32 = a.parse().unwrap();
    println!("b = {}", b);

    // 也可以通过泛型
    let c = a.parse::<i32>().unwrap();
    println!("c = {}", c);

    // 复杂类型通过实现 FromStr trait 来进行解析
    let d = " 12a ";

    let d_: Result<Position, ParseFloatError> = d.parse();

    match d_ {
        Ok(v) => {
            println!("{:?}", v)
        }
        Err(e) => println!("{}", e),
    }
}

```


