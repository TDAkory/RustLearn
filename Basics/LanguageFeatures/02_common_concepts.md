# Common Concepts

- [Common Concepts](#common-concepts)
  - [Variables & Mutability](#variables--mutability)
    - [Variables & Constants](#variables--constants)
    - [Shadowing](#shadowing)
  - [Data Types](#data-types)
    - [Scalar Type](#scalar-type)
      - [Integer Overflow](#integer-overflow)
    - [Compound Type](#compound-type)
  - [Function](#function)
    - [Statements & Expressions](#statements--expressions)
    - [Functions with Return](#functions-with-return)
  - [Comments](#comments)
  - [Control Flow](#control-flow)
    - [if](#if)
    - [Repetition with loop](#repetition-with-loop)
      - [loop](#loop)
      - [Looping through a collection with for](#looping-through-a-collection-with-for)

## Variables & Mutability

**By default variables are immutable.** This is one of many nudges Rust gives you to write your code in a way that takes advantage of the safety and easy concurrency that Rust offers. 

```rust
fn main() {
    let mut x = 5;          // `mut` is important, otherwise compile errors
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

### Variables & Constants

there are a few differences between constants and variables:

- Not allowed to use mut with constants. Constants aren’t just immutable by default—they’re always immutable.
- Declare constants using the `const` keyword instead of the `let` keyword, and the type of the value must be annotated. 
- Constants are valid for the entire time a program runs, within the scope they were declared in. 

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

### Shadowing

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }

    println!("The value of x is: {}", x);
}
```

```shell
cargo run
   Compiling variables v0.1.0 (/Users/admin/WorkPlace/RustWorkshop/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 2.06s
     Running `target/debug/variables`
The value of x in the inner scope is: 12
The value of x is: 6
```

Shadowing is different from marking a variable as mut, because we’ll get a compile-time error if we accidentally try to reassign to this variable without using the let keyword. By using let, we can perform a few transformations on a value but have the variable be immutable after those transformations have been completed.

The other difference between mut and shadowing is that because we’re effectively creating a new variable when we use the let keyword again, we can change the type of the value but reuse the same name.

But: **not allowed to mutate a variable’s type**

```rust
let mut spaces = "   ";
spaces = spaces.len();  // error: expected `&str`, found `usize`
```

## Data Types

### Scalar Type

A scalar type represents a single value. Rust has four primary scalar types: `integers`, `floating-point numbers`, `Booleans`, and `characters`. 

```rust
8-bit	i8	u8
16-bit	i16	u16
32-bit	i32	u32
64-bit	i64	u64
128-bit	i128	u128
arch	isize	usize
```

the `isize` and `usize` types depend on the kind of computer your program is running on: 64 bits if you’re on a 64-bit architecture and 32 bits if you’re on a 32-bit architecture.

#### Integer Overflow

To explicitly handle the possibility of overflow, you can use these families of methods that the standard library provides on primitive numeric types:

- Wrap in all modes with the wrapping_* methods, such as wrapping_add
- Return the None value if there is overflow with the checked_* methods
- Return the value and a boolean indicating whether there was overflow with the overflowing_* methods
- Saturate at the value’s minimum or maximum values with saturating_* methods

`floating-point numbers`, which are numbers with decimal points. Rust’s floating-point types are f32 and f64, which are 32 bits and 64 bits in size.

### Compound Type

Compound types can group multiple values into one type. Rust has two primitive compound types: tuples and arrays.

```rust
// Tuple
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1); 

    // The variable tup binds to the entire tuple, because a tuple is considered a single compound element. To get the individual values out of a tuple, we can use pattern matching to destructure a tuple value
    let (x, y, z) = tup;
    
    println!("The value of y is : {}", y);

    // Or can access a tuple element directly by using a period(.)
    let five_h = tup.0;
    let six_f = tup.1;
    let one = tup.2;
}
```

```rust
// Array
fn main() {
    let a1 = [1, 2, 3, 4, 5];

    let a2: [i32; 5] = [1, 2, 3, 4, 5];

    let a3 = [3; 5]; // initial value ; length
}
```

## Function

Rust code uses snake case as the conventional style for function and variable names. In snake case, all letters are lowercase and underscores separate words.

```rust
// function with parameters
fn function(x: i32) {                       
    println!("The value of x is {}", x);
}

// separate the parameter declarations with commas
fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {}{}", value, unit_label);
}
```

### Statements & Expressions

```rust
fn main() {
    let x = (let y = 6);    // ERROR: Rust不支持这类写法的连续赋值，let是一个声明，没有返回值
}

fn main() {
    let y = {
        let x = 3;
        x + 1               // y==4, 注意这里没有分号
    };

    println!("The value of y is: {}", y);
}
```

### Functions with Return

Functions can return values to the code that calls them. We don’t name return values, but we do declare their type after an arrow (->). In Rust, the return value of the function is synonymous with the value of the final expression in the block of the body of a function. You can return early from a function by using the return keyword and specifying a value.

```rust
fn five() -> i32 {
    5
}

// another example
fn plus_one(x: i32) -> i32 {
    x + 1
}

fn main() {
    let x = five();

    println!("The value of x is {}", x);
}
```

## Comments

use `//`

## Control Flow

### if

```rust
// if 
fn main() {
    let number = 6;

    if number % 4 == 0 {                        // normal if 
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }

    let condition = true;
    let number = if condition {5} else {6};     // if in let statement
}

```

### Repetition with loop

Rust has three kinds of loops: `loop`, `while`, and `for`. Let’s try each one.

#### loop

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {                            // The outer loop has the label 'counting_up, and it will count up from 0 to 2
        println!("count = {}", count);
        let mut remaining = 10;

        loop {                                      // The inner loop without a label counts down from 10 to 9. 
            println!("remaining = {}", remaining);
            if remaining == 9 {
                break;                              // The first break that doesn’t specify a label will exit the inner loop
            }
            if count == 2 {
                break 'counting_up;                 //  The break 'counting_up; statement will exit the outer loop.
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {}", count);
}

// count = 0
// remaining = 10
// remaining = 9
// count = 1
// remaining = 10
// remaining = 9
// count = 2
// remaining = 10
// End count = 2
```

```rust
// loop and return value
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}

// result = 20
````

#### while

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```

#### Looping through a collection with for

```rust
// Ugly way to do it
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index += 1;
    }
}

// Much more better
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {}", element);
    }
}

```