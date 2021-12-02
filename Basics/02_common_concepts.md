# Common Concepts

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
