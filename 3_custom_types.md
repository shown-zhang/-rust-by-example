# 自定义类型



### 结构体

1. 元组结构体
   
   ```rust
   struct Position(x,y);
   ```
   
   

2. 经典的结构体
   
   ```rust
   struct Person {
       name: String,
       age: u8,
   }
   ```
   
   
   

3. 单元结构体
   
   ```rust
   struct Person;
   ```

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

// 单元结构体 没有任何字段
struct Unit;

// 元组结构体
struct Pair(i32, f32);

// 经典的结构体
#[derive(Debug)]
struct Point {
    x: f32,
    y: f32,
}

// 结构体可以嵌套
#[derive(Debug)]
struct Rectangle {
    // A rectangle can be specified by where the top left and bottom right
    // corners are in space.
    top_left: Point,
    bottom_right: Point,
}

impl Rectangle {
    fn rect_area(rect: &Rectangle) -> f32 {
        // 嵌套解构
        let Rectangle {
            top_left: Point { x: x1, y: y1 },
            bottom_right: Point { x: x2, y: y2 },
        } = rect;

        (x2 - x1) * (y2 - y1)
    }

    fn square(Point { x, y }: &Point, size: f32) -> Rectangle {
        let top_left = Point { x: *x, y: *y };
        let bottom_right = Point {
            x: x + size,
            y: y + size,
        };
        Rectangle {
            bottom_right,
            top_left,
        }
    }
}

fn main() {
    // Create struct with field init shorthand
    let name = String::from("Peter");
    let age = 27;
    // 字段简写
    let peter = Person { name, age };

    // Print debug struct
    println!("{:?}", peter);

    // Instantiate a `Point`
    let point: Point = Point { x: 5.2, y: 0.4 };
    let another_point: Point = Point { x: 10.3, y: 0.6 };

    // 通过 '.' 访问元素
    println!("point coordinates: ({}, {})", point.x, point.y);

    // 部分更新
    let bottom_right = Point {
        x: 10.3,
        ..another_point
    };

    println!("second point: ({}, {})", bottom_right.x, bottom_right.y);

    // let 解构赋值
    let Point {
        x: left_edge,
        y: top_edge,
    } = point;

    let _rectangle = Rectangle {
        // 实例化也是表达式
        top_left: Point {
            x: left_edge,
            y: top_edge,
        },
        bottom_right: bottom_right,
    };

    // Instantiate a unit struct
    let _unit = Unit;

    // Instantiate a tuple struct
    let pair = Pair(1, 0.1);

    // 元组结构体通过下标访问
    println!("pair contains {:?} and {:?}", pair.0, pair.1);

    // 解构元组结构体
    let Pair(integer, decimal) = pair;

    println!("pair contains {:?} and {:?}", integer, decimal);

    let lt = Point { x: 1., y: 2. };
    let br = Point { x: 4., y: 3. };

    println!("Rectangle::square {:#?}", Rectangle::square(&lt, 4.0));

    let rect = &Rectangle {
        top_left: lt,
        bottom_right: br,
    };

    println!("Rectangle::rect_area {}", Rectangle::rect_area(rect));
}

```



### 枚举

和结构体类似， 但枚举只能是其中之一。可以使用枚举在数组中存储不同类型的元素。

> 可以使用 `use` 获取枚举元素的作用域。



```rust
// Create an `enum` to classify a web event. Note how both
// names and type information together specify the variant:
// `PageLoad != PageUnload` and `KeyPress(char) != Paste(String)`.
// Each is different and independent.
enum WebEvent {
    // 类似于单元
    PageLoad,
    PageUnload,
    // 元组结构体
    KeyPress(char),
    // 元组结构体
    Paste(String),
    // 结构体
    Click { x: i64, y: i64 },
}

// A function which takes a `WebEvent` enum as an argument and
// returns nothing.
fn inspect(event: WebEvent) {
    match event {
        WebEvent::PageLoad => println!("page loaded"),
        WebEvent::PageUnload => println!("page unloaded"),
        // KeyPress分支，可以对元组内容解构
        WebEvent::KeyPress(c) => println!("pressed '{}'.", c),
        WebEvent::Paste(s) => println!("pasted \"{}\".", s),
        // Click分支，对结构体解构
        WebEvent::Click { x, y } => {
            println!("clicked at x={}, y={}.", x, y);
        }
    }
}

fn main() {
    let pressed = WebEvent::KeyPress('x');
    // `to_owned()` creates an owned `String` from a string slice.
    let pasted = WebEvent::Paste("my text".to_owned());
    let click = WebEvent::Click { x: 20, y: 80 };
    let load = WebEvent::PageLoad;
    let unload = WebEvent::PageUnload;

    inspect(pressed);
    inspect(pasted);
    inspect(click);
    inspect(load);
    inspect(unload);
}


```



#### 使用枚举实现链表

```rust
use crate::List::*;

enum List {
    Nil,
    Con(f32, Box<List>),
}

impl List {
    fn new() -> List {
        Nil
    }

    fn append(self, num: f32) -> List {
        Con(num, Box::new(self))
    }

    fn len(&self) -> usize {
        match self {
            Nil => 0,
            Con(_, con) => 1 + con.len(),
        }
    }

    fn stringify(&self) -> String {
        match self {
            Nil => String::from("Nil"),
            Con(num, con) => format!("{{num: {num}, con:{}}}", con.stringify()),
        }
    }
}

fn main() {
    // Create an empty linked list
    let mut list = List::new();

    // Prepend some elements
    list = list.append(1.);
    list = list.append(2.);
    list = list.append(3.);

    // Show the final state of the list
    println!("linked list has length: {}", list.len());
    println!("{}", list.stringify());
}

```



### 常量

rust 有两种方式表示常量

- `const` : 不可变常量

- `static` : 拥有`'static` 生命周期。可变。不安全。



```rust
// Globals are declared outside all other scopes.
static mut LANGUAGE: &str = "Rust";
const THRESHOLD: i32 = 10;

fn is_big(n: i32) -> bool {
    // Access constant in some function
    n > THRESHOLD
}

fn main() {
    let n = 16;

    // Access constant in the main thread
    unsafe {
        println!("This is {}", LANGUAGE);
        LANGUAGE = "Rust 1";
        println!("This is {}", LANGUAGE);
    }
    println!("The threshold is {}", THRESHOLD);
    println!("{} is {}", n, if is_big(n) { "big" } else { "small" });

    // Error! Cannot modify a `const`.
    // THRESHOLD = 5;
    // FIXME ^ Comment out this line
}
```


