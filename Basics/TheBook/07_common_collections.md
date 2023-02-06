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

//////////////////////////////////////////////////
    let data = "initial contents";
    //to_string method, which is available on any type that implements the Display trait
    let s = data.to_string();
    // the method also works on a literal directly:
    let s = "initial contents".to_string();
    // String::from to create a String from a string literal
    let s = String::from("initial contents");

//////////////////////////////////////////////////
    let mut s = String::from("foo");
    s.push_str("bar");  // append a string slice, foobar
    s.push('l');     // append a letter, foobarl

    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used

    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");
    // Use format!
    let s = format!("{}-{}-{}", s1, s2, s3);
```

## HashMap

```rust
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

//////////////////////////////////////////////////
    let teams = vec![String::from("Blue"), String::from("Yellow")];
    let initial_scores = vec![10, 50];

    let mut scores: HashMap<_, _> =
        teams.into_iter().zip(initial_scores.into_iter()).collect();

//////////////////////////////////////////////////
    // Accessing Values
    let team_name = String::from("Blue");
    let score = scores.get(&team_name);

    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }

 //////////////////////////////////////////////////
    // Overwriting Value
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);

    println!("{:?}", scores); // 25, overwritten

    // insert if key has no value
    scores.insert(String::from("Blue"), 10);
    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);

    println!("{:?}", scores);

    // Updating a Value Based on the Old Value
    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{:?}", map); // {"world": 2, "hello": 1, "wonderful": 1}
```

For types that implement the Copy trait, like i32, the values are copied into the hash map. For owned values like String, the values will be moved and the hash map will be the owner of those values

The or_insert method on Entry is defined to return a mutable reference to the value for the corresponding Entry key if that key exists, and if not, inserts the parameter as the new value for this key and returns a mutable reference to the new value. 