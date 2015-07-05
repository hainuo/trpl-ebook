% Ownership 所有权

This guide is one of three presenting Rust’s ownership system. This is one of Rust’s most unique and compelling features, with which Rust developers should become quite acquainted. Ownership is how Rust achieves its largest goal, memory safety. There are a few distinct concepts, each with its own chapter:

本指引是三个当前Rust的所有权系统之一。这是Rust最特殊，最引人注目的特性，Rust开发者应该对他有一个相当的认知。所有权是Rust如何达成它最大的目标——内存安全的关键特性。这里有一些清晰的概念，每一个都有自己的章节：

* ownership, which you’re reading now  所有权系统  你正在阅读的章节
* [borrowing][borrowing], and their associated feature ‘references’   [借用][borrowing] 它们相关特性的‘引用’
* [lifetimes][lifetimes], an advanced concept of borrowing [生命周期][lifetimes] 借用的一个高级概念

These three chapters are related, and in order. You’ll need all three to fully understand the ownership system.

这三个章节是按照顺序相关联的。你需要它们三个来完全理解所有权系统。

[borrowing]: references-and-borrowing.html
[lifetimes]: lifetimes.html

# Meta  元

Before we get to the details, two important notes about the ownership system.

在我们详细说明之前，有两个关于所有权系统的重要事项。

Rust has a focus on safety and speed. It accomplishes these goals through many ‘zero-cost abstractions’, which means that in Rust, abstractions cost as little as possible in order to make them work. The ownership system is a prime example of a zero-cost abstraction. All of the analysis we’ll talk about in this guide is _done at compile time_. You do not pay any run-time cost for any of these features.

Rust注重安全和速度。它通过许多‘0成本抽象’来达成目标，这意味着在Rust中，抽象花费尽可能少的代价来使他们工作。所有权体系是0成本抽象的一个最佳实践。在本指引中我们要谈论的所有的分析是在 _编译时内完成_ 的。你不需要为这些特性花费任何运行时。

However, this system does have a certain cost: learning curve. Many new users to Rust experience something we like to call ‘fighting with the borrow checker’, where the Rust compiler refuses to compile a program that the author thinks is valid. This often happens because the programmer’s mental model of how ownership should work doesn’t match the actual rules that Rust implements.You probably will experience similar things at first. There is good news, however: more experienced Rust developers report that once they work with the rules of the ownership system for a period of time, they fight the borrow checker less and less.

然而，这样的系统确实需要一定的代价：学习曲线。很多新的Rust体验用户，我们喜欢称之为“与借用检查作战”，Rust编译器拒绝编译一个作者人为是有效的程序的地方。这是经常发生的因为程序的所有权应该运行的推断模型与Rust继承实际规则不匹配。你可能会砸第一次就遇到相似的情况。这是一个好消息，然而：有经验的Rust的开发者报告说：一旦他们开始使用所有权规则一段时间后，他们与借用检查作战的情况越来越少。

With that in mind, let’s learn about ownership.

有了这些概念，我们开始学习所有权。

# Ownership 所有权

[Variable bindings][bindings] have a property in Rust: they ‘have ownership’ of what they’re bound to. This means that when a binding goes out of scope, the resource that they’re bound to are freed. For example:

[变量绑定][bindings]在Rust中有一个属性：他们对被绑定的事物‘拥有所有权’。这意味着当一个变量绑定超出的作用域，被他们绑定的资源将被释放。举个例子：

```rust
fn foo() {
    let v = vec![1, 2, 3];
}
```

When `v` comes into scope, a new [`Vec<T>`][vect] is created. In this case, the vector also allocates space on [the heap][heap], for the three elements. When `v` goes out of scope at the end of `foo()`, Rust will clean up everything
related to the vector, even the heap-allocated memory. This happens deterministically, at the end of the scope.

当`v`进入作用域，一个新的[`Vet<t>`][vect]被创建了。在这个例子中，向量允许为三个元素在[堆][heap]上分配空间.在`foo()`的最后`v`离开作用域，Rust将清理与向量相关联的每一个内容，甚至是堆分配内存。这个确定会发生在作用域结束时。

[vect]: ../std/vec/struct.Vec.html
[heap]: the-stack-and-the-heap.html
[bindings]: variable-bindings.html

# Move semantics  移动语义

There’s some more subtlety here, though: Rust ensures that there is _exactly one_ binding to any given resource. For example, if we have a vector, we can assign it to another binding:

虽然在这里有精妙之处，Rust确保恰好绑定任意给定资源的变量。距离说来，如果我们有一个向量，我们可以把它分配给其他变量绑定。
```rust
let v = vec![1, 2, 3];

let v2 = v;
```

But, if we try to use `v` afterwards, we get an error:

然而，如果在之后我们尝试使用`v`，会报错：

```rust,ignore
let v = vec![1, 2, 3];

let v2 = v;

println!("v[0] is: {}", v[0]);
```

It looks like this:

它像这样样

