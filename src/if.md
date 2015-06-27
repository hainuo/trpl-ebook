% if  if语句

Rust’s take on `if` is not particularly complex, but it’s much more like the `if` you’ll find in a dynamically typed language than in a more traditional systems language. So let’s talk about it, to make sure you grasp the nuances.

Rust对`if`的处理并不是特别的复杂，但是比起传统的系统语言，你会发现它更像动态类型语言中的`if`。所以我们来讨论它吧，以确保你掌握细节。

`if` is a specific form of a more general concept, the ‘branch’. The name comes from a branch in a tree: a decision point, where depending on a choice,
multiple paths can be taken.

`if` 是一个广泛概念“分支”的一个明确的形式。这个名字来自于一个树状分支，依赖于一个选择点，有多条路径可以选择。

In the case of `if`, there is one choice that leads down two paths:

在`if`案例中，每一个选择都会引出亮条路径

```rust
let x = 5;

if x == 5 {
    println!("x is five!");
}
```

If we changed the value of `x` to something else, this line would not print.More specifically, if the expression after the `if` evaluates to `true`, then
the block is executed. If it’s `false`, then it is not.

如果我们改变了`x`的值，这一行就不会被打印。更确切的说，如果`if`后面的表达式等于`true`，代码块就被执行，如果它是`false`，就不会执行。

If you want something to happen in the `false` case, use an `else`:

如果你你想在`false`情况下发生一些事情，请使用`else`：

```rust
let x = 5;

if x == 5 {
    println!("x is five!");
} else {
    println!("x is not five :(");
}
```

If there is more than one case, use an `else if`:

如果多余一个条件，请使用`else if`：

```rust
let x = 5;

if x == 5 {
    println!("x is five!");
} else if x == 6 {
    println!("x is six!");
} else {
    println!("x is not five or six :(");
}
```

This is all pretty standard. However, you can also do this:

这是最完美的标准的，然而你也可以这样做：

```rust
let x = 5;

let y = if x == 5 {
    10
} else {
    15
}; // y: i32
```

Which we can (and probably should) write like this:

我们可以（并且可能应该）这样写：

```rust
let x = 5;

let y = if x == 5 { 10 } else { 15 }; // y: i32
```

This works because `if` is an expression. The value of the expression is the value of the last expression in whichever branch was chosen. An `if` without an
`else` always results in `()` as the value.

这样做是因为`if`是一个表达式。表达式的值是被选择的分支的最后的表达式的值。一个没有`else`的`if`总是会导致以`()`作为值。
