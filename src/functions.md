% Functions 函数

Every Rust program has at least one function, the `main` function:

每一个Rust程序至少都要有一个函数——`main`函数：

```rust
fn main() {
}
```

This is the simplest possible function declaration. As we mentioned before,`fn` says ‘this is a function’, followed by the name, some parentheses because this function takes no arguments, and then some curly braces to indicate the body. Here’s a function named `foo`:

这是最简单的函数声明。正如我们之前提到的，`fn`表示“这是一个函数”， 紧跟着的是函数名，然后是一对括号，因为这个函数没有使用参数，然后是一对花括号来表示函数体本身。这里有一个叫做`foo`的函数：

```rust
fn foo() {
}
```

So, what about taking arguments? Here’s a function that prints a number:

那么，有参数的话会怎样呢？这里有一个能够打印出一个数字的函数：

```rust
fn print_number(x: i32) {
    println!("x is: {}", x);
}
```

Here’s a complete program that uses `print_number`:

这里有一个完成的程序使用`print_number`函数;

```rust
fn main() {
    print_number(5);
}

fn print_number(x: i32) {
    println!("x is: {}", x);
}
```

As you can see, function arguments work very similar to `let` declarations:you add a type to the argument name, after a colon.

正如你看到的，函数参数非常想`let`声明：在冒号后面给参数名增加一个类型

Here’s a complete program that adds two numbers together and prints them:

这里有一个完整的例子，一次使用两个数字，并打印他们：

```rust
fn main() {
    print_sum(5, 6);
}

fn print_sum(x: i32, y: i32) {
    println!("sum is: {}", x + y);
}
```

You separate arguments with a comma, both when you call the function, as well as when you declare it.

当你调用一个函数时，使用一个逗号来分割参数，就像你声明它时一样。

Unlike `let`, you _must_ declare the types of function arguments. This does not work:

不像`let`声明，你 _必须_ 声明函数参数的类型。下面这样的程序不会运行：

```rust,ignore
fn print_sum(x, y) {
    println!("sum is: {}", x + y);
}
```

You get this error:

你会得到错误：

```text
expected one of `!`, `:`, or `@`, found `)`
fn print_number(x, y) {
```

This is a deliberate design decision. While full-program inference is possible,languages which have it, like Haskell, often suggest that documenting your types explicitly is a best-practice. We agree that forcing functions to declare types while allowing for inference inside of function bodies is a wonderful sweet spot between full inference and no inference.

这是一个深思熟虑的设计决定。虽然全程序推断是可能的，拥有这种方式的语言有Haskell，经常建议明确的注释你的参数类型是一个最佳实践。我们同意在允许函数体内部推断的同时强制函数声明类型是全局推断和禁止推断之间的灵活点。

What about returning a value? Here’s a function that adds one to an integer:

有返回值会怎样？这里有一个对整数+1函数：

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
```

Rust functions return exactly one value, and you declare the type after an ‘arrow’, which is a dash (`-`) followed by a greater-than sign (`>`). The last line of a function determines what it returns. You’ll note the lack of a semicolon here. If we added it in:

Rust函数的返回一个明确的值，你在一个“箭头”（有一个破折号`-`紧跟着一个大于号`>`）之后声明一个类型。函数的最后一行角定了它返回什么。你会注意到这里缺少一个分号。如果我们加入了它（分号）：

```rust,ignore
fn add_one(x: i32) -> i32 {
    x + 1;
}
```

We would get an error:

会报错

```text
error: not all control paths return a value
fn add_one(x: i32) -> i32 {
     x + 1;
}

help: consider removing this semicolon:
     x + 1;
          ^
