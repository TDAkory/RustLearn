# Iterators and Closures

* Closures, a function-like construct you can store in a variable
* Iterators, a way of processing a series of elements
* How to use closures and iterators to improve the I/O project in Chapter 12
* The performance of closures and iterators (Spoiler alert: they’re faster than you might think!)

## closures-anonymous-functions-that-capture-their-environment

```rust
#[derive(Debug, PartialEq, Copy, Clone)]
enum ShirtColor {
    Red,
    Blue,
}

struct Inventory {
    shirts: Vec<ShirtColor>,
}

impl Inventory {
    fn giveaway(&self, user_preference: Option<ShirtColor>) -> ShirtColor {
        // specify the closure expression || self.most_stocked() as the argument to unwrap_or_else.
        // if the closure had parameters, they would appear between the two vertical bars
        user_preference.unwrap_or_else(|| self.most_stocked())
    }

    fn most_stocked(&self) -> ShirtColor {
        let mut num_red = 0;
        let mut num_blue = 0;

        for color in &self.shirts {
            match color {
                ShirtColor::Red => num_red += 1,
                ShirtColor::Blue => num_blue += 1,
            }
        }
        if num_red > num_blue {
            ShirtColor::Red
        } else {
            ShirtColor::Blue
        }
    }
}

fn main() {
    let store = Inventory {
        shirts: vec![ShirtColor::Blue, ShirtColor::Red, ShirtColor::Blue],
    };

    let user_pref1 = Some(ShirtColor::Red);
    let giveaway1 = store.giveaway(user_pref1);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref1, giveaway1
    );

    let user_pref2 = None;
    let giveaway2 = store.giveaway(user_pref2);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref2, giveaway2
    );
}
```

* closure通常是不向库以外的用户暴露的，因此不需要像fn一样，强制要求指明返回类型（在fn中，返回类型也是函数签名的必要组成部分）。
* closure通常存在于一个相对短小紧凑的上下文中，在这种场景下，compile可以推导入参和返回值的类型。
* 当然closure也可以添加对入参和返回值的类型说明。

```rust
    let expensive_closure = |num: u32| -> u32 {
        thread::sleep(Duration::from_secs(2));
        num
    };
```

* 事实上，closure本身就和fn非常相似，区别在于closure有很多可以省略不写的元素来简化

```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

* 当未指明详细类型时，编译器会在closure的**首次**调用点，根据入参完成类型推导。
  * 在此之后，使用其他类型调用closure，会返回编译期错误。

* closure像fn一样，可以通过`borrowing immutably`, `borrowing mutably`,  `taking ownership`三种方式获取入参，closure会根据其对参数执行的操作来抉择使用何种方式。
  * 可以用`move`关键字强制closure获取所有权，常用于传参进入线程

```rust
use std::thread;

fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    thread::spawn(move || println!("From thread: {:?}", list))
        .join()
        .unwrap();
}
```

* 闭包的捕获行为是多样的：捕获并抛出一个值、修改捕获的值、既不移动也不改变该值、或者从一开始就从环境中不捕获任何内容。
  * The way a closure captures and handles values from the environment affects which traits the closure implements, and traits are how functions and structs can specify what kinds of closures they can use. Closures will automatically implement one, two, or all three of these Fn traits, in an additive fashion, depending on how the closure’s body handles the values:
    * `FnOnce` applies to closures that can be called once. All closures implement at least this trait, because all closures can be called. A closure that moves captured values out of its body will only implement FnOnce and none of the other Fn traits, because it can only be called once.
    * `FnMut` applies to closures that don’t move captured values out of their body, but that might mutate the captured values. These closures can be called more than once.
    * `Fn` applies to closures that don’t move captured values out of their body and that don’t mutate captured values, as well as closures that capture nothing from their environment. These closures can be called more than once without mutating their environment, which is important in cases such as calling a closure multiple times concurrently.
  
```rust
// the definition of the unwrap_or_else method on Option<T>
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

## processing-a-series-of-items-with-iterators

* In Rust, iterators are lazy, meaning they have no effect until you call methods that consume the iterator to use it up.
* All iterators implement a trait named Iterator that is defined in the standard library. The definition of the trait looks like this:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

* iterator 本身需要是 mutable 的，但其获取的 Item 是 immutable references
  * `iter`、`into_iter`、`iter_mut`
  
* iterator还实现了很多方法，这些方法通常会调用next方法，我们称之为 consuming adaptors

```rust
    #[test]
    fn iterator_sum() {
        let v1 = vec![1, 2, 3];

        let v1_iter = v1.iter();

        let total: i32 = v1_iter.sum();

        assert_eq!(total, 6);
    }
```

* 此外，还有一类方法不会消费iter，反而产生其他新的iterator，我们称之为 Iterator adaptors 
  
```rust
    let v1: Vec<i32> = vec![1, 2, 3];

    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

    assert_eq!(v2, vec![2, 3, 4]);
```