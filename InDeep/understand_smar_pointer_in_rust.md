# Understand Smart Pointer in Rust

- [Understand Smart Pointer in Rust](#understand-smart-pointer-in-rust)
  - [Box to Point to Data on the Heap](#box-to-point-to-data-on-the-heap)
    - [Enabling Recursive Types with Boxes](#enabling-recursive-types-with-boxes)
      - [More Information About the Cons List](#more-information-about-the-cons-list)
      - [Computing the Size of a Non-Recursive Type](#computing-the-size-of-a-non-recursive-type)
  - [Treating Smart Pointers Like Regular References with the `Deref` Trait](#treating-smart-pointers-like-regular-references-with-the-deref-trait)
    - [How Deref Coercion Interacts with Mutability](#how-deref-coercion-interacts-with-mutability)
  - [Running Code on Cleanup with the Drop Trait](#running-code-on-cleanup-with-the-drop-trait)
    - [Dropping a Value Early with std::mem::drop](#dropping-a-value-early-with-stdmemdrop)
  - [Rc, the Reference Counted Smart Pointer](#rc-the-reference-counted-smart-pointer)


The most common kind of pointer in Rust is a reference, indicated by the `&` symbol and borrow the value they point to. while references only borrow data, in many cases, smart pointers own the data they point to.

Though we didn’t call them as such at the time, we’ve already encountered a few smart pointers in this book, including `String` and `Vec<T>`.

smart pointers implement the `Deref` and `Drop` traits.

* `Box<T>` for allocating values on the heap
* `Rc<T>`, a reference counting type that enables multiple ownership
* `Ref<T>` and `RefMut<T>`, accessed through `RefCell<T>`, a type that enforces the borrowing rules at runtime instead of compile time

## Box<T> to Point to Data on the Heap

* Boxes don’t have performance overhead, other than storing their data on the heap instead of on the stack.
  * When you have a type whose size can’t be known at compile time and you want to use a value of that type in a context that requires an exact size
  * When you have a large amount of data and you want to transfer ownership but ensure the data won’t be copied when you do so
  * When you want to own a value and you care only that it’s a type that implements a particular trait rather than being of a specific type

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

### Enabling Recursive Types with Boxes

A value of recursive type can have another value of the same type as part of itself. Recursive types pose an issue because at compile time Rust needs to know how much space a type takes up. However, the nesting of values of recursive types could theoretically continue infinitely, so Rust can’t know how much space the value needs. Because boxes have a known size, we can enable recursive types by inserting a box in the recursive type definition.

#### More Information About the Cons List

A cons list is produced by recursively calling the cons function. The canonical name to denote the base case of the recursion is Nil. Note that this is not the same as the “null” or “nil” concept in Chapter 6, which is an invalid or absent value.

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
```

#### Computing the Size of a Non-Recursive Type

For enum, because only one variant will be used, the most space a Message value will need is the space it would take to store the largest of its variants.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

## Treating Smart Pointers Like Regular References with the `Deref` Trait

Implementing the Deref trait allows you to customize the behavior of the dereference operator `*`

The Deref trait, provided by the standard library, requires us to implement one method named `deref` that borrows `self` and returns a reference to the inner data.

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}

fn hello(name: &str) {
    println!("Hello, {name}!");
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);  // &MyBox --> &String --> &str
}
```

### How Deref Coercion Interacts with Mutability

Similar to how you use the `Deref` trait to override the `*` operator on immutable references, you can use the `DerefMut` trait to override the `*` operator on mutable references.

Rust does **deref coercion** when it finds types and trait implementations in three cases:

* From &T to &U when T: Deref<Target=U>
* From &mut T to &mut U when T: DerefMut<Target=U>
* From &mut T to &U when T: Deref<Target=U>

## Running Code on Cleanup with the Drop Trait

The second trait important to the smart pointer pattern is Drop, which lets you customize what happens when a value is about to go out of scope. 

The Drop trait requires you to implement one method named drop that takes a mutable reference to self.

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("CustomSmartPointers created.");
}

// CustomSmartPointers created.
// Dropping CustomSmartPointer with data `other stuff`!
// Dropping CustomSmartPointer with data `my stuff`!
```

### Dropping a Value Early with std::mem::drop

Rust doesn’t let you call the Drop trait’s drop method manually; instead you have to call the std::mem::drop function provided by the standard library if you want to force a value to be dropped before the end of its scope.

```rust
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}

/*
CustomSmartPointer created.
Dropping CustomSmartPointer with data `some data`!
CustomSmartPointer dropped before the end of main.
*/
```

## Rc<T>, the Reference Counted Smart Pointer