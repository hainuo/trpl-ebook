% Documentation 文档

Documentation is an important part of any software project, and it's first-class in Rust. Let's talk about the tooling Rust gives you to document your project.

文档在任何软件项目中都是一个很重要的部分，在Rust语言中，他是第一等级的。

## About `rustdoc`  关于`rustdoc`

The Rust distribution includes a tool, `rustdoc`, that generates documentation.`rustdoc` is also used by Cargo through `cargo doc`.

Rust分发版本内置了一个工具——`rustdoc`——用于生成文档。`rustdoc`也同样被Cargo命令`cargo doc`使用。

Documentation can be generated in two ways: from source code, and from
standalone Markdown files.

文档可以通过两种方式生成：从源代码，或者从单独的markdown语句

## Documenting source code

The primary way of documenting a Rust project is through annotating the source code. You can use documentation comments for this purpose:

文档化一个Rust语言项目的首要方式是通过注释源代码。你能够使用文档注释达到此目的：


```rust,ignore
/// Constructs a new `Rc<T>`.
///
/// # Examples
///
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
pub fn new(value: T) -> Rc<T> {
    // implementation goes here
}
```

This code generates documentation that looks [like this][rc-new]. I've left the implementation out, with a regular comment in its place. That's the first thing to notice about this annotation: it uses `///`, instead of `//`. The triple slash indicates a documentation comment.

此代码生成的文档看起来[像这样][rc-new]。我们在代码执行中使用了一个普通的注释。首先要注意的事情是这个符号：使用的是`///`而不是`//`。三个反斜杠表示文档注释。

Documentation comments are written in Markdown.

文档注释是使用Markdown格式来写。

Rust keeps track of these comments, and uses them when generating documentation.This is important when documenting things like enums:

Rust持续跟踪这些注释，并且在生成文档时使用他们。当文档化枚举类型数据时，这是很重要的。

```
/// The `Option` type. See [the module level documentation](../) for more.
enum Option<T> {
    /// No value
    None,
    /// Some value `T`
    Some(T),
}
```

The above works, but this does not:

上面的代码正常运行，但是下面的却不行：

```rust,ignore
/// The `Option` type. See [the module level documentation](../) for more.
enum Option<T> {
    None, /// No value
    Some(T), /// Some value `T`
}
```

You'll get an error:

你会得到一处报错：

```text
hello.rs:4:1: 4:2 error: expected ident, found `}`
hello.rs:4 }
           ^
```

