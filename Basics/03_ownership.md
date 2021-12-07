# Ownership

> Ownership is Rust’s most unique feature, and it enables Rust to make memory safety guarantees without needing a garbage collector.

## ownership rules

- Each value in Rust has a variable that’s called its owner.
- There can only be one owner at a time.
- When the owner goes out of scope, the value will be dropped.

## Scope

```rust
    {                      // s is not valid here, it’s not yet declared
        let s = "hello";   // s is valid from this point forward

        // do stuff with s
    }                      // this scope is now over, and s is no longer valid
```

- When s comes into scope, it is valid.
- It remains valid until it goes out of scope.

## string