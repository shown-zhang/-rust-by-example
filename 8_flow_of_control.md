# 控制流

### if/else

if/else是一个表达式，因此可以参与赋值/返回值。所有分支必须返回相同的类型。

```rust
fn main() {
    let n = 5;

    if n < 0 {
        print!("{} is negative", n);
    } else if n > 0 {
        print!("{} is positive", n);
    } else {
        print!("{} is zero", n);
    }

    let big_n =
        if n < 10 && n > -10 {
            println!(", and is a small number, increase ten-fold");

            // This expression returns an `i32`.
            10 * n
        } else {
            println!(", and is a big number, halve the number");

            // This expression must return an `i32` as well.
            n / 2
            // TODO ^ Try suppressing this expression with a semicolon.
        };
    //   ^ Don't forget to put a semicolon here! All `let` bindings need it.

    println!("{} -> {}", n, big_n);
}
```

### loop

loop 是一个无限循环。可以使用 `break` 关键字跳出循环。`continue` 关键字跳过剩余的语句，直接跳到下一个循环。

```rust
fn main() {
    let mut count = 0u32;

    println!("Let's count until infinity!");

    // Infinite loop
    loop {
        count += 1;

        if count == 3 {
            println!("three");

            // Skip the rest of this iteration
            continue;
        }

        println!("{}", count);

        if count == 5 {
            println!("OK, that's enough");

            // Exit this loop
            break;
        }
    }
}
```

#### 循环嵌套 和 标签

如果两层循环，内层循环可以停止外层循环。需要使用标签来实现。

```rust
fn main() {
    let mut i = 0;
    let mut j = 0;
    'out: loop {
        i += 1;

        loop {
            j += 1;
            if j % 2 == 0 {
                break;
            }
            println!("{i},{j}");

            if i + j == 20 {
                break 'out;
            }
        }
    }
}
```

#### 循环返回值

loop 是表达式。可以使用`break`关键字返回值。 一般的， loop用于重试某个操作，直到成功后，返回一些数据。

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    assert_eq!(result, 20);
}
```

### while 循环

```rust
fn main() {
    // A counter variable
    let mut n = 1;

    // Loop while `n` is less than 101
    while n < 101 {
        if n % 15 == 0 {
            println!("fizzbuzz");
        } else if n % 3 == 0 {
            println!("fizz");
        } else if n % 5 == 0 {
            println!("buzz");
        } else {
            println!("{}", n);
        }

        // Increment counter
        n += 1;
    }
}
```

#### for循环

for循环会消耗一个迭代器。 

> 可以使用range来生成一个迭代器。

```rust
fn main() {
    // a..b => [a,b)
    // a..=b => [a,b]
    // 可以使用标签'out来终止外层循环
    'out: for i in 0..10 {
        println! {"i={i}"};
        for j in 1..=10 {
            if i < j {
                continue 'out;
            }

            if i + j == 10 {
                break 'out;
            }
            println!("j={j}");
        }
    }
}
```



> 有几种将集合collections转换为迭代器的方法：`into_iter` `iter` `iter_mut` 。他们对集合中的数据提供了不同的视图。



- `iter` 提供了对集合的不可变借用。因此，迭代结束后，集合依旧可以访问。
  
  ```rust
  fn main() {
      let a = vec![
          "xuan", "xue", "ting", "nan", "qian", "kun", "lu", "di", "wang", "ling",
      ];
  
      // 注意这里是对&str的不可变借用 &&str
      // 因此需要解引用
      for item in a.iter() {
          match *item {
              "xuan" => {
                  println!("zhang xuan yes");
              }
              "xue" => {
                  println!("zhang xue yes");
              }
  
              "ting" => {
                  println!("zhang ting yes");
              }
  
              "kun" => {
                  println!("li kun yes");
              }
  
              "wang" => {
                  println!("wang nan yes");
              }
              v => {
                  println!("{v} no");
              }
          }
      }
  
      println!("{:?}", a);
  }
  
  ```

- `into_iter` 会获取集合内元素所有权，随着迭代，元素被消耗。后续无法访问。
  
  ```rust
  fn main() {
      let a = vec!["a", "b", "c", "d"];
  
      // 注意这里已经获取了元素的所有权
      // 因此不需要解引用
      for item in a.into_iter() {
          match item {
              "a" => {
                  println!("a yes");
              }
              v => {
                  println!("{v} no");
              }
          }
      }
  
      // println!("{:?}", a); // Error：a 被消耗完毕， 无法访问
  }
  
  ```

- `iter_mut` 是对集合的可变借用，因此可以修改集合的值。集合不会被消耗.
  
  ```rust
  fn main() {
      let mut a = vec!["a", "b", "c", "d"];
  
      // 这里是可变借用，需要解引用，可以修改集合的值
      for item in a.iter_mut() {
          match *item {
              // 通过 '@' 绑定临时变量 v
              ref mut v @ "a" => {
                  *v = "a yes";
                  println!("a yes");
              }
              v => {
                  println!("{v} no");
              }
          }
      }
  
      println!("{:?}", a);
  }
  
  ```



### match

`match` 和其他语言的`switch` 类似。通过模式匹配计算第一个匹配分支。必须覆盖所有情况。

> match 是 表达式，可以用作 赋值/ 返回值。

```rust
fn main() {
    let number = 13;

    println!("Tell me about {}", number);
    // 只会寻找第一个匹配arm
    // 所有分支必须是同一个类型
    match number {
        // 匹配单个项
        1 => println!("One!"),
        // 匹配多个项
        2 | 3 | 5 | 7 | 11 | 13 => println!("This is a prime"),
        // 匹配一个范围
        13..=19 => println!("A teen"),
        // 处理其余分支，放在最后
        _ => println!("Ain't special"),
    }

    let boolean = true;
    // match 是一个表达式，可以赋值/返回值
    let binary = match boolean {
        // 分支必须涵盖所有情况
        false => 0,
        true => 1,
    };

    println!("{} -> {}", boolean, binary);
}

