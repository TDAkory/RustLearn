# [Comprehensive Rust](https://google.github.io/comprehensive-rust/welcome.html#welcome-to-comprehensive-rust-)

This is a four day Rust course developed by the Android team. The course covers the full spectrum of Rust, from basic syntax to advanced topics like generics and error handling. It also includes Android-specific content on the last day.

- [Comprehensive Rust](#comprehensive-rust)
  - [Day 1](#day-1)
    - [Wht Rust](#wht-rust)
    - [Basic Syntax](#basic-syntax)
      - [Scalar Types](#scalar-types)
      - [Compound Types](#compound-types)
      - [Reference](#reference)
      - [slices](#slices)
      - [Functions](#functions)
        - [Method](#method)
        - [Overloading](#overloading)
    - [Variables](#variables)
      - [Type Conversions](#type-conversions)
      - [Type Inference](#type-inference)
      - [const \& static](#const--static)
      - [Scopes and Shadowing](#scopes-and-shadowing)
    - [Memory Management](#memory-management)
    - [Ownership](#ownership)
      - [Move Semantics](#move-semantics)
      - [Moves in Function Calls](#moves-in-function-calls)
      - [Copying and Cloning](#copying-and-cloning)
      - [Borrowing](#borrowing)
        - [shared and unique borrows](#shared-and-unique-borrows)
      - [Lifetimes](#lifetimes)
        - [Lifetimes in Function Calls](#lifetimes-in-function-calls)
        - [Lifetimes in Data Structures](#lifetimes-in-data-structures)
  - [Day 2](#day-2)
    - [Structs](#structs)
      - [Tuple Structs](#tuple-structs)
      - [Field Shorthand Syntax](#field-shorthand-syntax)
    - [Enums](#enums)
      - [Variant Payloads](#variant-payloads)
      - [Enum Sizes](#enum-sizes)
    - [Methods](#methods)
      - [Method Receiver](#method-receiver)
    - [Pattern Matching](#pattern-matching)
      - [Destructuring Enums](#destructuring-enums)
      - [Destructuring Structs](#destructuring-structs)
      - [Destructuring Arrays](#destructuring-arrays)
      - [Match Guards](#match-guards)
    - [Control Flow](#control-flow)
      - [Blocks](#blocks)


## Day 1

### Wht Rust

Static memory management at compile time:

* No uninitialized variables.
* No memory leaks (mostly, see notes).
* No double-frees.
* No use-after-free.
* No NULL pointers.
* No forgotten locked mutexes.
* No data races between threads.
* No iterator invalidation.

No undefined behavior at runtime:

* Array access is bounds checked.
* Integer overflow is defined.

Modern Feature & Toolings

* Enums and pattern matching.
* Generics.
* No overhead FFI.
* Zero-cost abstractions.
* Great compiler errors.
* Built-in dependency manager.
* Built-in support for testing.
* Excellent Language Server Protocol support.

### Basic Syntax

Much of the Rust syntax will be familiar to you from C, C++ or Java:

* Blocks and scopes are delimited by curly braces.
* Line comments are started with //, block comments are delimited by /* ... */.
* Keywords like if and while work the same.
* Variable assignment is done with =, comparison is done with ==.

#### Scalar Types

* Signed integers: i8, i16, i32, i64, i128, isize
* Unsigned integers: u8, u16, u32, u64, u128, usize
* Floating point numbers: f32, f64
* Strings: &str
* Unicode scalar values: char
* Byte strings: &[u8]
* Booleans: bool

The types have widths as follows:

* iN, uN, and fN are N bits wide,
* isize and usize are the width of a pointer,
* char is 32 bit wide,
* bool is 8 bit wide.

#### Compound Types

* Arrays: [T; N] 
* Tuples: () (T, ) (T1, T2, ...)

#### Reference

```rust
fn main() {
    let mut x: i32 = 10;
    let ref_x: &mut i32 = &mut x;
    *ref_x = 20;
    println!("x: {x}");
}
```

* We must dereference ref_x when assigning to it, similar to C and C++ pointers.
* Rust will auto-dereference in some cases, in particular when invoking methods (try ref_x.count_ones()).
* References that are declared as mut can be bound to different values over their lifetime.
* **Be sure to note the difference between `let mut ref_x: &i32` and `let ref_x: &mut i32`. The first one represents a mutable reference which can be bound to different values, while the second represents a reference to a mutable value.**

```rust
fn main() {
    let ref_x : &i32;
    {
        let x: i32 = 10;
        ref_x = &x;
    }
    println!("ref_x: {ref_x}");
}
```

* A reference is said to “borrow” the value it refers to.
* Rust is tracking the lifetimes of all references to ensure they live long enough.

#### slices

```rust
fn main() {
    let a: [i32; 6] = [10, 20, 30, 40, 50, 60];
    println!("a: {a:?}");

    let s: &[i32] = &a[2..4];
    println!("s: {s:?}");
}
```

* Slices borrow data from the sliced type.
* We create a slice by borrowing a and specifying the starting and ending indexes in brackets.
* If the slice starts at index 0, Rust’s range syntax allows us to drop the starting index, meaning that `&a[0..a.len()]` and `&a[..a.len()]` are identical.
* The same is true for the last index, so `&a[2..a.len()]` and `&a[2..]` are identical.
* To easily create a slice of the full array, we can therefore use `&a[..]`.
* s is a reference to a slice of i32s. Notice that the type of s (&[i32]) no longer mentions the array length. This allows us to perform computation on slices of different sizes.
* Slices always borrow from another object. In this example, a has to remain ‘alive’ (in scope) for at least as long as our slice.
* For memory safety reasons you cannot do it through a after you created a slice, but you can read the data from both a and s safely. More details will be explained in the borrow checker section.

```rust
fn main() {
    let s1: &str = "World";
    println!("s1: {s1}");
    let mut s2: String = String::from("Hello ");
    println!("s2: {s2}");
    s2.push_str(s1);
    println!("s2: {s2}");
    
    let s3: &str = &s2[6..];
    println!("s3: {s3}");
}
```

* `&str` an immutable reference to a string slice.
* `String` a mutable string buffer.

#### Functions

```rust
fn main() {
    fizzbuzz_to(20);   // Defined below, no forward declaration needed
}

fn is_divisible_by(lhs: u32, rhs: u32) -> bool {
    if rhs == 0 {
        return false;  // Corner case, early return
    }
    lhs % rhs == 0     // The last expression in a block is the return value
}

fn fizzbuzz(n: u32) -> () {  // No return value means returning the unit type `()`
    match (is_divisible_by(n, 3), is_divisible_by(n, 5)) {
        (true,  true)  => println!("fizzbuzz"),
        (true,  false) => println!("fizz"),
        (false, true)  => println!("buzz"),
        (false, false) => println!("{n}"),
    }
}

fn fizzbuzz_to(n: u32) {  // `-> ()` is normally omitted
    for i in 1..=n {
        fizzbuzz(i);
    }
}
```

* We refer in main to a function written below. Neither forward declarations nor headers are necessary.

* Declaration parameters are followed by a type (the reverse of some programming languages), then a return type.

* The last expression in a function body (or any block) becomes the return value. Simply omit the ; at the end of the expression.

* Some functions have no return value, and return the ‘unit type’, (). The compiler will infer this if the -> () return type is omitted.

* The range expression in the for loop in fizzbuzz_to() contains =n, which causes it to include the upper bound.

* The match expression in fizzbuzz() is doing a lot of work. It is expanded below to show what is happening.

```rust
let by_3: bool = is_divisible_by(n, 3);
let by_5: bool = is_divisible_by(n, 5);
let by_35: (bool, bool) = (by_3, by_5);
match by_35 {
  // ...
```

##### Method 

Rust has methods, they are simply functions that are associated with a particular type. The first argument of a method is an instance of the type it is associated with:

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn inc_width(&mut self, delta: u32) {
        self.width += delta;
    }
}

fn main() {
    let mut rect = Rectangle { width: 10, height: 5 };
    println!("old area: {}", rect.area());
    rect.inc_width(5);
    println!("new area: {}", rect.area());
}
```

##### Overloading

**Overloading is not supported**

Each function has a single implementation:

* Always takes a fixed number of parameters.
* Always takes a single set of parameter types.
  
Default values are not supported:

* All call sites have the same number of arguments.
* Macros are sometimes used as an alternative.

**function parameters can be generic**

### Variables

#### Type Conversions

the` From<T>` and `Into<T>` traits to let us convert between them. The `From<T>` trait has a single from() method and similarly, the `Into<T>` trait has a single into() method. Implementing these traits is how a type expresses that it can be converted into another type.

#### Type Inference

> Rust will look at how the variable is used to determine the type

```rust
fn takes_u32(x: u32) {
    println!("u32: {x}");
}

fn takes_i8(y: i8) {
    println!("i8: {y}");
}

fn main() {
    let x = 10;
    let y = 20;

    takes_u32(x);
    takes_i8(y);
    // takes_u32(y);
}
```

It is very important to emphasize that variables declared like this are not of some sort of dynamic “any type” that can hold any data. The machine code generated by such declaration is identical to the explicit declaration of a type. The compiler does the job for us and helps us to write a more concise code.

The following code tells the compiler to copy into a certain generic container without the code ever explicitly specifying the contained type, using _ as a placeholder:

```rust
fn main() {
    let mut v = Vec::new();
    v.push((10, false));
    v.push((20, true));
    println!("v: {v:?}");

    let vv = v.iter().collect::<std::collections::HashSet<_>>();
    println!("vv: {vv:?}");
}
```

#### const & static

You can declare compile-time constants, According the the [Rust RFC Book](https://rust-lang.github.io/rfcs/0246-const-vs-static.html) these are inlined upon use.

```rust
const DIGEST_SIZE: usize = 3;
const ZERO: Option<u8> = Some(42);
```

You can also declare static variables, As noted in the [Rust RFC Book](https://rust-lang.github.io/rfcs/0246-const-vs-static.html), these are not inlined upon use and have an actual associated memory location. This is useful for unsafe and embedded code, and the variable lives through the entirety of the program execution.

#### Scopes and Shadowing

```rust
fn main() {
    let a = 10;
    println!("before: {a}");

    {
        let a = "hello";
        println!("inner scope: {a}");

        let a = true;
        println!("shadowed in inner scope: {a}");
    }

    println!("after: {a}");
}
```

* Definition: Shadowing is different from mutation, because after shadowing both variable’s memory locations exist at the same time. Both are available under the same name, depending where you use it in the code.
* A shadowing variable can have a different type.
* Shadowing looks obscure at first, but is convenient for holding on to values after .unwrap().

### Memory Management

**Full control and safety via compile time enforcement of correct memory management.**

Memory management in Rust is a mix:

* Safe and correct like Java, but without a garbage collector.
* Depending on which abstraction (or combination of abstractions) you choose, can be a single unique pointer, reference counted, or atomically reference counted.
* Scope-based like C++, but the compiler enforces full adherence.
* A Rust user can choose the right abstraction for the situation, some even have no cost at runtime like C.

### Ownership

All variable bindings have a scope where they are valid and it is an error to use a variable outside its scope:

```rust
struct Point(i32, i32);

fn main() {
    {
        let p = Point(3, 4);
        println!("x: {}", p.0);
    }
    println!("y: {}", p.1);
}
```

* At the end of the scope, the variable is dropped and the data is freed.
* A destructor can run here to free up resources.
* We say that the variable owns the value.

#### Move Semantics

```rust
fn main() {
    let s1: String = String::from("Hello!");
    let s2: String = s1;
    println!("s2: {s2}");
    // println!("s1: {s1}");
}
```

* The assignment of s1 to s2 transfers ownership.
* The data was moved from s1 and s1 is no longer accessible.
* When s1 goes out of scope, nothing happens: it has no ownership.
* When s2 goes out of scope, the string data is freed.
* There is always exactly one variable binding which owns a value.

#### Moves in Function Calls

**When you pass a value to a function, the value is assigned to the function parameter. This transfers ownership**

Rust makes it harder than C++ to inadvertently create copies by making move semantics the default, and by forcing programmers to make clones explicit.

#### Copying and Cloning

**While move semantics are the default, certain types are copied by default**

You can opt-in your own types to use copy semantics:

```rust
#[derive(Copy, Clonw, Debug)]
struct Point(i32, i32);

fn main() {
    let p1 = Point(3, 4);
    let p2 = p1;
    println!("p1: {p1:?}");
    println!("p2: {p2:?}");
}
```

Copying and cloning are not the same thing:

* Copying refers to bitwise copies of memory regions and does not work on arbitrary objects.
* Copying does not allow for custom logic (unlike copy constructors in C++).
* Cloning is a more general operation and also allows for custom behavior by implementing the Clone trait.
* Copying does not work on types that implement the Drop trait.

In the above example, try the following:

* Add a String field to struct Point. It will not compile because String is not a Copy type.
* Remove Copy from the derive attribute. The compiler error is now in the println! for p1.
* Show that it works if you clone p1 instead.

#### Borrowing

**just like pass by reference in C++**

```rust
#[derive(Debug)]
struct Point(i32, i32);

fn add(p1: &Point, p2: &Point) -> Point {
    Point(p1.0 + p2.0, p1.1 + p2.1)
}

fn main() {
    let p1 = Point(3, 4);
    let p2 = Point(10, 20);
    let p3 = add(&p1, &p2);
    println!("{p1:?} + {p2:?} = {p3:?}");
}
```

* Demonstrate that the return from add is cheap because the compiler can eliminate the copy operation.
* The Rust compiler can do return value optimization (RVO).
* In C++, copy elision has to be defined in the language specification because constructors can have side effects. In Rust, this is not an issue at all. If RVO did not happen, Rust will always performs a simple and efficient memcpy copy.

##### shared and unique borrows

* You can have one or more `&T `values at any given time, or
* You can have exactly one `&mut T` value.

```rust
fn main() {
    let mut a: i32 = 10;
    let b: &i32 = &a;

    {
        let c: &mut i32 = &mut a;
        *c = 20;
    }

    println!("a: {a}");
    println!("b: {b}");
}
```

#### Lifetimes

A borrowed value has a lifetime:

* The lifetime can be elided:` add(p1: &Point, p2: &Point) -> Point`.
* Lifetimes can also be explicit: `&'a Point, &'document str`.
* Read `&'a Point` as “a borrowed Point which is valid for at least the lifetime a”.
* Lifetimes are always inferred by the compiler: you cannot assign a lifetime yourself.
  * Lifetime annotations create constraints; the compiler verifies that there is a valid solution.

##### Lifetimes in Function Calls

In addition to borrowing its arguments, a function can return a borrowed value:

```rust
#[derive(Debug)]
struct Point(i32, i32);

fn left_most<'a>(p1: &'a Point, p2: &'a Point) -> &'a Point {
    if p1.0 < p2.0 { p1 } else { p2 }
}

fn main() {
    let p1: Point = Point(10, 10);
    let p2: Point = Point(20, 20);
    let p3: &Point = left_most(&p1, &p2);
    println!("left-most point: {:?}", p3);
}
```

* `'a` is a generic parameter, it is inferred by the compiler.
* Lifetimes start with `'` and `'a` is a typical default name.
* Read `&'a Point` as “a borrowed Point which is valid for at least the lifetime a”.
  * The at least part is important when parameters are in different scopes.

##### Lifetimes in Data Structures

If a data type stores borrowed data, it must be annotated with a lifetime:

```rust
#[derive(Debug)]
struct Highlight<'doc>(&'doc str);

fn erase(text: String) {
    println!("Bye {text}!");
}

fn main() {
    let text = String::from("The quick brown fox jumps over the lazy dog.");
    let fox = Highlight(&text[4..19]);
    let dog = Highlight(&text[35..43]);
    // erase(text);
    println!("{fox:?}");
    println!("{dog:?}");
}
```

## Day 2

### Structs

Like C and C++, Rust has support for custom structs:

```rust
struct Person {
    name: String,
    age: u8,
}

let mut peter = Person {
        name: String::from("Peter"),
        age: 27,
    };
```

* Like in C++, and unlike in C, no typedef is needed to define a type.
* Unlike in C++, there is no inheritance between structs.
* Methods are defined in an `impl` block
* This may be a good time to let people know there are different types of structs.
  * Zero-sized structs e.g., struct Foo; might be used when implementing a trait on some type but don’t have any data that you want to store in the value itself.
  * The next slide will introduce Tuple structs.

#### Tuple Structs

If the field names are unimportant, you can use a tuple struct

```rust
struct Point(i32, i32);

fn main() {
    let p = Point(17, 23);
    println!("({}, {})", p.0, p.1);
}
```

#### Field Shorthand Syntax

If you already have variables with the right names, then you can create the struct using a shorthand:

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

impl Person {
    fn new(name: String, age: u8) -> Person {
        Person { name, age }
    }
}

fn main() {
    let peter = Person::new(String::from("Peter"), 27);
    println!("{peter:?}");
}
```

### Enums

The enum keyword allows the creation of a type which has a few different variants

```rust
n generate_random_number() -> i32 {
    4  // Chosen by fair dice roll. Guaranteed to be random.
}

#[derive(Debug)]
enum CoinFlip {
    Heads,
    Tails,
}

fn flip_coin() -> CoinFlip {
    let random_number = generate_random_number();
    if random_number % 2 == 0 {
        return CoinFlip::Heads;
    } else {
        return CoinFlip::Tails;
    }
}

fn main() {
    println!("You got: {:?}", flip_coin());
}
```

#### Variant Payloads

You can define richer enums where the variants carry data. You can then use the match statement to extract the data from each variant

```rust
enum WebEvent {
    PageLoad,                 // Variant without payload
    KeyPress(char),           // Tuple struct variant
    Click { x: i64, y: i64 }, // Full struct variant
}

#[rustfmt::skip]
fn inspect(event: WebEvent) {
    match event {
        WebEvent::PageLoad       => println!("page loaded"),
        WebEvent::KeyPress(c)    => println!("pressed '{c}'"),
        WebEvent::Click { x, y } => println!("clicked at x={x}, y={y}"),
    }
}

fn main() {
    let load = WebEvent::PageLoad;
    let press = WebEvent::KeyPress('x');
    let click = WebEvent::Click { x: 20, y: 80 };

    inspect(load);
    inspect(press);
    inspect(click);
}
```

#### Enum Sizes

Rust enums are packed tightly, taking constraints due to alignment into account

```rust
use std::mem::{align_of, size_of};

macro_rules! dbg_size {
    ($t:ty) => {
        println!("{}: size {} bytes, align: {} bytes",
                 stringify!($t), size_of::<$t>(), align_of::<$t>());
    };
}

enum Foo {
    A,
    B,
}

#[repr(u32)]
enum Bar {
    A,  // 0
    B = 10000,
    C,  // 10001
}

fn main() {
    dbg_size!(Foo);
    dbg_size!(Bar);
    dbg_size!(bool);
    dbg_size!(Option<bool>);
    dbg_size!(&i32);
    dbg_size!(Option<&i32>);
}

// Foo: size 1 bytes, align: 1 bytes
// Bar: size 4 bytes, align: 4 bytes
// bool: size 1 bytes, align: 1 bytes
// Option<bool>: size 1 bytes, align: 1 bytes
// &i32: size 8 bytes, align: 8 bytes
// Option<&i32>: size 8 bytes, align: 8 bytes
```

* Internally Rust is using a field (discriminant) to keep track of the enum variant.
* Bar enum demonstrates that there is a way to control the discriminant value and type. If repr is removed, the discriminant type takes 2 bytes, becuase 10001 fits 2 bytes.
* As a niche optimization an enum discriminant is merged with the pointer so that Option<&Foo> is the same size as &Foo.
* Option<bool> is another example of tight packing.
* For some types, Rust guarantees that size_of::<T>() equals size_of::<Option<T>>().
* Zero-sized types allow for efficient implementation of HashSet using HashMap with () as the value.

### Methods

An `impl` block allows you to associate functions with types

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

impl Person {
    fn say_hello(&self) {
        println!("Hello, my name is {}", self.name);
    }
}

fn main() {
    let peter = Person {
        name: String::from("Peter"),
        age: 27,
    };
    peter.say_hello();
}
```

#### Method Receiver

The &self above indicates that the method borrows the object immutably. There are other possible receivers for a method:

* `&self`: borrows the object from the caller using a shared and immutable reference. The object can be used again afterwards.
* `&mut self`: borrows the object from the caller using a unique and mutable reference. The object can be used again afterwards.
* `self`: takes ownership of the object and moves it away from the caller. The method becomes the owner of the object. The object will be dropped (deallocated) when the method returns, unless its ownership is explicitly transmitted.
* `mut self`: same as above, but while the method owns the object, it can mutate it too. Complete ownership does not automatically mean mutability.
* `No receiver`: this becomes a static method on the struct. Typically used to create constructors which are called new by convention.

```rust
#[derive(Debug)]
struct Race {
    name: String,
    laps: Vec<i32>,
}

impl Race {
    fn new(name: &str) -> Race {  // No receiver, a static method
        Race { name: String::from(name), laps: Vec::new() }
    }

    fn add_lap(&mut self, lap: i32) {  // Exclusive borrowed read-write access to self
        self.laps.push(lap);
    }

    fn print_laps(&self) {  // Shared and read-only borrowed access to self
        println!("Recorded {} laps for {}:", self.laps.len(), self.name);
        for (idx, lap) in self.laps.iter().enumerate() {
            println!("Lap {idx}: {lap} sec");
        }
    }

    fn finish(self) {  // Exclusive ownership of self
        let total = self.laps.iter().sum::<i32>();
        println!("Race {} is finished, total lap time: {}", self.name, total);
    }
}

fn main() {
    let mut race = Race::new("Monaco Grand Prix");
    race.add_lap(70);
    race.add_lap(68);
    race.print_laps();
    race.add_lap(71);
    race.print_laps();
    race.finish();
    // race.add_lap(42);
}
```

### Pattern Matching

The match keyword let you match a value against one or more patterns. The comparisons are done from top to bottom and the first match wins.

```rust
fn main() {
    let input = 'x';

    match input {
        'q'                   => println!("Quitting"),
        'a' | 's' | 'w' | 'd' => println!("Moving around"),
        '0'..='9'             => println!("Number input"),
        _                     => println!("Something else"),
    }
}
```

* `|` as an or
* `..`can expand as much as it needs to be
* `1..=5` represents an inclusive range
* `_`is a wild card

#### Destructuring Enums

Patterns can also be used to bind variables to parts of your values. This is how you inspect the structure of your types. Let us start with a simple enum type, The if/else expression is returning an enum that is later unpacked with a match.

```rust
enum Result {
    Ok(i32),
    Err(String),
}

fn divide_in_two(n: i32) -> Result {
    if n % 2 == 0 {
        Result::Ok(n / 2)
    } else {
        Result::Err(format!("cannot divide {n} into two equal parts"))
    }
}

fn main() {
    let n = 100;
    match divide_in_two(n) {
        Result::Ok(half) => println!("{n} divided in two is {half}"),
        Result::Err(msg) => println!("sorry, an error happened: {msg}"),
    }
}
```

#### Destructuring Structs

```rust
struct Foo {
    x: (u32, u32),
    y: u32,
}

#[rustfmt::skip]
fn main() {
    let foo = Foo { x: (2, 2), y: 2 };
    match foo {
        Foo { x: (1, b), y } => println!("x.0 = 1, b = {b}, y = {y}"),
        Foo { y: 2, x: i }   => println!("y = 2, i = {i:?}"),
        Foo { y, .. }        => println!("y = {y}, other fields were ignored"),
    }
}
```

#### Destructuring Arrays

```rust
#[rustfmt::skip]
fn main() {
    let triple = [0, -2, 3];
    println!("Tell me about {triple:?}");
    match triple {
        [0, y, z] => println!("First is 0, y = {y}, and z = {z}"),
        [1, ..]   => println!("First is 1 and the rest were ignored"),
        _         => println!("All elements were ignored"),
    }
}
```

#### Match Guards

When matching, you can add a guard to a pattern. This is an arbitrary Boolean expression which will be executed if the pattern matches:

```rust
#[rustfmt::skip]
fn main() {
    let pair = (2, -2);
    println!("Tell me about {pair:?}");
    match pair {
        (x, y) if x == y     => println!("These are twins"),
        (x, y) if x + y == 0 => println!("Antimatter, kaboom!"),
        (x, _) if x % 2 == 1 => println!("The first one is odd"),
        _                    => println!("No correlation..."),
    }
}
```

* Match guards as a separate syntax feature are important and necessary.
* They are not the same as separate if expression inside of the match arm. An if expression inside of the branch block (after =>) happens after the match arm is selected. Failing the if condition inside of that block won’t result in other arms of the original match expression being considered.
* You can use the variables defined in the pattern in your if expression.
* The condition defined in the guard applies to every expression in a pattern with an |.

### Control Flow

#### Blocks

A block in Rust has a value and a type: the value is the last expression of the block. The same rule is used for functions: the value of the function body is the return value. However if the last expression ends with ;, then the resulting value and type is ().

```rust
fn main() {
    let x = {
        let y = 10;
        println!("y: {y}");
        let z = {
            let w = {
                3 + 4
            };
            println!("w: {w}");
            y * w
        };
        println!("z: {z}");
        z - y
    };
    println!("x: {x}");
}
```

