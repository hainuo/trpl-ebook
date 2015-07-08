% Variable Bindings  变量绑定

Virtually every non-'Hello World’ Rust program uses *variable bindings*. They look like this:

几乎每一个非“hello world”的Rust程序都在使用了*变量绑定*。他们看起来是这样子的：

```rust
fn main() {
    let x = 5;
}
```

Putting `fn main() {` in each example is a bit tedious, so we’ll leave that out in the future. If you’re following along, make sure to edit your `main()` function, rather than leaving it off. Otherwise, you’ll get an error.

在每一个例子中写入一个`fn main(){` 是枯燥无味的，所以我们将它留到以后。如果你跟随者教程，请务必编辑你的`main()`函数，而不是把它删掉，不然你会得到一个报错。

In many languages, this is called a *variable*, but Rust’s variable bindings have a few tricks up their sleeves. For example the left-hand side of a `let` expression is a ‘[pattern][pattern]’, not just a variable name. This means we can do things like:

在许多语言中，这被称之为*变量*，但是Rust的变量绑定有一些高招。例如，一个`let`表达式的左侧是一个 `[pattern 模式][pattern]`，而不是只有一个变量名。这意味着我们可以这样做：

```rust
let (x, y) = (1, 2);
```

After this expression is evaluated, `x` will be one, and `y` will be two.Patterns are really powerful, and have [their own section][pattern] in the book. We don’t need those features for now, so we’ll just keep this in the back of our minds as we go forward.

这个表达式被求值后，`x`将会是1，`y`将会是2. 模式是非常强大的，并且在本书中有[自己的章节][pattern]。现在我们不需要那些特性，我们将这些置于脑后，因为我们还要继续学习。

[pattern]: patterns.md

Rust is a statically typed language, which means that we specify our types up front, and they’re checked at compile time. So why does our first example
compile? Well, Rust has this thing called ‘type inference’. If it can figure out what the type of something is, Rust doesn’t require you to actually type it
out.

Rust是一门静态类型的语言，这意味着我们首先要声明我们的类型，并且在编译时会被检查。那么为什么我们的第一个例子能够编译通过呢？好吧，Rust有个叫做‘类型接口’的东西。如果Rust能够算出某些变量的类型是什么，它并不要求你真正的将它表示出来。

We can add the type if we want to, though. Types come after a colon (`:`):

然而，如果我们需要，我们可以增加类型。类型总是在一个冒号（`:`）后面：

```rust
let x: i32 = 5;
```

If I asked you to read this out loud to the rest of the class, you’d say “`x` is a binding with the type `i32` and the value `five`.”

如果我要求你向全班其余同学大声的读出来，你应该说：“`x` 是一个`i32`类型的变量，它的值是`5`。”

In this case we chose to represent `x` as a 32-bit signed integer. Rust has many different primitive integer types. They begin with `i` for signed integers
and `u` for unsigned integers. The possible integer sizes are 8, 16, 32, and 64 bits. 

在这个案例中，我们选择声明`x`为一个32位的带符号的整数。Rust有很多不同的原始整数类型。他们使用`i`标记带符号的整数，使用`u`标记不带符号的整数。能够被使用的整数字节是8位，16位，32位和64位。

In future examples, we may annotate the type in a comment. The examples will look like this:

在未来的案例中，我们可能在注释中声明一个类型。例子看起来是这样的：

```rust
fn main() {
    let x = 5; // x: i32
}
```

Note the similarities between this annotation and the syntax you use with `let`. Including these kinds of comments is not idiomatic Rust, but we'll
occasionally include them to help you understand what the types that Rust infers are.

请注意这个声明和你使用`let`语法的相似性。包含这种注释不是地道的Rust代码编写方式，然而，我们会偶尔使用来帮助你理解Rust推断的类型是什么。

By default, bindings are *immutable*. This code will not compile:

默认，变量绑定是*不可变的*。以下代码将无法编译：

```rust,ignore
let x = 5;
x = 10;
```

It will give you this error:

它会给出错误：

```text
error: re-assignment of immutable variable `x`
     x = 10;
     ^~~~~~~
```

If you want a binding to be mutable, you can use `mut`:

如果你想要一个可变的变量绑定，你可以使用`mut`：

```rust
let mut x = 5; // mut x: i32
x = 10;
```

There is no single reason that bindings are immutable by default, but we can think about it through one of Rust’s primary focuses: safety. If you forget to
say `mut`, the compiler will catch it, and let you know that you have mutated something you may not have intended to mutate. If bindings were mutable by
default, the compiler would not be able to tell you this. If you _did_ intend mutation, then the solution is quite easy: add `mut`.

