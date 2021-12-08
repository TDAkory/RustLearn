# Ownership

> Ownership is Rust’s most unique feature, and it enables Rust to make memory safety guarantees without needing a garbage collector.

## Ownership rules

- Each value in Rust has a variable that’s called its owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

### Scope

```rust
    {                      // s is not valid here, it’s not yet declared
        let s = "hello";   // s is valid from this point forward

        // do stuff with s
    }                      // this scope is now over, and s is no longer valid
```

- When s comes into scope, it is valid.
- It remains valid until it goes out of scope.

### String

```rust
let s = String::from("hello");  // create a String from a string literal using the from function

s.push_str(", world!"); // push_str() appends a literal to a String

println!("{}", s); // This will print `hello, world!`
```

### Memory & Allocation

With the String type, in order to support a mutable, growable piece of text, we need to allocate an amount of memory on the heap, unknown at compile time, to hold the contents. This means:

- The memory must be requested from the memory allocator at runtime.
- We need a way of returning this memory to the allocator when we’re done with our String.

Rust takes a different path: the memory is automatically returned once the variable that owns it goes out of scope. Rust calls `drop` automatically at the closing curly bracket.

```rust
{
    let s = String::from("hello"); // s is valid from this point forward
    // do stuff with s
}                                  // this scope is now over, and s is no longer valid
```

#### Ways Variables and Data Interact: Move

```rust
    let s1 = String::from("hello");
    let s2 = s1;
```

When we assign s1 to s2, the String data is copied, meaning we copy the pointer, the length, and the capacity that are on the stack. We do not copy the data on the heap that the pointer refers to.

```rust
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{}, world!", s1);     // ERROR: value borrowed here after move, in case of double-free
```

If you’ve heard the terms shallow copy and deep copy while working with other languages, the concept of copying the pointer, length, and capacity without copying the data probably sounds like making a shallow copy. **But because Rust also invalidates the first variable**, instead of being called a shallow copy

#### Ways Variables and Data Interact: Clone

```rust
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
```

When you see a call to clone, you know that some arbitrary code is being executed and that code may be expensive. It’s a visual indicator that something different is going on.

#### Stack Only Data: Copy

```rust
    let x = 5;
    let y = x;

    println!("x = {}, y = {}", x, y);
```

The reason is that types such as integers that have a known size at compile time are stored entirely on the stack, so copies of the actual values are quick to make. 

Rust has a special annotation called the `Copy trait` that we can place on types like integers that are stored on the stack 

### Ownership & Functions

The semantics for passing a value to a function are similar to those for assigning a value to a variable. Passing a variable to a function will move or copy

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function, but i32 is Copy, so it's okay to still use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

If we tried to use `s` after the call to `takes_ownership`, Rust would throw a compile-time error. 

### Return Value and Scope

Returning values can also transfer ownership.

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into takes_and_gives_back, which also moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its return value into the function that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and moves out to the calling function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into scope

    a_string  // a_string is returned and moves out to the calling function
}
```

The ownership of a variable follows the same pattern every time: assigning a value to another variable moves it. When a variable that includes data on the heap goes out of scope, the value will be cleaned up by drop unless the data has been moved to be owned by another variable.

## References and Borrowing

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

// a reference to an object as a parameter instead of taking ownership of the value
fn calculate_length(s: &String) -> usize {
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what it refers to, nothing happens.
```

**Just as variables are immutable by default, so are references. We’re not allowed to modify something we have a reference to.**

### Mutable Reference

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);     // create a mutable reference with &mut s
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

**you can have only one mutable reference to a particular piece of data at a time.**, in case of `data race`

- Multi mutable is not allowed
- immutable & mutable is not allowed
- multi immituable is OK

### Dangling Reference

In Rust, by contrast, the compiler guarantees that references will never be dangling references

```rust
fn dangle() -> &String { // dangle returns a reference to a String

    let s = String::from("hello"); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away. Danger!

fn no_dangle() -> String {
    let s = String::from("hello");

    s   // Ownership is moved out, and nothing is deallocated.
}
```

- At any given time, you can have either one mutable reference or any number of immutable references.
- References must always be valid.