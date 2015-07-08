% while Loops  while循环

Rust also has a `while` loop. It looks like this:

Rust同样有`while`循环，它看起来这样：

```{rust}
let mut x = 5; // mut x: i32
let mut done = false; // mut done: bool

while !done {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 {
        done = true;
    }
}
```

`while` loops are the correct choice when you’re not sure how many times you need to loop.

`while`循环是当你不确定你需要循环多少次时的正确的选择。

If you need an infinite loop, you may be tempted to write this:

如果你需要无限循环，你可能视图这样写：

```rust,ignore
while true {
```

However, Rust has a dedicated keyword, `loop`, to handle this case:

然而，Rust有一个用来处理这个案例的专门的关键词，`loop`:

```rust,ignore
loop {
```

Rust’s control-flow analysis treats this construct differently than a `while true`, since we know that it will always loop. In general, the more information we can give to the compiler, the better it can do with safety and code generation, so you should always prefer `loop` when you plan to loop
infinitely.

Rust的控制流分析系统对待这个结构不同于`while true`，因为我们知道它总是循环。通常情况下，我们能够给到编译器的信息越多，编译器安全操作和代码生成就做的越好，所以，当你计划使用无限循环时，你应该总是偏爱`loop`。

## Ending iteration early  及早结束迭代

Let’s take a look at that `while` loop we had earlier:

让我们看一下前面已经有的`while`循环：

```rust
let mut x = 5;
let mut done = false;

while !done {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 {
        done = true;
    }
}
```

We had to keep a dedicated `mut` boolean variable binding, `done`, to know when we should exit out of the loop. Rust has two keywords to help us with modifying iteration: `break` and `continue`.

我们不得不保持一个专门的`mut`布尔变量绑定，`done`，来确认什么时候我们应该跳出循环。Rist有两个关键词来帮助我们修改迭代：`break`和`continue`。

In this case, we can write the loop in a better way with `break`:

在这个案例中，我们可以使用`break`以一种更好的方式写这个循环：

```rust
let mut x = 5;

loop {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 { break; }
}
```

We now loop forever with `loop` and use `break` to break out early.

我们可以使用`loop`永远循环，并且使用`break`来及早打断跳出循环。

`continue` is similar, but instead of ending the loop, goes to the next iteration. This will only print the odd numbers:

`continu` 类似，然而不同于结束循环，它会进入到下次迭代中。这将只打印偶数：

```rust
for x in 0..10 {
    if x % 2 == 0 { continue; }

    println!("{}", x);
}
```

Both `continue` and `break` are valid in both `while` loops and [`for` loops][for].

`continue`和`break`对`while`循环和[`for`循环][for]同样有效。

[for]: for-loops.html