```



### match 解构

#### 元组解构

```rust
fn main() {
    let triple = (0, -2, 3);

    println!("Tell me about {:?}", triple);
    // match 分支 可以解构元组
    match triple {
        // Destructure the second and third elements
        // (0, y, ..) => println!("First is `0`, `y` is {:?}, and `z` is not care", y),
        (0, y, z) => println!("First is `0`, `y` is {:?}, and `z` is {:?}", y, z),
        (1, ..) => println!("First is `1` and the rest doesn't matter"),
        (.., 2) => println!("last is `2` and the rest doesn't matter"),
        (3, .., 4) => println!("First is `3`, last is `4`, and the rest doesn't matter"),
        // `..` 可以被用来忽略一些值
        _ => println!("It doesn't matter what they are"),
        // `_` means don't bind the value to a variable
    }
}

```



#### 数组/切片解构

```rust
fn main() {
    let array = [0, -2, 6, -1, 3, 4, 0, 7, 8];

    // Array or slice
    match &array[4..] {
        // 第一个元素是0， 绑定第二/三个元素，其余不关注
        [0, second, third, ..] => {
            println!("array[0] = 0, array[1] = {}, array[2] = {}", second, third)
        }

        // '_' 可以忽略单个元素
        [1, _, third, ..] => println!(
            "array[0] = 1, array[2] = {} and array[1] was ignored",
            third
        ),

        // '..' 可以忽略剩余元素
        [-1, second, ..] => println!(
            "array[0] = -1, array[1] = {} and all the other ones were ignored",
            second
        ),

        // 数组长度必须一致，否则永远不会执行
        // [-1, second] => ...

        // 通过 '@' 绑定到临时变量
        [first @ 3, second, tail @ ..] => println!(
            "first = {first}, array[1] = {} and the other elements were {:?}",
            second, tail
        ),

        // 组合这些模式
        [first, middle @ .., last] => println!(
            "array[0] = {}, middle = {:?}, array[2] = {}",
            first, middle, last
        ),

        _ => {
            println!("Fuck");
        }
    }
}

```

#### 枚举解构

```rust
// `allow` required to silence warnings because only
// one variant is used.
#[allow(dead_code)]
enum Color {
    // These 3 are specified solely by their name.
    Red,
    Blue,
    Green,
    // These likewise tie `u32` tuples to different names: color models.
    RGB(u32, u32, u32),
    HSV(u32, u32, u32),
    HSL(u32, u32, u32),
    CMY(u32, u32, u32),
    CMYK(u32, u32, u32, u32),
    Vect { x: u32, y: u32, z: u32 },
}

