# Generic Types, Traits, Lifetime

- [Generic Types, Traits, Lifetime](#generic-types-traits-lifetime)
	- [Generic Types](#generic-types)
		- [const generics(Rust 1.51)](#const-genericsrust-151)
	- [Traits](#traits)
		- [Trait Bound Syntax](#trait-bound-syntax)
		- [Specifying Multiple Trait Bounds with the + Syntax](#specifying-multiple-trait-bounds-with-the--syntax)
		- [有条件的实现特征](#有条件的实现特征)
		- [特征用于指定函数返回值](#特征用于指定函数返回值)
	- [Validating References with Lifetimes](#validating-references-with-lifetimes)
		- [Lifetime Annotations in Struct Definitions](#lifetime-annotations-in-struct-definitions)
		- [The Static Lifetime](#the-static-lifetime)


## Generic Types

> The generic Option<T> is replaced with the specific definitions created by the compiler. Because Rust compiles generic code into code that specifies the type in each instance, **we pay no runtime cost for using generics**.

```rust
fn largest<T>(list: &[T]) -> &T {}
```

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };
    println!("p.x = {}", p.x());

    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };
    let p3 = p1.mixup(p2);
    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

### const generics(Rust 1.51)

简而言之，是针对数值的泛型

```rust
fn display_array<T: std::fmt::Debug, const N: usize>(arr: [T; N]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(arr);

    let arr: [i32; 2] = [1, 2];
    display_array(arr);
}
```

const generics expression 

```rust
fn something<T>(val: T)
where
    Assert<{ core::mem::size_of::<T>() < 768 }>: IsTrue,
    //       ^-----------------------------^ 这里是一个 const 表达式，换成其它的 const 表达式也可以
{
    //
}
```

目前，仅允许使用以下形式的 const 参数实例化模板参数：

一个独立的 const 参数。
一个字面量（即整数、布尔值或字符）。
一个具体的常量表达式（用 {} 包围），不涉及任何通用参数。

```rust
fn foo<const N: usize>() {}

fn bar<T, const M: usize>() {
    foo::<M>(); // Okay: `M` is a const parameter
    foo::<2021>(); // Okay: `2021` is a literal
    foo::<{20 * 100 + 20 * 10 + 1}>(); // Okay: const expression contains no generic parameters
    
    foo::<{ M + 1 }>(); // Error: const expression contains the generic parameter `M`
    foo::<{ std::mem::size_of::<T>() }>(); // Error: const expression contains the generic parameter `T`
    
    let _: [u8; M]; // Okay: `M` is a const parameter
    let _: [u8; std::mem::size_of::<T>()]; // Error: const expression contains the generic parameter `T`
}
```

## Traits

> concept & require in C++20

> Traits are similar to a feature often called interfaces in other languages, although with some differences.

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}

// Traits as Parameters
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

### Trait Bound Syntax

```rust
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

// pub fn notify(item1: &impl Summary, item2: &impl Summary) {

// pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

### Specifying Multiple Trait Bounds with the + Syntax

```rust
pub fn notify(item: &(impl Summary + Display)) {}

pub fn notify<T: Summary + Display>(item: &T) {}

fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {}

fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{}
```

### 有条件的实现特征

```rust
impl<T: Display> ToString for T {
    // 未所有实现了Display的类型，实现ToString的trait
}
```

### 特征用于指定函数返回值

> 这种方式其实有弊端，只能返回某一种具体类型。当可选的返回多个类型时，编译会报错，因为实际的返回值类型，在编译到第一个返回分支时被确定了。
> 可以用`特征对象`来实现这个能力

```rust
fn returns_summarizable() -> impl Summary {
    ...
}
```

## Validating References with Lifetimes

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### Lifetime Annotations in Struct Definitions

We can define structs to hold references, but in that case we would need to add a lifetime annotation on every reference in the struct’s definition.

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

The compiler uses three rules to figure out the lifetimes of the references when there aren’t explicit annotations. The first rule applies to input lifetimes, and the second and third rules apply to output lifetimes. These rules apply to fn definitions as well as impl blocks.

* The first rule is that the compiler assigns a lifetime parameter to each parameter that’s a reference.
* The second rule is that, if there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters
* The third rule is that, if there are multiple input lifetime parameters, but one of them is &self or &mut self because this is a method, the lifetime of self is assigned to all output lifetime parameters.

### The Static Lifetime

```rust
let s: &'static str = "I have a static lifetime.";
```