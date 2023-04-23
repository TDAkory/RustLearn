# Understand Smart Pointer in Rust

The most common kind of pointer in Rust is a reference, indicated by the `&` symbol and borrow the value they point to. 

Though we didn’t call them as such at the time, we’ve already encountered a few smart pointers in this book, including `String` and `Vec<T>`.

smart pointers implement the `Deref` and `Drop` traits.

* `Box<T>` for allocating values on the heap
* `Rc<T>`, a reference counting type that enables multiple ownership
* `Ref<T>` and `RefMut<T>`, accessed through `RefCell<T>`, a type that enforces the borrowing rules at runtime instead of compile time

## Box