变量绑定默认为不可变的，有很多原因，但是我们可以考虑一下Rust的主要关注点之一：安全。如果你忘记标记`mut`，编译器将扑捉它，并且让你知道你已经改变了之前你没有预计到会改变的一个变量。如果变量绑定默认是可变的，编译器将不能够告诉你这个。如果你_希望_改变，解决方式很简单：增加`mut`。

There are other good reasons to avoid mutable state when possible, but they’re out of the scope of this guide. In general, you can often avoid explicit mutation, and so it is preferable in Rust. That said, sometimes, mutation is what you need, so it’s not verboten.

在可能的情况下，还有其他一些好理由来避免可变状态，然而，他们超出了本教程的范围。通常情况下，你能够经常避免明确的可变类型，所以在Rust语言中，这种默认方式是最可取的。这就是说，有些时候，你需要可变类型，所以它们不被禁止。

Let’s get back to bindings. Rust variable bindings have one more aspect that differs from other languages: bindings are required to be initialized with a
value before you're allowed to use them.

让我们回到变量绑定上来。Rust的变量绑定跟其他语言有不止一个的不同方面：在你被允许使用它们之前，变量绑定必须要有一个初始化值。

Let’s try it out. Change your `src/main.rs` file to look like this:

让我们试一下。将你的`src/main.rs`文件更改成下面的样子：

```rust
fn main() {
    let x: i32;

    println!("Hello world!");
}
```

You can use `cargo build` on the command line to build it. You’ll get a warning, but it will still print "Hello, world!":

你可以在命令行中使用`cargo build`来构建它。你会得到一个警告，但是它仍然将打印出“hello world！”：

```text
   Compiling hello_world v0.0.1 (file:///home/you/projects/hello_world)
src/main.rs:2:9: 2:10 warning: unused variable: `x`, #[warn(unused_variable)]
   on by default
src/main.rs:2     let x: i32;
                      ^
```

Rust warns us that we never use the variable binding, but since we never use it, no harm, no foul. Things change if we try to actually use this `x`,however. Let’s do that. Change your program to look like this:

Rust警告我们我们没有使用变量绑定，因为我们没有使用它，没有害处，就没有犯规。然而，如果我们尝试真正的使用这个`x`变量，事情就会变化了。让我们那样做一下。改变你的程序如下：

```rust,ignore
fn main() {
    let x: i32;

    println!("The value of x is: {}", x);
}
```

And try to build it. You’ll get an error:  

然后试图构建它，你会得到一个错误：

```bash
$ cargo build
   Compiling hello_world v0.0.1 (file:///home/you/projects/hello_world)
src/main.rs:4:39: 4:40 error: use of possibly uninitialized variable: `x`
src/main.rs:4     println!("The value of x is: {}", x);
                                                    ^
note: in expansion of format_args!
<std macros>:2:23: 2:77 note: expansion site
<std macros>:1:1: 3:2 note: in expansion of println!
src/main.rs:4:5: 4:42 note: expansion site
error: aborting due to previous error
Could not compile `hello_world`.
```

Rust will not let us use a value that has not been initialized. Next, let’s talk about this stuff we've added to `println!`.

Rust将不会允许我们使用一个我们没有初始化的值。下一步，让我们讨论下我们加入到`println!`方法中的东西。

If you include two curly braces (`{}`, some call them moustaches...) in your string to print, Rust will interpret this as a request to interpolate some sort
of value. *String interpolation* is a computer science term that means "stick in the middle of a string." We add a comma, and then `x`, to indicate that we
want `x` to be the value we’re interpolating. The comma is used to separate arguments we pass to functions and macros, if you’re passing more than one.

如果你在你要打印的字符串中包含了两个花括号（`{}`,有些人称他们胡子），Rust会推断他们为一个请求插入一些顺序的值的敌方。*字符串插入*是一个计算机科学术语，这表示“插入到一个字符串的中间。”我们增加一个逗号，然后是`x`，来表名我们想要将`x`作为我们插入的值。如果你要传递不止一个参数的话，逗号被用于分割我们传递给函数和宏的参数。

When you just use the curly braces, Rust will attempt to display the value in a meaningful way by checking out its type. If you want to specify the format in a more detailed manner, there are a [wide number of options available][format].For now, we'll just stick to the default: integers aren't very complicated to
print.

当我们只需要使用花括号时，Rust通过检查它的类型将试图以一种有意义的方式来显示这个值。如果我们想要指定更详细的格式，这里有一个[很广泛的可选数字类型][format]。现在我们只使用默认：整数对打印来说并不是很复杂的。

[format]: ../std/fmt/index.html
