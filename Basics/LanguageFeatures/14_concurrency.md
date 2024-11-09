# Concurrency

Because threads can run simultaneously, there’s no inherent guarantee about the order in which parts of your code on different threads will run. This can lead to problems, such as:

* Race conditions, where threads are accessing data or resources in an inconsistent order
* Deadlocks, where two threads are waiting for each other, preventing both threads from continuing
* Bugs that happen only in certain situations and are hard to reproduce and fix reliably

**The Rust standard library uses a 1:1 model of thread implementation, whereby a program uses one operating system thread per one language thread.**

**There are crates that implement other models of threading that make different tradeoffs to the 1:1 model.**

## std::thread

* If main thread of a Rust program completes, all spawned threads are shut down, like C++
* use `join` to wait for its thread to finish, the return type of `thread::spawn` is `JoinHandle`

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

Use move to force the closure to take ownership of the values it’s using rather than allowing Rust to infer that it should borrow the values.

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

## Using Message Passing to Transfer Data Between Threads

```rust
// mpsc stands for multiple producer, single consumer.
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

The receiver has two useful methods: recv and try_recv. We’re using recv, short for receive, which will block the main thread’s execution and wait until a value is sent down the channel. Once a value is sent, recv will return it in a Result<T, E>. When the transmitter closes, recv will return an error to signal that no more values will be coming.

The try_recv method doesn’t block, but will instead return a Result<T, E> immediately: an Ok value holding a message if one is available and an Err value if there aren’t any messages this time. Using try_recv is useful if this thread has other work to do while waiting for messages: we could write a loop that calls try_recv every so often, handles a message if one is available, and otherwise does other work for a little while until checking again.

### Ownership Transference

The `send` function takes ownership of its parameter, and when the value is moved, the `receiver` takes ownership of it. This stops us from accidentally using the value again after sending it; the ownership system checks that everything is okay.

### Creating Multiple Producers by Cloning the Transmitter

```rust
    // --snip--

    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }

    // --snip--
```

## Shared-State Concurrency

### Mutexes

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);
    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }
    println!("{}", m);
}
```

The call to lock would fail if another thread holding the lock panicked. In that case, no one would ever be able to get the lock, so we’ve chosen to unwrap and have this thread panic if we’re in that situation.

After we’ve acquired the lock, we can treat the return value, named num in this case, as a mutable reference to the data inside. The type system ensures that we acquire a lock before using the value in m. The type of m is Mutex<i32>, not i32, so we must call lock to be able to use the i32 value. We can’t forget; the type system won’t let us access the inner i32 otherwise.

As you might suspect, **Mutex<T> is a smart pointer.** More accurately, the call to lock returns a smart pointer called `MutexGuard`, wrapped in a LockResult that we handled with the call to unwrap. The `MutexGuard` smart pointer implements Deref to point at our inner data; the smart pointer also has a Drop implementation that releases the lock automatically when a `MutexGuard` goes out of scope, which happens at the end of the inner scope. As a result, we don’t risk forgetting to release the lock and blocking the mutex from being used by other threads, because the lock release happens automatically.

### Sharing a Mutex<T> between multiple threads

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // let counter = Mutex::new(0); ERROR: `Mutex<i32>`, which does not implement the `Copy` trait

    // let counter = Rc::new(Mutex::new(0)); ERROR: `Rc<Mutex<i32>>` cannot be sent between threads safely

    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

## `Sync` and `Send` Traits

### Transfer ownership between threads with `Send`

* Almost every Rust type is Send, but there are some exceptions, including `Rc<T>` 
  * avoid both threads might update the reference count at the same time
* Almost all primitive types are Send, aside from raw pointers

### Access from multiple threads with `Sync`

* The Sync marker trait indicates that it is safe for the type implementing Sync to be referenced from multiple threads. 
* any type T is Sync if &T (an immutable reference to T) is Send
* Similar to Send, primitive types are Sync
* types composed entirely of types that are Sync are also Sync

### Implementing Send and Sync manually is unsafe
