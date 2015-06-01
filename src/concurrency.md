% Concurrency 并发

Concurrency and parallelism are incredibly important topics in computer science, and are also a hot topic in industry today. Computers are gaining more and more cores, yet many programmers aren't prepared to fully utilize them.

并发和并行是计算机科学的非常重要课题，也是当今社会行业的一个热门话题。计算机正获得越来越多的内核，然而，大多数的程序员都没有做好充分利用他们的准备。

Rust's memory safety features also apply to its concurrency story too. Even concurrent Rust programs must be memory safe, having no data races. Rust's type system is up to the task, and gives you powerful ways to reason about concurrent code at compile time.

Rust的内存安全特性同样也允许它适用于并发。甚至并发的Rust程序也必须是内存安全的，没有数据溢出。Rust的类型体系能够胜任这一工作，并能够提供给你在编译时推断并发代码的强大的方式。

Before we talk about the concurrency features that come with Rust, it's important to understand something: Rust is low-level enough that all of this is provided by the standard library, not by the language. This means that if you don't like some aspect of the way Rust handles concurrency, you can implement an alternative way of doing things. [mio](https://github.com/carllerche/mio) is a real-world example of this principle in action.

在我们谈论Rust语言的并发之前，理解一些概念是非常重要的：Rust语言是非常底层的语言，所有的一切都是由标准库来提供的，而不是由语言本身。这意味着，如果你不喜欢Rust语言处理并发方式的某些部分时，你可以使用另一种方式来做同样的事情。[mio](https://github.com/carllerche/mio) 是一个践行这一原则的真实的例子。

## Background: `Send` and `Sync` 后台`发送`和`同步`

Concurrency is difficult to reason about. In Rust, we have a strong, static type system to help us reason about our code. As such, Rust gives us two traits to help us make sense of code that can possibly be concurrent.

并发是很难推理的。在Rust语言中，我们有一个强大的静态类型体系来帮助我们推断代码。正因此，Rust提供给我们两个显著特性帮助我们写出感觉上不可能并发的代码。

### `Send` `发送`

The first trait we're going to talk about is [`Send`](../std/marker/trait.Send.html). When a type `T` implements `Send`, it indicates to the compiler that something of this type is able to have ownership transferred safely between threads.

我们要讲的第一个特性就是[`send发送`](../std/marker/trait.Send.html)。当一个类型`T`实现了`Send`时，它表示着，编译器有权在线程间安全传递这个类型的内容。

This is important to enforce certain restrictions. For example, if we have a channel connecting two threads, we would want to be able to send some data down the channel and to the other thread. Therefore, we'd ensure that `Send` was implemented for that type.

强制执行一定的限制是非常必要的。例如，我们有一个包含两个线程的信道，我们想要能够发送一些数据到其他的线程中。所以我们需要确定`Send`是那个类型的实现方式。



In the opposite way, if we were wrapping a library with FFI that isn't threadsafe, we wouldn't want to implement `Send`, and so the compiler will help
us enforce that it can't leave the current thread.

反过来，如果我们正在封装一个不是线程安全的，使用了外部函数接口的库，我们不需要实现`send`，所以编译器需要帮助我们限制它不能够离开当前的线程。

### `Sync`  `同步`

The second of these traits is called [`Sync`](../std/marker/trait.Sync.html).When a type `T` implements `Sync`, it indicates to the compiler that something of this type has no possibility of introducing memory unsafety when used from multiple threads concurrently.

第二个显著特性叫做[`Sync同步`](../std/marker/trait.Sync.html)。当一个类型`T`实现了`sync`时，他表示着，在使用多个并发线程时，编译器没有可能引入不安全内存类型的内容。

For example, sharing immutable data with an atomic reference count is threadsafe. Rust provides a type like this, `Arc<T>`, and it implements `Sync`,so it is safe to share between threads.

例如，使用一个原子级引用计数共享不可变数据室线程安全的。Rust语言提供了一个像这样的类型——`Arc<T>`,它实现了`Sync`,所以它是安全的在线程之间共享内容。

These two traits allow you to use the type system to make strong guarantees about the properties of your code under concurrency. Before we demonstrate why, we need to learn how to create a concurrent Rust program in the first place!

这两个显著特性允许你使用类型体系，从而在并发时给你代码属性的一个强大的保障。。在我们解释原因之前，我们需要学习，如何在一开始便创建一个并发的Rust程序。

## Threads 线程

Rust's standard library provides a library for threads, which allow you to run Rust code in parallel. Here's a basic example of using `std::thread`:

Rust标准库提供一个线程库，这个库允许你并行运行Rust代码。这是一个使用`std::thread`的基本的例子：

```
use std::thread;

fn main() {
    thread::spawn(|| {
        println!("Hello from a thread!");
    });
}
```

The `thread::spawn()` method accepts a closure, which is executed in a new thread. It returns a handle to the thread, that can be used to wait for the child thread to finish and extract its result:

`thread::spawn()`方法接受一个闭包，这个闭包被执行在一个新的线程上。它返回一个线程的句柄，这个句柄可以用来等待子线程结束并提取结果：

```
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        "Hello from a thread!"
    });

    println!("{}", handle.join().unwrap());
}
```

Many languages have the ability to execute threads, but it's wildly unsafe.There are entire books about how to prevent errors that occur from shared mutable state. Rust helps out with its type system here as well, by preventing data races at compile time. Let's talk about how you actually share things
between threads.

许多语言具有执行多线程的能力，然而，大部分都不安全。有很多正本都是关于如何在共享可变状态时避免产生错的的书籍。同样，Rust用它的类型体系通过避免在编译时产生数据溢出来帮助我们。让我们讨论一下如何真正的在线程间共享内容。

## Safe Shared Mutable State  安全共享的可变状态

Due to Rust's type system, we have a concept that sounds like a lie: "safe shared mutable state." Many programmers agree that shared mutable state is
very, very bad.

因为Rust的类型体系，我们有了一个概念，听起来这像是一个谎言：“安全共享的可变状态”。很多程序员都认为 共享可变状态时非常非常可怕的。

Someone once said this:

有些人曾经这样说过：

> Shared mutable state is the root of all evil. Most languages attempt to deal with this problem through the 'mutable' part, but Rust deals with it by solving the 'shared' part.
> 共享可变状态时所有罪恶的根源。大多数语言试图通过`可变`部分来处理这个问题，然而Rust是通过解决`共享`部分来处理它的。

The same [ownership system](ownership.html) that helps prevent using pointers incorrectly also helps rule out data races, one of the worst kinds of
concurrency bugs.

同样，[ownership system所有权体系](ownership.html) 也是帮助避免使用错误的指针，同样帮助排除数据溢出——最坏情况下的并发错误。

As an example, here is a Rust program that would have a data race in many languages. It will not compile:

这里有一个例子，是一个Rust程序，在很多语言它会产生数据溢出。它不能编译：

```ignore
use std::thread;

fn main() {
    let mut data = vec![1u32, 2, 3];

    for i in 0..3 {
        thread::spawn(move || {
            data[i] += 1;
        });
    }

    thread::sleep_ms(50);
}
```

This gives us an error:

这个会给出一个错误信息：

```text
8:17 error: capture of moved value: `data`
        data[i] += 1;
        ^~~~
```

In this case, we know that our code _should_ be safe, but Rust isn't sure. And it's actually not safe: if we had a reference to `data` in each thread, and the thread takes ownership of the reference, we have three owners! That's bad. We can fix this by using the `Arc<T>` type, which is an atomic reference counted pointer. The 'atomic' part means that it's safe to share across threads.

在这个案例中，我们知道我们的代码 _应当_ 是安全的，然而，Rust无法确定。事实上，它并不安全：如果我们有一个`data`的地址因为在每一个线程中，线程获得了地址引用的所有权，我们就有了三个拥有者！这是很可怕的。我们可以通过使用一个原子引用计数指针的`Arc<T>`类型来修复这个问题。`原子`部分意味着，他是安全跨线程共享的。

`Arc<T>` assumes one more property about its contents to ensure that it is safe to share across threads: it assumes its contents are `Sync`. But in our
case, we want to be able to mutate the value. We need a type that can ensure only one person at a time can mutate what's inside. For that, we can use the
`Mutex<T>` type. Here's the second version of our code. It still doesn't work,but for a different reason:

`Arc<T>`假定其内容多了一部分，一次来确定，他是跨线程安全共享的：它假定它的内容是`Sync`。然而在我们的例子中，我们想要的是能够改变这个值。我们需要一个能够确定在同一时间只有一个人能够改变内部内容。为此我们可以使用`Mutex<t>`类型。这是第二版的代码。因为另一个原因，它仍然不能够运行：

```ignore
use std::thread;
use std::sync::Mutex;

fn main() {
    let mut data = Mutex::new(vec![1u32, 2, 3]);

    for i in 0..3 {
        let data = data.lock().unwrap();
        thread::spawn(move || {
            data[i] += 1;
        });
    }

    thread::sleep_ms(50);
}
```

Here's the error:

这是错误信息：

```text
<anon>:9:9: 9:22 error: the trait `core::marker::Send` is not implemented for the type `std::sync::mutex::MutexGuard<'_, collections::vec::Vec<u32>>` [E0277]
<anon>:11         thread::spawn(move || {
                  ^~~~~~~~~~~~~
<anon>:9:9: 9:22 note: `std::sync::mutex::MutexGuard<'_, collections::vec::Vec<u32>>` cannot be sent between threads safely
<anon>:11         thread::spawn(move || {
                  ^~~~~~~~~~~~~
```

You see, [`Mutex`](../std/sync/struct.Mutex.html) has a [`lock`](../std/sync/struct.Mutex.html#method.lock) method which has this signature:

看明白了[`Meutex`](../std/sync/struct.Mutex.html)有一个[`lock锁方法`](../std/sync/struct.Mutex.html#method.lock)，该方法拥有这个签名：

```ignore
fn lock(&self) -> LockResult<MutexGuard<T>>
```

Because `Send` is not implemented for `MutexGuard<T>`, we can't transfer the guard across thread boundaries, which gives us our error.

因为`Send`不能够实现`MutexGuard<T>`，所以我们无法跨线程边界进行防护，这给我们错误信息。

We can use `Arc<T>` to fix this. Here's the working version:

我们可以使用`Arc<T>`来修复它。这是可以正常运行的版本代码：

```
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let data = Arc::new(Mutex::new(vec![1u32, 2, 3]));

    for i in 0..3 {
        let data = data.clone();
        thread::spawn(move || {
            let mut data = data.lock().unwrap();
            data[i] += 1;
        });
    }

    thread::sleep_ms(50);
}
```

We now call `clone()` on our `Arc`, which increases the internal count. This handle is then moved into the new thread. Let's examine the body of the
thread more closely:

我们现在在`Arc`上调用`clone()`，它能够增加内部计数。这个句柄将移动新的线程。让我们更加密切地检查下线程内部信息：

```rust
# use std::sync::{Arc, Mutex};
# use std::thread;
# fn main() {
#     let data = Arc::new(Mutex::new(vec![1u32, 2, 3]));
#     for i in 0..3 {
#         let data = data.clone();
thread::spawn(move || {
    let mut data = data.lock().unwrap();
    data[i] += 1;
});
#     }
#     thread::sleep_ms(50);
# }
```

First, we call `lock()`, which acquires the mutex's lock. Because this may fail,it returns an `Result<T, E>`, and because this is just an example, we `unwrap()` it to get a reference to the data. Real code would have more robust error handling here. We're then free to mutate it, since we have the lock.

首先我们调用`lock()`,它获取互斥的锁状态。因为这个可能会失败，所以他返回一个`Result<T,E>`的信息，并且由于这只是一个例子，我们打开它，获取内部数据的引用。在这个地方，真正的代码将有更多的错误处理。一旦我们拥有了这个锁，我们随意地改变它，

Lastly, while the threads are running, we wait on a short timer. But this is not ideal: we may have picked a reasonable amount of time to wait but it's more likely we'll either be waiting longer than necessary or not long enough, depending on just how much time the threads actually take to finish computing when the program runs.

最后，当线程运行时，我们需要等待一小段时间。然而，这并不够理想：我们可能选择了一个合理的得带时间，但是取决于在程序运行时，线程真正结束计算小号的时间多少，它可能比我们的等待时间还长，或者比如等待时间长。

A more precise alternative to the timer would be to use one of the mechanisms provided by the Rust standard library for synchronizing threads with each other. Let's talk about one of them: channels.

一个更精准的替代计时器是使用一个由Rust标准库提供的，用于同步每一个线程的机制。下面让我们来讨论下他们中的一个：信道。

## Channels  信道

Here's a version of our code that uses channels for synchronization, rather than waiting for a specific time:

这是代码使用信道同步的一个版本，而不是等待一个特定时间：

```
use std::sync::{Arc, Mutex};
use std::thread;
use std::sync::mpsc;

fn main() {
    let data = Arc::new(Mutex::new(0u32));

    let (tx, rx) = mpsc::channel();

    for _ in 0..10 {
        let (data, tx) = (data.clone(), tx.clone());

        thread::spawn(move || {
            let mut data = data.lock().unwrap();
            *data += 1;

            tx.send(());
        });
    }

    for _ in 0..10 {
        rx.recv();
    }
}
```

We use the `mpsc::channel()` method to construct a new channel. We just `send` a simple `()` down the channel, and then wait for ten of them to come back.

我们使用`mpsc::channel()`方式来构造一个新的信道。我们只是`send`一个简单地`()`给信道，然后等待他们十个回来。

While this channel is just sending a generic signal, we can send any data that is `Send` over the channel!

然而，这个信道只是发送了一个通用的信号，我们可以通过信道发送任意数据！

```
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    for _ in 0..10 {
        let tx = tx.clone();

        thread::spawn(move || {
            let answer = 42u32;

            tx.send(answer);
        });
    }

   rx.recv().ok().expect("Could not receive answer");
}
```

A `u32` is `Send` because we can make a copy. So we create a thread, ask it to calculate the answer, and then it `send()`s us the answer over the channel.

`u32`被`Send` 因为我们制造了一个副本。因此，我们创建了一个新的线程，请求他计算答案，然后它通过信道将答案`send()`回来。


## Panics

A `panic!` will crash the currently executing thread. You can use Rust's threads as a simple isolation mechanism:

`pannic!`会使当前正在执行的线程崩溃。你可以使用Rust的线程作为一个简单地隔离机制：


```
use std::thread;

let result = thread::spawn(move || {
    panic!("oops!");
}).join();

assert!(result.is_err());
```

Our `Thread` gives us a `Result` back, which allows us to check if the thread has panicked or not.

`Thread线程`返回来一个`Result结果`以便我们能够检查线程是否崩溃。