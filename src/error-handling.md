% Error Handling 错误处理

> The best-laid plans of mice and men Often go awry
>
> "Tae a Moose", Robert Burns
>“不管是人是鼠，即使最如意的安排设计，结局也往往会出其不意。”
>致老鼠，罗伯特·彭斯

Sometimes, things just go wrong. It's important to have a plan for when the inevitable happens. Rust has rich support for handling errors that may (let's be honest: will) occur in your programs.

有些时候，程序就是出错。所以当事情不可避免的发生时，有一个计划是非常重要的。Rust拥有丰富的错误处理机制，或许就会（说实话，应该将会）在你的程序中出现。

There are two main kinds of errors that can occur in your programs: failures,and panics. Let's talk about the difference between the two, and then discuss how to handle each. Then, we'll discuss upgrading failures to panics.

在程序中发生的错误主要有两大主要错误：失败和panics。让我们先谈谈两者的区别，然后再讨论如何处理他们。最后我们将讨论将失败升级为panics。

# Failure vs. Panic  失败 和 panic

Rust uses two terms to differentiate between two forms of error: failure, and panic. A *failure* is an error that can be recovered from in some way. A *panic* is an error that cannot be recovered from.

Rust语言有两种术语来区分两种形式的错误：失败和panic。*failure* 是能够以某种方式恢复的错误。而*panic*则是不能够恢复的错误。

What do we mean by "recover"? Well, in most cases, the possibility of an error is expected. For example, consider the `parse` function:

那么恢复对我们来说意味着什么？好吧，在大多数情况下，错误是能够被预见到的。例如，参考一下`parse`函数：

```ignore
"5".parse();
```

This method converts a string into another type. But because it's a string, you can't be sure that the conversion actually works. For example, what should this convert to?

这个方法是转换字符串到另一个类型。但是因为他是一个字符串，我们不能够确定转换能够实际发生。例如这个应该转换成什么？

```ignore
"hello5world".parse();
```

This won't work. So we know that this function will only work properly for some inputs. It's expected behavior. We call this kind of error a *failure*.

这是无法运行的。尽管我们知道，这个方法只对某些输入正常解析。他是预期行为。我们成这种错误叫做*failure*。

On the other hand, sometimes, there are errors that are unexpected, or which we cannot recover from. A classic example is an `assert!`:

另一方面，有时候，有些错误是无法预见的，或者是我们无法进行转换的。一个经典的例子是一个`assert!`:

```rust
# let x = 5;
assert!(x == 5);
```

We use `assert!` to declare that something is true. If it's not true, something is very wrong. Wrong enough that we can't continue with things in the current state. Another example is using the `unreachable!()` macro:

我们使用`assert!`来声明某些事情是正确的。如果它不是正确的，事情就会非常糟糕。错误以致我们在当前状态下无法继续。另一个例子是使用`unreachable!()`宏：

```{rust,ignore}
enum Event {
    NewRelease,
}

fn probability(_: &Event) -> f64 {
    // real implementation would be more complex, of course
    0.95
}

fn descriptive_probability(event: Event) -> &'static str {
    match probability(&event) {
        1.00 => "certain",
        0.00 => "impossible",
        0.00 ... 0.25 => "very unlikely",
        0.25 ... 0.50 => "unlikely",
        0.50 ... 0.75 => "likely",
        0.75 ... 1.00 => "very likely",
    }
}

fn main() {
    std::io::println(descriptive_probability(NewRelease));
}
```

This will give us an error:

这将如下显示错误：

```text
error: non-exhaustive patterns: `_` not covered [E0004]
```

While we know that we've covered all possible cases, Rust can't tell. It doesn't know that probability is between 0.0 and 1.0. So we add another case:

尽管我们知道我们已经涵盖了所有可能的情况，Rust不能。他不知道的概率在0.0到1.0。所以我们又举出了另一个例子：

```rust
use Event::NewRelease;

enum Event {
    NewRelease,
}

fn probability(_: &Event) -> f64 {
    // real implementation would be more complex, of course
    0.95
}

fn descriptive_probability(event: Event) -> &'static str {
    match probability(&event) {
        1.00 => "certain",
        0.00 => "impossible",
        0.00 ... 0.25 => "very unlikely",
        0.25 ... 0.50 => "unlikely",
        0.50 ... 0.75 => "likely",
        0.75 ... 1.00 => "very likely",
        _ => unreachable!()
    }
}

fn main() {
    println!("{}", descriptive_probability(NewRelease));
}
```

