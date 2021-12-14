# Collections

## vector

```rust
    let v: Vec<i32> = Vec::new();   // create a new, empty vector by `Vec::new`

    let v = vec![1, 2, 3];          // create a Vec<T> that has initial values by `vec!`

    let mut v = Vec::new();         // create a vector and then add elements to it by `push`
    v.push(5);
    v.push(6);

    {
        let v = vec![1, 2, 3, 4];   // a vector is freed when it goes out of scope
        // do stuff with v
    } // <- v goes out of scope and is freed here       When the vector gets dropped, all of its contents are also dropped

//////////////////////////////////////////////////

    let v = vec![1, 2, 3, 4, 5];
    let third: &i32 = &v[2];        // by using & and [], which gives us a reference
    println!("The third element is {}", third);

    match v.get(2) {                // by using the `get` method with the index passed as an argument, which gives us an Option<&T>
        Some(third) => println!("The third element is {}", third),
        None => println!("There is no third element."),
    }

//////////////////////////////////////////////////

    let v = vec![1, 2, 3, 4, 5];

    let does_not_exist = &v[100];       // panic
    let does_not_exist = v.get(100);    // returns None without panicking

//////////////////////////////////////////////////

    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];                              // immutable borrow occurs here

    v.push(6);                                      // mutable borrow occurs here

    println!("The first element is: {}", first);    // immutable borrow later used here

//////////////////////////////////////////////////

    let v = vec![100, 32, 57];
    for i in &v {               // iterate through all of the elements woth immutable references
        println!("{}", i);
    }

    let mut v = vec![100, 32, 57];
    for i in &mut v {           // iterate over mutable references to each element in a mutable vector
        *i += 50;               // use the dereference operator (*) to get to the value
    }

//////////////////////////////////////////////////
// Using an Enum to Store Multiple Types
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }

    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
```

## Storing UTF-8 Encoded Text with String

Rust has only one string type in the core language, which is the string slice `str` that is usually seen in its borrowed form `&str`.

The `String` type, which is provided by Rust’s standard library rather than coded into the core language, is a growable, mutable, owned, UTF-8 encoded string type.

```rust
let mut s = String::new();      // Create a new String
```