fn main() {
    let color = Color::Vect { x: 1, y: 2, z: 3 };

    println!("What color is it?");
    // An `enum` can be destructured using a `match`.
    match color {
        Color::Red => println!("The color is Red!"),
        Color::Blue => println!("The color is Blue!"),
        Color::Green => println!("The color is Green!"),
        // 枚举解构
        Color::RGB(r, g, b) => println!("Red: {}, green: {}, and blue: {}!", r, g, b),
        Color::HSV(h, s, v) => println!("Hue: {}, saturation: {}, value: {}!", h, s, v),
        Color::HSL(h, s, l) => println!("Hue: {}, saturation: {}, lightness: {}!", h, s, l),
        Color::CMY(c, m, y) => println!("Cyan: {}, magenta: {}, yellow: {}!", c, m, y),
        Color::CMYK(c, m, y, k) => println!(
            "Cyan: {}, magenta: {}, yellow: {}, key (black): {}!",
            c, m, y, k
        ),
        Color::Vect { x, y, z } => println!("x: {}, y: {}, z: {}", x, y, z),
        // _ => (), // Don't need another arm because all variants have been examined
    }
}

```



#### 指针解构

指针解构和解引用不同。 我们可以通过 `*` 解引用。而指针解构，则有三种方式：`&` `ref` `ref mut`

```rust
fn main() {
    // 这里的 '&' 是取地址符。
    // reference 拿到的是 4 的地址
    let reference = &4;

    match reference {
        // reference 是一个引用
        // 这里的模式匹配， '&' 就是解构
        // &4 = &val => 4 = val
        &val => println!("Got a value via destructuring: {:?}", val),
    }

    // 如果不想解构指针。可以在模式匹配前解引用
    // 就跟正常的值一样
    match *reference {
        val => println!("Got a value via dereferencing: {:?}", val),
    }

    // 正常赋值，这里不是引用
    let _not_a_reference = 3;

    // rust 提供了 'ref' / 'ref mut' 关键字获取值的地址
    let ref _is_a_reference = 3;

    // 如果我们为变量定义了值，而不是引用。
    // 在模式匹配的时候，可以通过 `ref` and `ref mut` 获取引用。
    let value = "sss".to_owned();
    // 通过 ref 获得值的引用
    let bbb = match value {
        ref r => &r[0..1],
    };

    println!("value = {value}, bbb = {}", bbb);

    let mut mut_value = 6;
    // 通过 `ref mut` 获得值的可变引用.
    match mut_value {
        ref mut m => {
            // Got a reference. Gotta dereference it before we can
            // add anything to it.
            *m += 10;
            println!("We added 10. `mut_value`: {:?}", m);
        }
    }

    println!("mut_value finally is {mut_value}");
}

```



#### 结构体解构

```rust
fn main() {
    struct Foo {
        x: (u32, u32),
        y: u32,
    }

    let foo = Foo { x: (1, 2), y: 3 };

    match foo {
        // 嵌套解构
        Foo { x: (1, b), y } => println!("First of x is 1, b = {},  y = {} ", b, y),

        // 可以对解构的元素重新命名
        // 结构体顺序无影响
        Foo { y: 2, x: i } => println!("y is 2, i = {:?}", i),

        // '..' 忽略部分元素
        Foo { y, .. } => println!("y = {}, we don't care about x", y),
        // match arm 结构体 必须包含所有元素
        //Foo { y } => println!("y = {}", y),
    }

    let faa = Foo { x: (1, 2), y: 3 };

    // 结构体可以直接通过 let 嵌套解构 ，绑定到变量
    let Foo { x: x0, y: y0 } = faa;
    println!("Outside: x0 = {x0:?}, y0 = {y0}");

    // 嵌套解构
    struct Bar {
        foo: Foo,
    }

    let bar = Bar { foo: faa };
    let Bar {
        foo: Foo {
            x: nested_x,
            y: nested_y,
        },
    } = bar;
    println!("Nested: nested_x = {nested_x:?}, nested_y = {nested_y:?}");
}

```



### match 分支守卫

通过分支守卫 可以进行过滤。守卫主要是if语句。添加守卫，就是对一个分支进行细分。

> 注：我们要保证match分支覆盖所有情况。但是守卫与分支覆盖的完整性无关。

```rust
#[allow(dead_code)]
enum Temperature {
    Celsius(i32),
    Fahrenheit(i32),
}

fn main() {
    let temperature = Temperature::Celsius(35);
    // ^ TODO try different values for `temperature`

    match temperature {
        Temperature::Celsius(t) if t > 30 => println!("{}C is above 30 Celsius", t),
        // The `if condition` part ^ is a guard
        Temperature::Celsius(t) => println!("{}C is equal to or below 30 Celsius", t),

        Temperature::Fahrenheit(t) if t > 86 => println!("{}F is above 86 Fahrenheit", t),
        Temperature::Fahrenheit(t) => println!("{}F is equal to or below 86 Fahrenheit", t),
    }
}
```



### @绑定

可以使用 `@` 将值绑定到临时变量上。

```rust
// A function `age` which returns a `u32`.
fn age() -> u32 {
    15
}