```text
error: use of moved value: `v`
println!("v[0] is: {}", v[0]);
                        ^
```

A similar thing happens if we define a function which takes ownership, and try to use something after we’ve passed it as an argument:

在我们定义了一个拥有所有权的函数，并且在我们传递它给一个参数后，使用某些内容时，想死的情况会发生：

```rust,ignore
fn take(v: Vec<i32>) {
    // what happens here isn’t important.
}

let v = vec![1, 2, 3];

take(v);

println!("v[0] is: {}", v[0]);
```

Same error: ‘use of moved value’. When we transfer ownership to something else, we say that we’ve ‘moved’ the thing we refer to. You don’t need some sort of special annotation here, it’s the default thing that Rust does.

同样的错误：‘使用了移动的值’。当我们传递所有权给其他内容后，我们说我们已经`转移了`我们引用的内容。你不需要特别的声明，这是Rust的默认的事情。

## The details 细节

The reason that we cannot use a binding after we’ve moved it is subtle, but important. When we write code like this:

我们不能够在我们转移后使用一个变量绑定的原因是非常微妙的，但是非常重要。当我们想这样写代码时：

```rust
let v = vec![1, 2, 3];

let v2 = v;
```

The first line allocates memory for the vector object, `v`, and for the data it contains. The vector object is stored on the [stack][sh] and contains a pointer to the content (`[1, 2, 3]`) stored on the [heap][sh]. When we move `v` to `v2`, it creates a copy of that pointer, for `v2`. Which means that there would be two pointers to the content of the vector on the heap. It would violate Rust’s
safety guarantees by introducing a data race. Therefore, Rust forbids using `v` after we’ve done the move.

第一行给向量对象`v`包括它包含的数据分配了内存。向量对象被存储在[栈][sh]上，并且包含一个指向被存储在[堆][sh]上的内容（`[1,2,3]`）。当我们转移`v`到`v2`时，它为`v2`创建了那个指针的副本。这意味着将有两个指针指向堆上的向量的内容。这因为引入了一个数据竞争，将违犯Rust语言的安全保证。因此，Rust禁止在我们完成转移后使用`v`。

[sh]: the-stack-and-the-heap.html

It’s also important to note that optimizations may remove the actual copy of the bytes on the stack, depending on circumstances. So it may not be as inefficient as it initially seems.

同样需要注意的是，优化操作将视情况而定，移除栈中的字节实际拷贝。所以它可能不像起初那样低效。

## `Copy` types   `Copy`类型

We’ve established that when ownership is transferred to another binding, you cannot use the original binding. However, there’s a [trait][traits] that changes this behavior, and it’s called `Copy`. We haven’t discussed traits yet, but for now, you can think of them as an annotation to a particular type that adds extra behavior. For example:

在所有权被传递给另一个绑定时，我们已经确认，你不能够使用原来的绑定。然而，有一个[特征][traits]可以改变这个行为，它被称为`Copy`。我们还没有讨论过特性，但是现在，你可以认为他们是一个注解，向特定类型增加额外行为。举例说来：

```rust
let v = 1;

let v2 = v;

println!("v is: {}", v);
```

In this case, `v` is an `i32`, which implements the `Copy` trait. This means that, just like a move, when we assign `v` to `v2`, a copy of the data is made.But, unlike a move, we can still use `v` afterward. This is because an `i32` has no pointers to data somewhere else, copying it is a full copy.

在这个案例中，`v`是一个`i32`32字节的整型，这个类型有`Copy`特性。这意味着，当我们将`v`赋值给`v2`的时候，一个数据副本被创建。但是，不像移动，我们仍然可以在移动之后使用`v`。这是因为`i32`没有指针指向其他啊地方的数据，复制它是时完整复制。

We will discuss how to make your own types `Copy` in the [traits][traits] section.

我们将在[特征][traits]章节中讨论，如何制作自己的类型`Copy`。

[traits]: traits.md

# More than ownership   不只是所有权

Of course, if we had to hand ownership back with every function we wrote:

当然，我们不得不为我们写的每一个函数处理所有权返回：

```rust
fn foo(v: Vec<i32>) -> Vec<i32> {
    // do stuff with v

    // hand back ownership
    v
}
```

This would get very tedious. It gets worse the more things we want to take ownership of:

这会变得非常繁琐。我们想要处理内容所有权时，事情变得更早。

```rust
fn foo(v1: Vec<i32>, v2: Vec<i32>) -> (Vec<i32>, Vec<i32>, i32) {
    // do stuff with v1 and v2

    // hand back ownership, and the result of our function
    (v1, v2, 42)
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let (v1, v2, answer) = foo(v1, v2);
```

Ugh! The return type, return line, and calling the function gets way more complicated.

返回类型，返回行，调用函数使得更加复杂。

Luckily, Rust offers a feature, borrowing, which helps us solve this problem.It’s the topic of the next section!

幸运的是，Rust提供了一个特性，借用，它能够帮助我们解决这个问题。这是下一个章节的话题！









