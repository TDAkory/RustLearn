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