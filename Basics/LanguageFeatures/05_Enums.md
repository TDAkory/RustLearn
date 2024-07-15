# Enums and Pattern Matching

- [Enums and Pattern Matching](#enums-and-pattern-matching)
  - [Enum](#enum)
    - [The `Option` Enum](#the-option-enum)
  - [The `match` Control Flow Operator](#the-match-control-flow-operator)
    - [Patterns that Bind to Values](#patterns-that-bind-to-values)
    - [Matching with `Option<T>`](#matching-with-optiont)
    - [Matches Are Exhaustive](#matches-are-exhaustive)
    - [Catch-all Patterns and the `_` Placeholder](#catch-all-patterns-and-the-_-placeholder)
  - [Concise Control Flow with `if let`](#concise-control-flow-with-if-let)

## Enum

```rust
enum IpAddrKind {   // IpAddrKind is now a custom data type that we can use elsewhere in our code.
    V4,
    V6,
}

let four = IpAddrKind::V4;
let six = IpAddrKind::V6;

struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
};

let loopback = IpAddr {
    kind: IpAddrKind::V6,
    address: String::from("::1"),
};
```

We can represent the same concept in a more concise way using just an enum, rather than an enum inside a struct, by putting data directly into each enum variant.

```rust
    enum IpAddr {
        V4(String),
        V6(String),
    }

    let home = IpAddr::V4(String::from("127.0.0.1"));   // IpAddr::V4() is a function call that takes a String argument and returns an instance of the IpAddr type.

    let loopback = IpAddr::V6(String::from("::1"));
```

There’s another advantage to using an enum rather than a struct: **each variant can have different types and amounts of associated data.**

```rust
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));
```

This enum has four variants with different types:

```rust
enum Message {
    Quit,                       // no data associated with it at all
    Move { x: i32, y: i32 },    // has named fields like a struct does
    Write(String),              // includes a single String
    ChangeColor(i32, i32, i32), // includes three i32 values
}
```

There is one more similarity between enums and structs: just as we’re able to define methods on structs using impl, **we’re also able to define methods on enums.**

```rust
    impl Message {
        fn call(&self) {
            // method body would be defined here
        }
    }

    let m = Message::Write(String::from("hello"));
    m.call();
```

### The `Option` Enum

Rust does not have nulls, but it does have an enum that can encode the concept of a value being present or absent.

```rust
enum Option<T> {    // T: generic type parameter
    None,
    Some(T),        // <T> means the Some variant of the Option enum can hold one piece of data of any type
}

    let some_number = Some(5);
    let some_string = Some("a string");

    let absent_number: Option<i32> = None;
```

why is having `Option<T>` any better than having null?

In short, because `Option<T>` and `T` (where `T` can be any type) are different types, the compiler won’t let us use an `Option<T>` value as if it were definitely a valid value. 

```rust
    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y;        // ERROR: no implementation for `i8 + Option<i8>`
```

 When we have a value of a type like i8 in Rust, the compiler will ensure that we always have a valid value. We can proceed confidently without having to check for null before using that value. Only when we have an Option<i8> (or whatever type of value we’re working with) do we have to worry about possibly not having a value, and the compiler will make sure we handle that case before using the value.

 In general, in order to use an Option<T> value, you want to have code that will handle each variant. You want some code that will run only when you have a Some(T) value, and this code is allowed to use the inner T. You want some other code to run if you have a None value, and that code doesn’t have a T value available. The match expression is a control flow construct that does just this when used with enums: it will run different code depending on which variant of the enum it has, and that code can use the data inside the matching value.

## The `match` Control Flow Operator

`match` allows you to compare a value against a series of patterns and then execute code based on which pattern matches. Patterns can be made up of `literal values`, `variable names`, `wildcards`, and many other things; The power of `match` comes from the expressiveness of the patterns and the fact that **the compiler confirms that all possible cases are handled.**

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,       // the => operator that separates the pattern and the code to run
        Coin::Nickel => 5,      // Each arm is separated from the next with a comma
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {        // use curly brackets for multi lines of code
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### Patterns that Bind to Values

```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),   // add a variable called state to the pattern that matches values of the variant Coin::Quarter
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {   // When a Coin::Quarter matches, the state variable will bind to the value of that quarter’s state. 
            println!("State quarter from {:?}!", state);    // Then we can use state in the code for that arm
            25
        }
    }
}
```

### Matching with `Option<T>`

```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1), // 2. Some(5) match Some(i) with i=5. Then, add 1 to the value of i and create a new Some value with our total 6 inside
        }
    }

    let five = Some(5);          
    let six = plus_one(five);   // 1. the variable x in the body of plus_one will have the value Some(5)
    let none = plus_one(None);
```

### Matches Are Exhaustive

```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            Some(i) => Some(i + 1),     // ERROR: pattern `None` not covered
        }
    }
```

**we must exhaust every last possibility in order for the code to be valid**

### Catch-all Patterns and the `_` Placeholder

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),    // Note that we have to put the catch-all arm last because the patterns are evaluated in order
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
```

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),  // we’re explicitly ignoring all other values in the last arm; we haven’t forgotten anything.
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}

    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => (),    // we’re telling Rust explicitly that we aren’t going to use any other value that doesn’t match a pattern in an earlier arm, and we don’t want to run any code in this case.
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
```

## Concise Control Flow with `if let`

The `if let` syntax lets you combine `if` and `let` into a less verbose way to handle values that match one pattern while ignoring the rest.

```rust
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {}", max),   // by binding the value to the variable max in the pattern
        _ => (),
    }

    let config_max = Some(3u8);
    if let Some(max) = config_max {     // The syntax if let takes a pattern and an expression separated by an equal sign
        println!("The maximum is configured to be {}", max);    // works the same way as a match
    }
```