We shouldn't ever hit the `_` case, so we use the `unreachable!()` macro to indicate this. `unreachable!()` gives a different kind of error than `Result`.Rust calls these sorts of errors *panics*.

我们永远无法命中`_`情况，所以我们使用了`unreachable!()`宏来表明这一点。`unreachable!()`产生了一个不同的错误 而不是`Result`。Rust语言中，这些类型的错误统称为`panics`。

# Handling errors with `Option` and `Result` 使用`option选项`和`Result结果`来处理错误信息

The simplest way to indicate that a function may fail is to use the `Option<T>` type. For example, the `find` method on strings attempts to find a pattern in a string, and returns an `Option`:

表示一个函数可能失败的最简单方式是使用`Option<T>`类型。举个栗子，基于字符串的`find`方法，试图在一个字符串中找到一个模式，并返回一个`Option`:

```rust
let s = "foo";

assert_eq!(s.find('f'), Some(0));
assert_eq!(s.find('z'), None);
```


This is appropriate for the simplest of cases, but doesn't give us a lot of information in the failure case. What if we wanted to know _why_ the function failed? For this, we can use the `Result<T, E>` type. It looks like this:

这个只能适用于最简单的情况，也无法给我提供在失败情况下更多的信息。如果我们想知道， _为什么_ 这个方法会失败？为此，我们可以使用`Result<T,E>`类型。它看起来是这样子的：

```rust
enum Result<T, E> {
   Ok(T),
   Err(E)
}
```

This lumen is provided by Rust itself, so you don't need to define it to use it in your code. The `Ok(T)` variant represents a success, and the `Err(E)` variant represents a failure. Returning a `Result` instead of an `Option` is recommended for all but the most trivial of situations.

枚举类型是由Rust本身提供的，所以我们在代码中，不需要定义它即可使用。`Ok<T>`变量代表着成功，`Err<T>`变量代表着失败。除了最简单的情况，我们推荐你使用返回一个`Return`而不是使用一个`Option`。

Here's an example of using `Result`:

这里有一个使用`Result`的例子：

```rust
#[derive(Debug)]
enum Version { Version1, Version2 }

#[derive(Debug)]
enum ParseError { InvalidHeaderLength, InvalidVersion }

fn parse_version(header: &[u8]) -> Result<Version, ParseError> {
    if header.len() < 1 {
        return Err(ParseError::InvalidHeaderLength);
    }
    match header[0] {
        1 => Ok(Version::Version1),
        2 => Ok(Version::Version2),
        _ => Err(ParseError::InvalidVersion)
    }
}

let version = parse_version(&[1, 2, 3, 4]);
match version {
    Ok(v) => {
        println!("working with version: {:?}", v);
    }
    Err(e) => {
        println!("error parsing header: {:?}", e);
    }
}
```

This function makes use of an enum, `ParseError`, to enumerate the various errors that can occur.

这个方法使用了一个枚举类型的变量`ParsError`，来列举可能发生的各种错误。

The [`Debug`](../std/fmt/trait.Debug.html) trait is what lets us print the enum value using the `{:?}` format operation.

[`Debug`](../std/fmt/trait.Debug.html)特性使用`{:?}`格式化操作打印枚举值。

# Non-recoverable errors with `panic!` 无法恢复的错误`panic`

In the case of an error that is unexpected and not recoverable, the `panic!` macro will induce a panic. This will crash the current thread, and give an error:

在这个错误的案例中，错误是不能够预见的，且无法恢复的，`panic`宏将引发一个panic。这将会导致当前线程崩溃，并给出如下错误：

```{rust,ignore}
panic!("boom");
```

gives  

```text
thread '<main>' panicked at 'boom', hello.rs:2
```

when you run it.

当你运行它时，会给出如上信息。

Because these kinds of situations are relatively rare, use panics sparingly.

由于这种情况那个是很少见的，所以应该尽量避免使用panics。

# Upgrading failures to panics  将失败升级为panic

In certain circumstances, even though a function may fail, we may want to treat it as a panic instead. For example, `io::stdin().read_line(&mut buffer)` returns a `Result<usize>`, when there is an error reading the line. This allows us to handle and possibly recover from error.