```

This reveals two interesting things about Rust: it is an expression-based language, and semicolons are different from semicolons in other ‘curly brace and semicolon’-based languages. These two things are related.

这里揭示了Rust的两个有趣的内容：它是一个基于表达式的语言，并且分号不同于其他基于“花括号和分号”的语言。这两件事情是相关联的。

## Expressions vs. Statements  表达式 VS. 声明

Rust is primarily an expression-based language. There are only two kinds of statements, and everything else is an expression.

Rust是一个主要基于表达是的语言。只有两种声明类型，并且其他一切都是表达式。

So what's the difference? Expressions return a value, and statements do not.That’s why we end up with ‘not all control paths return a value’ here: the statement `x + 1;` doesn’t return a value. There are two kinds of statements in Rust: ‘declaration statements’ and ‘expression statements’. Everything else is an expression. Let’s talk about declaration statements first.

有什么区别呢？表达式返回一个值，声明不会。这就是为什么我们在这里使用‘不是全部控制路径返回值’：声明`x + 1;`不会返回值。在Rust中有两种声明方式：‘宣布声明’和‘表达式声明’。其他一切都是表达式。我们首先讨论下宣布声明；

In some languages, variable bindings can be written as expressions, not just statements. Like Ruby:

在某些语言中，变量绑定可以被写成表达式，而不只是一个声明。像是Ruby语言：

```ruby
x = y = 5
```

In Rust, however, using `let` to introduce a binding is _not_ an expression. The following will produce a compile-time error:

然而在Rust中 在一个变量绑定时使用`let`_不是_ 一个表达式。下面的代码将产生一个编译错误：

```ignore
let x = (let y = 5); // expected identifier, found keyword `let`
```

The compiler is telling us here that it was expecting to see the beginning of an expression, and a `let` can only begin a statement, not an expression.

编译器在告诉我们 这里被预期为一个表达式的开始，但是`let`只能够开始一个声明，不能够开始一个表达式。

Note that assigning to an already-bound variable (e.g. `y = 5`) is still an expression, although its value is not particularly useful. Unlike other languages where an assignment evaluates to the assigned value (e.g. `5` in the previous example), in Rust the value of an assignment is an empty tuple `()`:

需要注意的是，指定一个已经准备好的变量（例如`y = 5`）仍然是一个表达式，尽管他的值不是特别有用。不像其他语言，一个赋值语句回来指定变量（例如前面一个例子中的`5`），在Rust中，指定的值僵尸一个空的元组`()`：

```
let mut y = 5;

let x = (y = 6);  // x has the value `()`, not `6`
```

The second kind of statement in Rust is the *expression statement*. Its purpose is to turn any expression into a statement. In practical terms, Rust's grammar expects statements to follow other statements. This means that you use semicolons to separate expressions from each other. This means that Rust looks a lot like most other languages that require you to use semicolons at the end of every line, and you will see semicolons at the end of almost every line of Rust code you see.

在Rust中的第二种声明是*表达式声明*。它的目的是将任意表达式转换成语句。就真实情况而言，Rus的语法希望语句来跟随其他声明。这意味着，你使用分号来区分表达式。这也意味着Rust看起来像是其他大多数要求你在每一行的结束使用分号的语言，并且你将会在你见到的Rust代码的几乎每一行的末尾都有分号。

What is this exception that makes us say "almost"? You saw it already, in this code:

是什么让我们说“几乎”?你已经见过了，在这个代码中：

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
```

Our function claims to return an `i32`, but with a semicolon, it would return `()` instead. Rust realizes this probably isn’t what we want, and suggests removing the semicolon in the error we saw before.

我们的函数声明返回一个`i32`整数，但是使用了分号后，他只能够反悔`()`。Rust意识到这可能不是我们需要的，并会建议移除我们之前看到的错误中的分号。

## Early returns  及早返回

But what about early returns? Rust does have a keyword for that, `return`:

什么是及早返回？Rust有一个关键词`return`：

```rust
fn foo(x: i32) -> i32 {
    return x;

    // we never run this code!
    x + 1
}
```

Using a `return` as the last line of a function works, but is considered poor style:

在函数的最后一行使用`return`，是可以运行的，但是被认为是差的格式：

```rust
fn foo(x: i32) -> i32 {
    return x + 1;
}
```

The previous definition without `return` may look a bit strange if you haven’t worked in an expression-based language before, but it becomes intuitive over time.

如果你之前没有使用过一门基于表达式的语言的话，前面没有使用`return`的定义语句看起可能有点怪，但是随着时间的迁移，它将变得直观。

## Diverging functions  发散函数

Rust has some special syntax for ‘diverging functions’, which are functions that do not return:

Rust有一些特殊语法用于‘发散函数’，它是一个没有返回值的函数：

```
fn diverges() -> ! {
    panic!("This function never returns!");
}
```

`panic!` is a macro, similar to `println!()` that we’ve already seen. Unlike `println!()`, `panic!()` causes the current thread of execution to crash with the given message.

`panic!`是一个宏，跟我们已经见过的`println!()`相似。不像`println!()`,`panic!()`使用给定的信息致使执行的当前线程崩溃。

Because this function will cause a crash, it will never return, and so it has the type ‘`!`’, which is read ‘diverges’. A diverging function can be used as any type:

因为这个函数会致使一个崩溃，它永远没有返回值，所以他使用被称为‘发散’的`!`类型。一个发散函数可以被用于任意类型：

```should_panic
# fn diverges() -> ! {
#    panic!("This function never returns!");
# }
let x: i32 = diverges();
let x: String = diverges();
```
