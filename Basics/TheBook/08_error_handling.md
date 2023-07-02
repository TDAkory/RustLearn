# Error Handling

- [Error Handling](#error-handling)
  - [Panic!](#panic)
    - [Backtrace](#backtrace)
  - [Recoverable Errors with Result](#recoverable-errors-with-result)
    - [Matching on Different Errors](#matching-on-different-errors)
      - [Alternatives to Using match with Result\<T, E\>](#alternatives-to-using-match-with-resultt-e)
    - [Shortcuts for Panic on Error: unwrap and expect](#shortcuts-for-panic-on-error-unwrap-and-expect)
    - [Propagating Errors](#propagating-errors)
      - [Shortcut: the ? Operator](#shortcut-the--operator)


## Panic!

> By default, when a panic occurs, the program starts unwinding, which means Rust walks back up the stack and cleans up the data from each function it encounters. However, this walking back and cleanup is a lot of work. Rust, therefore, allows you to choose the alternative of immediately aborting, which ends the program without cleaning up, by adding panic = 'abort' to the appropriate [profile] sections in your Cargo.toml file. 

```rust
fn main() {
    panic!("crash and burn");
}

// thread 'main' panicked at 'crash and burn', src/main.rs:2:5
```

### Backtrace

```shell
RUST_BACKTRACE=1 cargo run
```

## Recoverable Errors with Result

```rust
use std::fs::File;

enum Result<T, E> {
    Ok(T),
    Err(E),
}


fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```

### Matching on Different Errors

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error);
            }
        },
    };
}
```

#### Alternatives to Using match with Result<T, E>

By using `closures` and the `unwrap_or_else` method

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```

### Shortcuts for Panic on Error: unwrap and expect

The `unwrap` method is a shortcut method implemented just like the match expression we wrote above.  If the Result value is the `Ok` variant, `unwrap` will return the value inside the Ok. If the Result is the `Err` variant, `unwrap` will call the `panic!` macro for us. 

```rust
let greeting_file = File::open("hello.txt").unwrap();
```

Similarly, the `expect` method lets us also choose the panic! error message.

```rust
let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
```

### Propagating Errors

```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```

#### Shortcut: the ? Operator

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

The ? placed after a Result value is defined to work in almost the same way as the match expressions we defined to handle the Result values above.

There is a difference between what the match expression from Listing 9-6 does and what the ? operator does: error values that have the ? operator called on them go through the from function, defined in the From trait in the standard library, which is used to convert values from one type into another. When the ? operator calls the from function, the error type received is converted into the error type defined in the return type of the current function. This is useful when a function returns one error type to represent all the ways a function might fail, even if parts might fail for many different reasons.

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();

    File::open("hello.txt")?.read_to_string(&mut username)?;

    Ok(username)
}
```