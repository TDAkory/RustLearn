# Struct

- [Struct](#struct)
  - [Defining & Instantiating](#defining--instantiating)
    - [Using Typle Structs without Named Fields to Create Different Types](#using-typle-structs-without-named-fields-to-create-different-types)
    - [Unit-like Structs Without Any Fields](#unit-like-structs-without-any-fields)
  - [Example](#example)
    - [Debug](#debug)
  - [Method](#method)
    - [Defining Methods](#defining-methods)
    - [Methods with More Parameters](#methods-with-more-parameters)
    - [Assocaited Functions](#assocaited-functions)

## Defining & Instantiating

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

    let user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    let mut user1 = User {          // the entire instance must be mutable; Rust doesn’t allow us to mark only certain fields as mutable. 
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");

fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}

// Using the Field Init Shorthand when Variables and Fields Have the Same Name
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}

    // Creating From Other
    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };

    // struct update syntax **Note that the struct update syntax is like assignment with = because it moves the data**
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1     // syntax .. specifies that the remaining fields not explicitly set should have the same value as the fields in the given instance
    };
```

### Using Typle Structs without Named Fields to Create Different Types

```rust
// tuple structs behave like tuples: you can destructure them into their individual pieces, you can use a . followed by the index to access an individual value
    struct Color(i32, i32, i32);
    struct Point(i32, i32, i32);

    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
```

### Unit-like Structs Without Any Fields

Unit-like structs can be useful in situations in which you need to implement a trait on some type but don’t have any data that you want to store in the type itself.

```rust
    struct AlwaysEqual;

    let subject = AlwaysEqual;
```

## Example

```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}

// Use Tuple
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}

// Use Struct
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

### Debug

若想要输出Struct的内容，可以：

- 为Struct添加`#[derive(Debug)]`
- 使用`dbg!`宏输出

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:?}", rect1);
}
```

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```

## Method

### Defining Methods

```rust
// change the area function that has a Rectangle instance as a parameter and instead make an area method defined on the Rectangle struct
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {                // Everything within this impl block will be associated with the Rectangle type.
    fn area(&self) -> u32 {     // The &self is actually short for self: &Self
        self.width * self.height
    }

    fn width(&self) -> bool {   // can choose to give a method the same name as one of the struct’s fields
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );

    if rect1.width() {  // with parentheses, Rust knows we mean the method width. 
        println!("The rectangle has a nonzero width; it is {}", rect1.width);   // When we don’t use parentheses, Rust knows we mean the field width
    }
}
```

**Here’s how it works: when you call a method with object.something(), Rust automatically adds in &, &mut, or * so object matches the signature of the method.**

### Methods with More Parameters

```rust
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

### Assocaited Functions

Associated functions that aren’t methods are often used for constructors that will return a new instance of the struct. 

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle {
            width: size,
            height: size,
        }
    }
}
```

**Each struct is allowed to have multiple impl blocks.**