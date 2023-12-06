# Concurrency

Because threads can run simultaneously, there’s no inherent guarantee about the order in which parts of your code on different threads will run. This can lead to problems, such as:

* Race conditions, where threads are accessing data or resources in an inconsistent order
* Deadlocks, where two threads are waiting for each other, preventing both threads from continuing
* Bugs that happen only in certain situations and are hard to reproduce and fix reliably

**The Rust standard library uses a 1:1 model of thread implementation, whereby a program uses one operating system thread per one language thread.**

**There are crates that implement other models of threading that make different tradeoffs to the 1:1 model.**

## std::thread

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