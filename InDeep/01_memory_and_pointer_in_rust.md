# 如何在rust中操作内存

## [Layout](https://doc.rust-lang.org/std/alloc/struct.Layout.html)

An instance of Layout describes a particular layout of memory. You build a Layout up as an input to give to an allocator.

All layouts have an associated size and a power-of-two alignment. The size, when rounded up to the nearest multiple of align, does not overflow isize (i.e., the rounded value will always be less than or equal to isize::MAX).

(Note that layouts are not required to have non-zero size, even though GlobalAlloc requires that all memory requests be non-zero in size. A caller must either ensure that conditions like this are met, use specific allocators with looser requirements, or use the more lenient Allocator interface.)

## [Trait std::alloc::GlobalAlloc](https://doc.rust-lang.org/std/alloc/trait.GlobalAlloc.html)

```rust
pub unsafe trait GlobalAlloc {
    // Required methods
    unsafe fn alloc(&self, layout: Layout) -> *mut u8;
    unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout);

    // Provided methods
    unsafe fn alloc_zeroed(&self, layout: Layout) -> *mut u8 { ... }
    unsafe fn realloc(
        &self,
        ptr: *mut u8,
        layout: Layout,
        new_size: usize,
    ) -> *mut u8 { ... }
}
```

A memory allocator that can be registered as the standard library’s default through the #[global_allocator] attribute.

Some of the methods require that a memory block be currently allocated via an allocator. This means that:

* the starting address for that memory block was previously returned by a previous call to an allocation method such as alloc, and

* the memory block has not been subsequently deallocated, where blocks are deallocated either by being passed to a deallocation method such as dealloc or by being passed to a reallocation method that returns a non-null pointer.


## Unique 和 NonNull

`Unique<T>`是Rust中的一种指针类型，通常在Rust内部使用它来表示唯一所有权语义，对于构建像`Box<T>`、`Vec<T>`、`String`和`HashMap<K, V>`这样的抽象非常有用。

它被定义在标准库的core/src/ptr/unique.rs中：

```rust
pub struct Unique<T: ?Sized> {
    pointer: NonNull<T>,
    _marker: PhantomData<T>,
}
```

`Unique<T>`是非空的、协变的、唯一的指针。以下是这些术语的含义：
* 非空：它保证永远不会为NULL。
* 协变：这个属性允许它与子类型和超类型一起使用。在Rust中，`Unique<T>`的协变被用来确保`Box<T>`与T协变。
* 唯一：它表示指针只有一个所有者。

这些属性的组合允许Rust做出某些可以在代码中进行优化的假设。例如，因为`Unique<T>`指针是非空且唯一的，Rust的优化器可以假设对`Unique<T>`的解引用总是安全的，并且该指针不会被代码的任何其他部分修改或移动。

请注意`Unique<T>`是Rust标准库的内部实现。因此，你通常不会在自己的代码中直接使用这种类型。相反，你应该使用`Box<T>`、`Vec<T>`等类型，它们是在幕后使用Unique构建的。

例如，Box<T>在标准库中的定义如下：

```rust
pub struct Box<
    T: ?Sized,
    #[unstable(feature = "allocator_api", issue = "32838")] A: Allocator = Global,
>(Unique<T>, A);
```

如上所示，`Unique<T>`有一个类型为`PhantomData<T>`的字段_marker。
Rust中的`PhantomData<T>`是一种类型，用于指示特定的泛型T是数据结构的一部分，即使T没有在数据结构中使用。
`PhantomData<T>`在Rust编译器理解所有权、借用和生命周期的能力中起着至关重要的作用，在创建复杂的数据结构或处理原始指针时非常重要。
在`Unique<T>`结构的声明中，`PhantomData<T>`表示它“拥有”一个T，即使T实际上并没有在结构中使用。这告诉Rust编译器，当一个`Unique<T>`被删除时，它可能会删除一个T，所以它不应该允许之后使用这个T。

如果没有`PhantomData<T>`， Rust的类型系统将无法理解`Unique<T>`拥有一个T，这可能会导致未定义的行为。
总之，在Rust中，`PhantomData<T>`和`Unique<T>`经常一起使用来表达复杂的所有权关系，这些关系是Rust的类型系统无法单独表达的。

`Unique<T>`不直接存储原始指针。相反，它有一个`NonNull<T>`类型的指针字段。
在Rust中，`NonNull<T>`是保证不为NULL的原始指针的包装器。它本质上和*mut是一样的，只是增加了一个不为null的不变量。Rust中的原始指针通常在与C代码交互或构建不安全的Rust抽象时使用。然而，空指针经常会导致错误和崩溃，因此提供`NonNull<T>`作为一种确保原始指针永远不会为null的方法。

这提供了一些安全保证，因为你不必在使用它之前检查null。但是请记住，即使它不能为空，使用`NonNull<T>`仍然是不安全的，因为它不提供任何引用(&T和&mut)所提供的借用或生命周期保证。

下面是一个如何使用NonNull<T>的例子：

```rust
use std::ptr::NonNull;
let mut value = 10;
let pointer = NonNull::new(&mut value as *mut _); 
if let Some(ptr) = pointer {
    unsafe {
        *ptr.as_ptr() = 20;
    }
    assert_eq!(value, 20);
}
```

在本例中，`NonNull::new`用于创建一个新的`NonNull<T>`。然后，使用`NonNull::as_ptr`获取原始指针，该指针可以在`unsafe`块中解引用和修改。

## 一个例子

```rust
use std::alloc::{alloc, dealloc, Layout};

fn main() {
    // 使用layout来说明需要申请的内存大小，这里需要4字节来存储一个i32
    let layout = Layout::new::<i32>();
    // 使用alloc分配内存，并返回这块内存的可变的i32指针        
    let ptr = unsafe { alloc(layout) } as *mut i32;

    unsafe {
        *ptr = 42;
    }

    let value = unsafe { *ptr };
    println!("value: {}", value);

    unsafe {
        dealloc(ptr as *mut u8, layout);    // 需要统一使用 *mut u8来释放内存
    }
}
```


## Ref

- [Rust中的Unique 和 NonNull的用法](https://www.duidaima.com/Group/Topic/Rust/14535)