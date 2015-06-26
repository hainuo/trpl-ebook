% Comments  注释

Now that we have some functions, it’s a good idea to learn about comments.Comments are notes that you leave to other programmers to help explain things about your code. The compiler mostly ignores them.

我们有了几个函数，了解注释是一个好主意。注释是标注在你将代码给其他程序员后，帮助他们理解你的代码的东西。编译器大都忽略他们。

Rust has two kinds of comments that you should care about: *line comments*
and *doc comments*.

Rust有两种你需要关注的注释类型，*行注释*和*块注释*。

```rust
// Line comments are anything after ‘//’ and extend to the end of the line.

let x = 5; // this is also a line comment.

// If you have a long explanation for something, you can put line comments next
// to each other. Put a space between the // and your comment so that it’s
// more readable.
```

The other kind of comment is a doc comment. Doc comments use `///` instead of `//`, and support Markdown notation inside:

另一种注释类是是块注释。块注释使用`///`替代`//`，并且支持Markdown符号

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let five = 5;
///
/// assert_eq!(6, add_one(5));
/// ```
fn add_one(x: i32) -> i32 {
    x + 1
}
```

When writing doc comments, providing some examples of usage is very, very helpful. You’ll notice we’ve used a new macro here: `assert_eq!`. This compares two values, and `panic!`s if they’re not equal to each other. It’s very helpful in documentation. There’s another macro, `assert!`, which `panic!`s if the value passed to it is `false`.

当些块注释时，提供一些应用示例是非常非常有帮助的。你会注意到我们在这里使用了一个新的宏：`assert_eq!`。它对比两个值，并且在他们相互见不相等时产生`panic!`。在文档中它是非常有用的。这里还有另一个宏，`assert!`，如果传递给它的值是`false`，就会`panic!`。

You can use the [`rustdoc`](documentation.md) tool to generate HTML documentation from these doc comments, and also to run the code examples as tests!

你可以使用[`rustdoc`](documentation.md)工具从这些块注释来生成html文档注释，同样可以运行的代码例子作为测试！