fn main() {
    println!("Tell me what type of person you are");

    match age() {
        0             => println!("I haven't celebrated my first birthday yet"),
        // Could `match` 1 ..= 12 directly but then what age
        // would the child be? Instead, bind to `n` for the
        // sequence of 1 ..= 12. Now the age can be reported.
        n @ 1  ..= 12 => println!("I'm a child of age {:?}", n),
        n @ 13 ..= 19 => println!("I'm a teen of age {:?}", n),
        // Nothing bound. Return the result.
        n             => println!("I'm an old person of age {:?}", n),
    }
}
```

也可以通过 `@` 解构

```rust
fn some_number() -> Option<u32> {
    Some(42)
}

fn main() {
    match some_number() {
        // Got `Some` variant, match if its value, bound to `n`,
        // is equal to 42.
        Some(n @ 42) => println!("The Answer: {}!", n),
        // Match any other number.
        Some(n)      => println!("Not interesting... {}", n),
        // Match anything else (`None` variant).
        _            => (),
    }
}
```



### if let

在上面探究 `match` 的时候，我们必须穷尽所有arm。至少得有两个arm。这样写起来很冗余。如果我们只想匹配其中一个arm， 可以使用`if let`

```rust
fn main() {
    let number = Some(7);
    let letter: Option<i32> = None;
    let emoticon: Option<i32> = None;

    // 枚举的解构赋值
    // 如果 let Some(i) = number 解构成功，执行 {} 中的语句
    if let Some(i) = number {
        println!("Matched {:?}!", i);
    }

    // 使用else表示 match失败的其他情况
    if let Some(i) = letter {
        println!("Matched {:?}!", i);
    } else {
        // Destructure failed. Change to the failure case.
        println!("Didn't match a number. Let's go with a letter!");
    }

    // Provide an altered failing condition.
    let i_like_letters = false;

    if let Some(i) = emoticon {
        println!("Matched {:?}!", i);
    // 这里解构失败，那就看else if 分支的条件是否满足
    // 不满足继续向下执行
    } else if i_like_letters {
        println!("Didn't match a number. Let's go with a letter!");
    } else {
        // The condition evaluated false. This branch is the default:
        println!("I don't like letters. Let's go with an emoticon :)!");
    }
}

```



此外， `if let` 可以用来匹配所有的枚举类型。并且，有一个优点是，`if let` 可以匹配没有参数的枚举值。

```rust
// This enum purposely neither implements nor derives PartialEq.
// That is why comparing Foo::Bar == a fails below.
#[derive(PartialEq)]
enum Foo {
    Bar,
}

fn main() {
    let a = Foo::Bar;

    // Variable a matches Foo::Bar
    // 因为 Foo::Bar 没有实际意义，这里不能判断相等
    // 可以使用 PartialEq 宏来实现比较
    // 或者使用 if let
    if Foo::Bar == a {
        println!("a is foobar");
    }

    if let a = Foo::Bar {
        println!("a is foobar");
    }
}

```



### let else

用于绑定一个option的值到一个变量。如果绑定失败，会执行else中的值。`let else` 更加简洁, 而且没有`if let` 或者 `match`作用域问题。明显地，我们可以使用 `if let` 或者 `match` 实现同样的效果。

```rust
fn get_value_form_str(s: &str) -> (u64, &str) {
    // split() 可以传递一个字符，也可以传一个闭包, 也可以是一个字符切片
    let mut s = s.split(|a| a == ' ');
    let (Some(count_str), Some(value)) = (s.next(), s.next()) else {
        panic!("split error");
    };

    let Ok(count) = u64::from_str_radix(count_str, 10) else {
        panic!("count_str error");
    };

    (count, value)
}

fn main() {
    assert_eq!(get_value_form_str("12 zhang"), (12, "zhang"));
}

```



### while let

循环判断变成了 let 绑定。写法简化

```rust
fn main() {
    // Make `optional` of type `Option<i32>`
    let mut optional = Some(0);
    
    // This reads: "while `let` destructures `optional` into
    // `Some(i)`, evaluate the block (`{}`). Else `break`.
    while let Some(i) = optional {
        if i > 9 {
            println!("Greater than 9, quit!");
            optional = None;
        } else {
            println!("`i` is `{:?}`. Try again.", i);
            optional = Some(i + 1);
        }
        // ^ Less rightward drift and doesn't require
        // explicitly handling the failing case.
    }
    // ^ `if let` had additional optional `else`/`else if`
    // clauses. `while let` does not have these.
}
```
