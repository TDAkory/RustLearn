# Generic Types, Traits, Lifetime

- [Generic Types, Traits, Lifetime](#generic-types-traits-lifetime)
  - [Generic Types](#generic-types)
    - [const generics(Rust 1.51)](#const-genericsrust-151)
  - [Traits](#traits)
    - [Trait Bound Syntax](#trait-bound-syntax)
    - [Specifying Multiple Trait Bounds with the + Syntax](#specifying-multiple-trait-bounds-with-the--syntax)
    - [有条件的实现特征](#有条件的实现特征)
    - [特征用于指定函数返回值](#特征用于指定函数返回值)
    - [特征对象](#特征对象)
      - [特征对象的layout](#特征对象的layout)
      - [特征对象的限制](#特征对象的限制)
      - [关联类型](#关联类型)
      - [默认泛型类型参数](#默认泛型类型参数)
      - [同名方法的处理](#同名方法的处理)
      - [特征定义中，也可以增加特征约束](#特征定义中也可以增加特征约束)
      - [在外部类型上实现外部特征(newtype)](#在外部类型上实现外部特征newtype)
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

### 特征对象

> 其实就是C++继承，也有vtable

特征对象指向实现了特征的类型的实例，可以通过 `&` 引用或者 `Box<T>` 智能指针的方式来创建特征对象:

* `draw1` 函数的参数是 `Box<dyn Draw>` 形式的特征对象，该特征对象是通过 `Box::new(x)` 的方式创建的
* `draw2` 函数的参数是 `&dyn Draw` 形式的特征对象，该特征对象是通过 `&x` 的方式创建的
* `dyn` 关键字只用在特征对象的类型声明上，在创建时无需使用 `dyn`

```rust
trait Draw {
    fn draw(&self) -> String;
}

impl Draw for u8 {
    fn draw(&self) -> String {
        format!("u8: {}", *self)
    }
}

impl Draw for f64 {
    fn draw(&self) -> String {
        format!("f64: {}", *self)
    }
}

// 若 T 实现了 Draw 特征， 则调用该函数时传入的 Box<T> 可以被隐式转换成函数参数签名中的 Box<dyn Draw>
fn draw1(x: Box<dyn Draw>) {
    // 由于实现了 Deref 特征，Box 智能指针会自动解引用为它所包裹的值，然后调用该值对应的类型上定义的 `draw` 方法
    x.draw();
}

fn draw2(x: &dyn Draw) {
    x.draw();
}

fn main() {
    let x = 1.1f64;
    // do_something(&x);
    let y = 8u8;

    // x 和 y 的类型 T 都实现了 `Draw` 特征，因为 Box<T> 可以在函数调用时隐式地被转换为特征对象 Box<dyn Draw> 
    // 基于 x 的值创建一个 Box<f64> 类型的智能指针，指针指向的数据被放置在了堆上
    draw1(Box::new(x));
    // 基于 y 的值创建一个 Box<u8> 类型的智能指针
    draw1(Box::new(y));
    draw2(&x);
    draw2(&y);
}
```


#### 特征对象的layout
TODO 特征对象的layout

简而言之，当类型 Button 实现了特征 Draw 时，类型 Button 的实例对象 btn 可以当作特征 Draw 的特征对象类型来使用，btn 中保存了作为特征对象的数据指针（指向类型 Button 的实例数据）和行为指针（指向 vtable）。

一定要注意，此时的 btn 是 Draw 的特征对象的实例，而不再是具体类型 Button 的实例，而且 btn 的 vtable 只包含了实现自特征 Draw 的那些方法（比如 draw），因此 btn 只能调用实现于特征 Draw 的 draw 方法，而不能调用类型 Button 本身实现的方法和类型 Button 实现于其他特征的方法。也就是说，btn 是哪个特征对象的实例，它的 vtable 中就包含了该特征的方法。

#### 特征对象的限制

不是所有特征都能拥有特征对象，只有对象安全的特征才行。当一个特征的所有方法都有如下属性时，它的对象才是安全的：

* 方法的返回类型不能是 Self
* 方法没有任何泛型参数

> self是函数参数，指代当前实例，Self指代类型

#### 关联类型

关联类型是在特征定义的语句块中，申明一个自定义类型，这样就可以在特征的方法签名中使用该类型：

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

关联类型主要是为了使代码更加可读，比如下面的对比案例：

```rust
trait Container<A,B> {
    fn contains(&self,a: A,b: B) -> bool;
}

fn difference<A,B,C>(container: &C) -> i32
  where
    C : Container<A,B> {...}
```

```rust
trait Container{
    type A;
    type B;
    fn contains(&self, a: &Self::A, b: &Self::B) -> bool;
}

fn difference<C: Container>(container: &C) {}
```

#### 默认泛型类型参数

> 等价于：C++模板的默认类型参数
>
> Rust 并不支持创建自定义运算符，你也无法为所有运算符进行重载，目前来说，只有定义在 std::ops 中的运算符才能进行重载

```rust
// 以标准库std::ops::Add为例
trait Add<RHS=Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}
```

#### 同名方法的处理

不同特征拥有同名的方法是很正常的事情，除此之外，类型上，也有同名方法

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
```

针对接收&self的方法，默认时会调用类型上的定义，传参情况下可以进行显式调用

```rust
fn main() {
    let person = Human;
    Pilot::fly(&person); // 调用Pilot特征上的方法
    Wizard::fly(&person); // 调用Wizard特征上的方法
    person.fly(); // 调用Human类型自身的方法
}
```

针对不接收&self的关联函数，可以采用完全限定语法来指明调用类型`<Type as Trait>::function(receiver_if_method, next_arg, ...);`

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

#### 特征定义中，也可以增加特征约束

```rust
use std::fmt::Display;

// 想要实现 OutlinePrint 特征，首先你需要实现 Display 特征
trait OutlinePrint: Display {
    fn outline_print(&self) {
        let output = self.to_string();
        ...
    }
}
```

#### 在外部类型上实现外部特征(newtype)

孤儿规则，简单来说，就是特征或者类型必需至少有一个是本地的，才能在此类型上定义特征。

这里提供一个办法来绕过孤儿规则，那就是使用newtype 模式，简而言之：就是为一个元组结构体创建新类型。该元组结构体封装有一个字段，该字段就是希望实现特征的具体类型。

该封装类型是本地的，因此我们可以为此类型实现外部的特征。

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

## Validating References with Lifetimes

生命周期注解不会改变任何引用的存活时间。相反，它们描述多个引用的生命周期之间的关系，而不会影响这些生命周期。

在函数签名中使用生命周期注解，我们需要在函数名和参数列表之间的尖括号内声明泛型的 “生命周期” 参数，就像我们对泛型 “类型” 参数所做的那样。

在这个函数签名中指定生命周期参数时，并没有改变任何传入或返回值的生命周期，而是借助`borrow checker`来检测并拒绝所有不符合这个约束的值。函数也并不需要知道这个生命周期具体是多久。

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