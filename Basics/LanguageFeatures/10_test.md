# Test in Rust

## How

The bodies of test functions typically perform these three actions:

* Set up any needed data or state.
* Run the code you want to test.
* Assert the results are what you expect.

### The Anatomy of a Test Function

```rust
#[cfg(test)]
mod tests {
    #[test]     // indicates this is a test function
    fn it_works() {
        let ret = 2 * 2;
        assert_eq!(ret, 4);
    }
}
```