在某些确定的情况下，即使一个函数可能会失败，我们可能要将它当作panic对待。例如，当有一行内容读取错误时，`io::stdin().read_line(&mut buffer)` 返回一个`Result<usize>`。这允许我们处理，并且可能恢复错误。

If we don't want to handle this error, and would rather just abort the program,we can use the `unwrap()` method:

如果我们不希望处理此错误，宁愿退出程序，我们可以使用`unwrap()`方法：

```{rust,ignore}
io::stdin().read_line(&mut buffer).unwrap();
```

`unwrap()` will `panic!` if the `Result` is `Err`. This basically says "Give me the value, and if something goes wrong, just crash." This is less reliable than matching the error and attempting to recover, but is also significantly shorter. Sometimes, just crashing is appropriate.

如果`Result`是`Err`类型，`unwrap()`将引起`panic!`。基本上来就是“给我那个值，如果出错，只管崩溃！”比起匹配错误，并且试图恢复它，这是更不可靠的，然而这样做明显的简短。

There's another way of doing this that's a bit nicer than `unwrap()`:

还有另一种方式的做法要比`unwrap()`要好：

```{rust,ignore}
let mut buffer = String::new();
let input = io::stdin().read_line(&mut buffer)
                       .ok()
                       .expect("Failed to read line");
```

`ok()` converts the `Result` into an `Option`, and `expect()` does the same
thing as `unwrap()`, but takes a message. This message is passed along to the underlying `panic!`, providing a better error message if the code errors.

`ok()`将`Result`转换成`Option`，`expect()`对`unwrap()`做同样的事情，但是会返回一个消息。这个消息被传递给底层的`panic!`，如果代码出错了，这个底层的`panic!`会提供了一个更好的错误消息

# Using `try!`  使用`try!`

When writing code that calls many functions that return the `Result` type, the error handling can be tedious. The `try!` macro hides some of the boilerplate of propagating errors up the call stack.

当编写的代码调用了很多返回`Result`类型的方法时，错误处理变得枯燥乏味。`try!`宏隐藏了一些会传播错误信息到调用堆栈上的模板。

It replaces this:

它取代了这一点

```rust
use std::fs::File;
use std::io;
use std::io::prelude::*;

struct Info {
    name: String,
    age: i32,
    rating: i32,
}

fn write_info(info: &Info) -> io::Result<()> {
    let mut file = File::create("my_best_friends.txt").unwrap();

    if let Err(e) = writeln!(&mut file, "name: {}", info.name) {
        return Err(e)
    }
    if let Err(e) = writeln!(&mut file, "age: {}", info.age) {
        return Err(e)
    }
    if let Err(e) = writeln!(&mut file, "rating: {}", info.rating) {
        return Err(e)
    }

    return Ok(());
}
```

With this:

使用这个

```rust
use std::fs::File;
use std::io;
use std::io::prelude::*;

struct Info {
    name: String,
    age: i32,
    rating: i32,
}

fn write_info(info: &Info) -> io::Result<()> {
    let mut file = try!(File::create("my_best_friends.txt"));

    try!(writeln!(&mut file, "name: {}", info.name));
    try!(writeln!(&mut file, "age: {}", info.age));
    try!(writeln!(&mut file, "rating: {}", info.rating));

    return Ok(());
}
```

Wrapping an expression in `try!` will result in the unwrapped success (`Ok`) value, unless the result is `Err`, in which case `Err` is returned early from the enclosing function.

在`try!`中包裹一个表达式，将可以得到一个成功的(`Ok`)值，除非结果是`Err`,在每一种情况下，`Err`总是从封闭的函数中及早返回。

It's worth noting that you can only use `try!` from a function that returns a `Result`, which means that you cannot use `try!` inside of `main()`, because `main()` doesn't return anything.

值得一提的是，你只能够对能够返回`Result`值的函数使用`try!`类型，这意味着，你不能够在`main()`方法中使用，因为`main()`不会返回任何内容。

`try!` makes use of [`From<Error>`](../std/convert/trait.From.html) to determine what to return in the error case.

`try!`是使用[`from<Error>`](../std/convert/trait.From.html)来决定在错误情况下返回什么内容。