This [unfortunate error](https://github.com/rust-lang/rust/issues/22547) is correct: documentation comments apply to the thing after them, and there's no thing after that last comment.

这个[unfortunate error 不期望的错误](https://github.com/rust-lang/rust/issues/22547)是正确的：文档注释会请求在他们后面的内容，并且在最后一条注释后面没有内容。

[rc-new]: http://doc.rust-lang.org/nightly/std/rc/struct.Rc.html#method.new

### Writing documentation comments  如何写文档注释

Anyway, let's cover each part of this comment in detail:

无论任何情况，让我们详细的了解注释的没一部分：

```
/// Constructs a new `Rc<T>`.
# fn foo() {}
```

The first line of a documentation comment should be a short summary of its
functionality. One sentence. Just the basics. High level.

文档注释的第一行应该是一个功能的简要说明。一句话。只包括基本的内容。高度概括。

```
///
/// Other details about constructing `Rc<T>`s, maybe describing complicated
/// semantics, maybe additional options, all kinds of stuff.
///
# fn foo() {}
```

Our original example had just a summary line, but if we had more things to say,we could have added more explanation in a new paragraph.

我们上面的例子只有一个摘要行，如果我们有很多东西要说，我们我们需要在一个新的段落中增加更多的说明。

#### Special sections  特殊段

```
/// # Examples
# fn foo() {}
```

Next, are special sections. These are indicated with a header, `#`. There
are three kinds of headers that are commonly used. They aren't special syntax,just convention, for now.

下一部分是特殊段。他们使用`#`作为标记。通常有三种标题的用法。现在，这些内容并不是特殊语法，只是惯例。

```
/// # Panics
# fn foo() {}
```

Unrecoverable misuses of a function (i.e. programming errors) in Rust are
usually indicated by panics, which kill the whole current thread at the very least. If your function has a non-trivial contract like this, that is
detected/enforced by panics, documenting it is very important.

在Rust语言中一个函数的滥用（例如 变成错误）通常被标记为`panics`,这至少会杀死整个线程。如果你的函数有一个重要的约定 就像这样 会被`panics`检测红着强制使用，文档化就会变得非常重要。

```
/// # Failures
# fn foo() {}
```

If your function or method returns a `Result<T, E>`, then describing the
conditions under which it returns `Err(E)` is a nice thing to do. This is
slightly less important than `Panics`, because failure is encoded into the type system, but it's still a good thing to do.

如果你的函数或者方法有一个返回值`Result<T, E>`,那么在它返回`Err(E)`前提下描述是一个非常好的事情。相比于`Panics`,这个稍微次要些，因为失败是被编码进类型系统的，但这样做仍然不失为一个好事情。

```
/// # Safety
# fn foo() {}
```

If your function is `unsafe`, you should explain which invariants the caller is responsible for upholding.

如果你的函数是`unsafe`的，你需要说明地哪一个不变量的调用者来负责维护。

```
/// # Examples
///
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
# fn foo() {}
```

Third, `Examples`. Include one or more examples of using your function or
method, and your users will love you for it. These examples go inside of
code block annotations, which we'll talk about in a moment, and can have
more than one section:

第三个是`Examples 例子`。包含一个或者多个你的函数或者方法的使用例子，你的用户将因为这个喜欢你。这些例子将进入到代码块注释中，我们一会会讨论这个，并且他可能有多个部分。


```
/// # Examples
///
/// Simple `&str` patterns:
///
/// ```
/// let v: Vec<&str> = "Mary had a little lamb".split(' ').collect();
/// assert_eq!(v, vec!["Mary", "had", "a", "little", "lamb"]);
/// ```
///
/// More complex patterns with a lambda:
///
/// ```
/// let v: Vec<&str> = "abc1def2ghi".split(|c: char| c.is_numeric()).collect();
/// assert_eq!(v, vec!["abc", "def", "ghi"]);
/// ```
# fn foo() {}
```

Let's discuss the details of these code blocks.

下面我们讨论代码块的详细内容。

#### Code block annotations  代码块注释

To write some Rust code in a comment, use the triple graves:

使用三个“`”可以在注释中写一些Rust代码。

```
/// ```
/// println!("Hello, world");
/// ```
# fn foo() {}
```

If you want something that's not Rust code, you can add an annotation:

如果你想使用其他语言代码，你可以增加一个注释：

```
/// ```c
/// printf("Hello, world\n");
/// ```
# fn foo() {}
```

This will highlight according to whatever language you're showing off.If you're just showing plain text, choose `text`.

这将根据你的语言来显示高亮，如果你只想显示一个纯文本，那么请使用`text`。


It's important to choose the correct annotation here, because `rustdoc` uses it in an interesting way: It can be used to actually test your examples, so that they don't get out of date. If you have some C code but `rustdoc` thinks it's Rust because you left off the annotation, `rustdoc` will complain when trying to generate the documentation.

选择一个恰当的注释是非常重要的，因为`rustdoc`命令用一种有趣的方式来使用它：它被用来实际测试你的例子，所以它们不会过时。如果你试用了c语言代码，因为你没有留下注释，那么`rustdoc`将会默认他为Rust代码，那么当`rustdoc`尝试生成文档的时候，就会报错。

## Documentation as tests  文档即测试

Let's discuss our sample example documentation:

来说一下我们简单例子的文档：

```
/// ```
/// println!("Hello, world");
/// ```
# fn foo() {}
```

You'll notice that you don't need a `fn main()` or anything here.`rustdoc` will automatically add a main() wrapper around your code, and in the right place.For example:

你会注意到，在这里，你不需要一个`fn main()` 或者任何东西。`rustdoc`将自动增加一个 main() 包在你的代码中，并且在正确的位置。例如：

```
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
# fn foo() {}
```

This will end up testing:

这会产生下面的测试：

```
fn main() {
    use std::rc::Rc;
    let five = Rc::new(5);
}
```

Here's the full algorithm rustdoc uses to postprocess examples:

下面是rustdoc使用的处理后的完整算法：

1. Any leading `#![foo]` attributes are left intact as crate attributes.任何前面带有`#![foo]`属性的是保持完整不变的crate属性
2. Some common `allow` attributes are inserted, including `unused_variables`,`unused_assignments`,`unused_mut`,`unused_attributes`, and `dead_code`. Small examples often trigger these lines.包括`unused_variables`, `unused_assignments`, `unused_mut`,`unused_attributes`和`dead_code`的一些允许的属性是可以插入的。小粒子长长出发这些代码。
3. If the example does not contain `extern crate`, then `extern crate <mycrate>;` is inserted.如果例子中不包括`extern crate`,那么`extern crete<mycrate>;`会被插入。
4. Finally, if the example does not contain `fn main`, the remainder of the text is wrapped in `fn main() { your_code }`.最后，如果你的例子没有包含`fn main`,文本剩余的部分将被包裹进`fn main() { your_code }`。

Sometimes, this isn't enough, though. For example, all of these code samples with `///` we've been talking about? The raw text:

然而有时候，这些是不够的。比如，我们说过所有使用`///`的代码？原始文本是：

```text
/// Some documentation.
# fn foo() {}
```

looks different than the output:
看出他们的不同了嘛？

```
/// Some documentation.
# fn foo() {}
```

Yes, that's right: you can add lines that start with `# `, and they will
be hidden from the output, but will be used when compiling your code.You
can use this to your advantage. In this case, documentation comments need
to apply to some kind of function, so if I want to show you just a documentation comment, I need to add a little function definition below
it. At the same time, it's just there to satisfy the compiler, so hiding
it makes the example more clear. You can use this technique to explain
longer examples in detail, while still preserving the testability of your
documentation. For example, this code:

是的，这是正确的：你可以在开始位置使用`#`,他们会在输出时隐藏掉，但在编译代码时会被用到。你可以使用这个作为你的优势。这种情况下，文档注释需要适用于某种功能，所以如果我要告诉你它只是一个文档注释，我需要在下面加点函数定义它。同时，它只是为了满足编译器，所以隐藏它会使例子更加清晰。你可以使用此方法在细节处注释你的例子，同时还能够保证文档的可测试性。例如下面的代码：
```
let x = 5;
let y = 6;
println!("{}", x + y);
```

Here's an explanation, rendered:

这是一个给出的一个注释。

First, we set `x` to five:

首先我们设置`x`的值为5：

```
let x = 5;
# let y = 6;
# println!("{}", x + y);
```

Next, we set `y` to six:

然后我们设置`y`的值为6：

```
# let x = 5;
let y = 6;
# println!("{}", x + y);
```

Finally, we print the sum of `x` and `y`:

最终打印出`x`和`y`的和：

```
# let x = 5;
# let y = 6;
println!("{}", x + y);
```

Here's the same explanation, in raw text:

这原始文本中类似的注释：

> First, we set `x` to five:
>
> ```text
> let x = 5;
> # let y = 6;
> # println!("{}", x + y);
> ```
>
> Next, we set `y` to six:
>
> ```text
> # let x = 5;
> let y = 6;
> # println!("{}", x + y);
> ```
>
> Finally, we print the sum of `x` and `y`:
>
> ```text
> # let x = 5;
> # let y = 6;
> println!("{}", x + y);
> ```

By repeating all parts of the example, you can ensure that your example still compiles, while only showing the parts that are relevant to that part of your explanation.

通过重复例子的所有部分，你能够确定你的代码仍然能够通过编译，同时只显示相关解释说明的部分内容。

### Documenting macros   文档化宏

Here’s an example of documenting a macro:

这是文档化宏的一个例子：

```
/// Panic with a given message unless an expression evaluates to true.
///
/// # Examples
///
/// ```
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(1 + 1 == 2, “Math is broken.”);
/// # }
/// ```
///
/// ```should_panic
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(true == false, “I’m broken.”);
/// # }
/// ```
#[macro_export]
macro_rules! panic_unless {
    ($condition:expr, $($rest:expr),+) => ({ if ! $condition { panic!($($rest),+); } });
}
# fn main() {}
```

You’ll note three things: we need to add our own `extern crate` line, so that we can add the `#[macro_use]` attribute. Second, we’ll need to add our own `main()` as well. Finally, a judicious use of `#` to comment out those two things, so they don’t show up in the output.

你会注意到三点事情：因为我们需要手动增加我们自己的`extern crate`代码，所以我们能够使用`#[macro_use]`属性。第二点，我们同样需要增加自己的`main()`。最后，`#`用来注释以上两点内容，所以他们不会在输出中显示出来。

### Running documentation tests

To run the tests, either

```bash
$ rustdoc --test path/to/my/crate/root.rs
# or
$ cargo test
```

That's right, `cargo test` tests embedded documentation too. However, 
`cargo test` will not test binary crates, only library ones. This is
due to the way `rustdoc` works: it links against the library to be tested,
but with a binary, there’s nothing to link to.

There are a few more annotations that are useful to help `rustdoc` do the right
thing when testing your code:

```
/// ```ignore
/// fn foo() {
/// ```
# fn foo() {}
```

The `ignore` directive tells Rust to ignore your code. This is almost never
what you want, as it's the most generic. Instead, consider annotating it
with `text` if it's not code, or using `#`s to get a working example that
only shows the part you care about.

```
/// ```should_panic
/// assert!(false);
/// ```
# fn foo() {}
```

`should_panic` tells `rustdoc` that the code should compile correctly, but
not actually pass as a test.

```
/// ```no_run
/// loop {
///     println!("Hello, world");
/// }
/// ```
# fn foo() {}
```

The `no_run` attribute will compile your code, but not run it. This is
important for examples such as "Here's how to start up a network service,"
which you would want to make sure compile, but might run in an infinite loop!

### Documenting modules

Rust has another kind of doc comment, `//!`. This comment doesn't document the next item, but the enclosing item. In other words:

```
mod foo {
    //! This is documentation for the `foo` module.
    //!
    //! # Examples

    // ...
}
```

This is where you'll see `//!` used most often: for module documentation. If
you have a module in `foo.rs`, you'll often open its code and see this:

```
//! A module for using `foo`s.
//!
//! The `foo` module contains a lot of useful functionality blah blah blah
```

### Documentation comment style

Check out [RFC 505][rfc505] for full conventions around the style and format of
documentation.

[rfc505]: https://github.com/rust-lang/rfcs/blob/master/text/0505-api-comment-conventions.md

## Other documentation

All of this behavior works in non-Rust source files too. Because comments
are written in Markdown, they're often `.md` files.

When you write documentation in Markdown files, you don't need to prefix
the documentation with comments. For example:

```
/// # Examples
///
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
# fn foo() {}
```

is just

~~~markdown
# Examples

```
use std::rc::Rc;

let five = Rc::new(5);
```
~~~

when it's in a Markdown file. There is one wrinkle though: Markdown files need
to have a title like this:

```markdown
% The title

This is the example documentation.
```

This `%` line needs to be the very first line of the file.

## `doc` attributes

At a deeper level, documentation comments are sugar for documentation attributes:

```
/// this
# fn foo() {}

#[doc="this"]
# fn bar() {}
```

are the same, as are these:

```
//! this

#![doc="/// this"]
```

You won't often see this attribute used for writing documentation, but it
can be useful when changing some options, or when writing a macro.

### Re-exports

`rustdoc` will show the documentation for a public re-export in both places:

```ignore
extern crate foo;

pub use foo::bar;
```

This will create documentation for bar both inside the documentation for the
crate `foo`, as well as the documentation for your crate. It will use the same
documentation in both places.

This behavior can be suppressed with `no_inline`:

```ignore
extern crate foo;

#[doc(no_inline)]
pub use foo::bar;
```

### Controlling HTML

You can control a few aspects of the HTML that `rustdoc` generates through the
`#![doc]` version of the attribute:

```
#![doc(html_logo_url = "http://www.rust-lang.org/logos/rust-logo-128x128-blk-v2.png",
       html_favicon_url = "http://www.rust-lang.org/favicon.ico",
       html_root_url = "http://doc.rust-lang.org/")];
```

This sets a few different options, with a logo, favicon, and a root URL.

## Generation options

`rustdoc` also contains a few other options on the command line, for further customization:

- `--html-in-header FILE`: includes the contents of FILE at the end of the
  `<head>...</head>` section.
- `--html-before-content FILE`: includes the contents of FILE directly after
  `<body>`, before the rendered content (including the search bar).
- `--html-after-content FILE`: includes the contents of FILE after all the rendered content.

## Security note

The Markdown in documentation comments is placed without processing into
the final webpage. Be careful with literal HTML:

```rust
/// <script>alert(document.cookie)</script>
# fn foo() {}
```
