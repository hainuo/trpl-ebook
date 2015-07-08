% References and Borrowing 地址引用和借用

This guide is one of three presenting Rust’s ownership system. This is one of Rust’s most unique and compelling features, with which Rust developers should
become quite acquainted. Ownership is how Rust achieves its largest goal,memory safety. There are a few distinct concepts, each with its own chapter:

本指引是三个当前Rust的所有权系统之一。这是Rust最特殊，最引人注目的特性，Rust开发者应该对他有一个相当的认知。所有权是Rust如何达成它最大的目标——内存安全的关键特性。这里有一些清晰的概念，每一个都有自己的章节：

* [ownership][ownership], the key concept   [所有权][ownership]关键概念
* borrowing, which you’re reading now      [借用]你正在阅读的章节
* [lifetimes][lifetimes], an advanced concept of borrowing [生命周期][lifetimes] 一个借用的高级概念

These three chapters are related, and in order. You’ll need all three to fully
understand the ownership system.

这三个章节是按照顺序相关联的。你需要它们三个来完全理解所有权系统。

[ownership]: ownership.html
[lifetimes]: lifetimes.html

# Meta  元

Before we get to the details, two important notes about the ownership system.

在我们详细说明之前，有两个关于所有权系统的重要事项。

Rust has a focus on safety and speed. It accomplishes these goals through many ‘zero-cost abstractions’, which means that in Rust, abstractions cost as little
as possible in order to make them work. The ownership system is a prime example of a zero cost abstraction. All of the analysis we’ll talk about in this guide
is _done at compile time_. You do not pay any run-time cost for any of these features.

Rust注重安全和速度。它通过许多‘0成本抽象’来达成目标，这意味着在Rust中，抽象花费尽可能少的代价来使他们工作。所有权体系是0成本抽象的一个最佳实践。在本指引中我们要谈论的所有的分析是在 _编译时内完成_ 的。你不需要为这些特性花费任何运行时。

However, this system does have a certain cost: learning curve. Many new users to Rust experience something we like to call ‘fighting with the borrow checker’, where the Rust compiler refuses to compile a program that the author thinks is valid. This often happens because the programmer’s mental model of how ownership should work doesn’t match the actual rules that Rust implements.You probably will experience similar things at first. There is good news,however: more experienced Rust developers report that once they work with the rules of the ownership system for a period of time, they fight the borrow checker less and less.

然而，这样的系统确实需要一定的代价：学习曲线。很多新的Rust体验用户，我们喜欢称之为“与借用检查作战”，Rust编译器拒绝编译一个作者人为是有效的程序的地方。这是经常发生的因为程序的所有权应该运行的推断模型与Rust继承实际规则不匹配。你可能会砸第一次就遇到相似的情况。这是一个好消息，然而：有经验的Rust的开发者报告说：一旦他们开始使用所有权规则一段时间后，他们与借用检查作战的情况越来越少。

With that in mind, let’s learn about borrowing.

有了这些概念，我们开始学习借用。


# Borrowing  借用

At the end of the [ownership][ownership] section, we had a nasty function that looked
like this:

在[所有权][ownership]章节的最后，我们有一个看起来像这样子的讨厌的函数：

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

This is not idiomatic Rust, however, as it doesn’t take advantage of borrowing. Here’s
the first step:

```rust
fn foo(v1: &Vec<i32>, v2: &Vec<i32>) -> i32 {
    // do stuff with v1 and v2

    // return the answer
    42
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let answer = foo(&v1, &v2);

// we can use v1 and v2 here!
```

Instead of taking `Vec<i32>`s as our arguments, we take a reference:`&Vec<i32>`. And instead of passing `v1` and `v2` directly, we pass `&v1` and `&v2`. We call the `&T` type a ‘reference’, and rather than owning the resource,it borrows ownership. A binding that borrows something does not deallocate the resource when it goes out of scope. This means that after the call to `foo()`,we can use our original bindings again.

我们使用了一个地址引用`&Vec<i32>`，而不是`Vec<i32>`。我们传递`&v1`给`&v2`而不是直接传递`v1`给`v2`。我们称`&T`类型为一个‘地址引用’，不止是拥有了资源，它还借来了所有权。一个借用类型的变量绑定，在它离开作用域时，不需要释放资源。这意味着，在调用`foo()`后，我们可以再次使用我们原来的变量绑定。

References are immutable, just like bindings. This means that inside of `foo()`,the vectors can’t be changed at all:

引用是不可改变的，就像变量绑定。这意味着在`foo()`的内部，向量根本不可能被改变：

```rust,ignore
fn foo(v: &Vec<i32>) {
     v.push(5);
}

let v = vec![];

foo(&v);
```

errors with:

错误：

```text
error: cannot borrow immutable borrowed content `*v` as mutable
v.push(5);
^
```

Pushing a value mutates the vector, and so we aren’t allowed to do it.

推送一个值会改变向量，所以我们不允许这样做。


# &mut references   可变的地址引用

There’s a second kind of reference: `&mut T`. A ‘mutable reference’ allows you to mutate the resource you’re borrowing. For example:

有另一种形式的地址引用：`&mut T`。 一个“可变的地址引用”允许你改变你借用的资源。例如：

```rust
let mut x = 5;
{
    let y = &mut x;
    *y += 1;
}
println!("{}", x);
```

This will print `6`. We make `y` a mutable reference to `x`, then add one to the thing `y` points at. You’ll notice that `x` had to be marked `mut` as well,if it wasn’t, we couldn’t take a mutable borrow to an immutable value.

这回打印`6`。我们设定`y`为一个`x`的可变类型的地址引用，并对`y`指向的内容增加1。你讲注意到`x`同样必须被标记为`mut`，如果它不是，我们不可能让一个可变的借用指向一个不可变的值。

Otherwise, `&mut` references are just like references. There _is_ a large difference between the two, and how they interact, though. You can tell something is fishy in the above example, because we need that extra scope, with the `{` and `}`. If we remove them, we get an error:

否则，`&mut`地址引用就像是地址引用。然而，两者之间和他们之间如何交互 _是_ 有很大不同。你可以说上个例子中某些内容不是个味儿，因为我们需要使用`{`和`}`来扩展作用域。如果我们移除它们，我们会得到一个报错：

```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
note: previous borrow of `x` occurs here; the mutable borrow prevents
subsequent moves, borrows, or modification of `x` until the borrow ends
        let y = &mut x;
                     ^
note: previous borrow ends here
fn main() {

}
^
```

As it turns out, there are rules.

事实证明，确实是有规则的。

# The Rules  规则

Here’s the rules about borrowing in Rust:

在Rust中有关于借用的规则：

First, any borrow must last for a smaller scope than the owner. Second, you may have one or the other of these two kinds of borrows, but not both at the same time:

首先，任何借用持续存在的作用域必须比owner的作用域更小。第二，你必须有两种借用类型之一，但是它们不能够同时存在：

* 0 to N references (`&T`) to a resource.  0到n引用地址(`&T`)指向一个资源
* exactly one mutable reference (`&mut T`)  明确一个可变类型的地址引用（`&mut T`）


You may notice that this is very similar, though not exactly the same as, to the definition of a data race:

你可能会注意到，这非常相似，尽管不是完全相似，来定义一个数据竞争：

> There is a ‘data race’ when two or more pointers access the same memory location at the same time, where at least one of them is writing, and the operations are not synchronized.
> 当两个或者多个指针在同一时间存取相同内存位置时，其中至少有一个会写入，并且操作不会被同步，这就是一个“数据竞争”。

With references, you may have as many as you’d like, since none of them are writing. If you are writing, you need two or more pointers to the same memory,and you can only have one `&mut` at a time. This is how Rust prevents data races at compile time: we’ll get errors if we break the rules.

通过使用地址引用，你会拥有比你想要的还要多德，因为他们不需要写入。如果你要写入，你需要两个或者更多个指向相同内存的指针，并且你只有一个`&mut`在同一时间。这正是Rust在编译时内阻止数据竞争的原因：如果我们打破了规则我们将得到报错。

With this in mind, let’s consider our example again.

有了这个概念，让我们再次注视下我们的例子。

## Thinking in scopes 作用域思想

Here’s the code:

下面是代码：

```rust,ignore
let mut x = 5;
let y = &mut x;

*y += 1;

println!("{}", x);
```

This code gives us this error:

这段代码给出报错


```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
```

This is because we’ve violated the rules: we have a `&mut T` pointing to `x`,and so we aren’t allowed to create any `&T`s. One or the other. The note hints at how to think about this problem:

这是因为我们违反了规则：我们有一个指向`x`的指针`&mut T`,所以我们不被允许创建任何`&T`。一个或者另一个。注释暗示着如何看待这个问题：

```text
note: previous borrow ends here
fn main() {

}
^
```

In other words, the mutable borow is held through the rest of our example. What we want is for the mutable borrow to end _before_ we try to call `println!` and make an immutable borrow. In Rust, borrowing is tied to the scope that the borrow is valid for. And our scopes look like this:

换句话说，可变的借用视同例子中剩余部分来保持的。对于可变的借用，我们想要的是在我们尝试调用`println!`，和生成一个不可变的借用 _之前_ 结束。在Rust中，借用是被置于一个对借用是有效的作用域中。作用域看起来像这样：

```rust,ignore
let mut x = 5;

let y = &mut x;    // -+ &mut borrow of x starts here
                   //  |
*y += 1;           //  |
                   //  |
println!("{}", x); // -+ - try to borrow x here
                   // -+ &mut borrow of x ends here
```

The scopes conflict: we can’t make an `&x` while `y` is in scope.

作用域冲冲突：我们不能够在`y`的作用域内创建一个`&x`.

So when we add the curly braces:

所以在这个时候我们增加了花括号：

```rust
let mut x = 5;

{                   
    let y = &mut x; // -+ &mut borrow starts here
    *y += 1;        //  |
}                   // -+ ... and ends here

println!("{}", x);  // <- try to borrow x here
```

There’s no problem. Our mutable borrow goes out of scope before we create an immutable one. But scope is the key to seeing how long a borrow lasts for.

没有任何问题。在我们创建一个不可变借用时，可变的借用已经离开作用域。但是作用域是一个借用能够持续多久的关键。

## Issues borrowing prevents  借用防止的问题

Why have these restrictive rules? Well, as we noted, these rules prevent data races. What kinds of issues do data races cause? Here’s a few.

为什么要有这些限制性规则呢？我们注意到，这些规则阻止了数据竞争。数据竞争会引起哪些问题？这里有些例子。

### Iterator invalidation   迭代器失效

One example is ‘iterator invalidation’, which happens when you try to mutate a collection that you’re iterating over. Rust’s borrow checker prevents this from happening:

一个例子是“迭代器失效”，这发生在当我们试图改变一个正在遍历的集合时。Rust的借用检查阻止这种情况发生：

```rust
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
}
```

This prints out one through three. As we iterate through the vectors, we’re only given references to the elements. And `v` is itself borrowed as immutable,which means we can’t change it while we’re iterating:

这将打印出三个中的一个。当我们遍历向量时，我们只是将地址引用传递给元素。`v`是它本身的借用作为不可变变量，这意味着当我们正在遍历时，我们不能够改变它：


```rust,ignore
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
    v.push(34);
}
```

Here’s the error:

错误：

```text
error: cannot borrow `v` as mutable because it is also borrowed as immutable
    v.push(34);
    ^
note: previous borrow of `v` occurs here; the immutable borrow prevents
subsequent moves or mutable borrows of `v` until the borrow ends
for i in &v {
          ^
note: previous borrow ends here
for i in &v {
    println!(“{}”, i);
    v.push(34);
}
^
```

We can’t modify `v` because it’s borrowed by the loop.

我们不能够修改`v`因为它被循环借用者。

### use after free   释放后使用

References must live as long as the resource they refer to. Rust will check the scopes of your references to ensure that this is true.

地址引用必须跟他们引用的资源的存活期一样。Rust将检查你引用的作用域来确定这是正确的。

If Rust didn’t check that this property, we could accidentally use a reference which was invalid. For example:

如果Rust不检查这个属性，我们可能会使用一个失效的地址引用。例如：

```rust,ignore
let y: &i32;
{ 
    let x = 5;
    y = &x;
}

println!("{}", y);
```

We get this error:

我们得到错误：

```text
error: `x` does not live long enough
    y = &x;
         ^
note: reference must be valid for the block suffix following statement 0 at 2:16...
let y: &i32;
{ 
    let x = 5;
    y = &x;
}

note: ...but borrowed value is only valid for the block suffix following statement 0 at 4:18
    let x = 5;
    y = &x;
}
```

In other words, `y` is only valid for the scope where `x` exists. As soon as `x` goes away, it becomes invalid to refer to it. As such, the error says that the borrow ‘doesn’t live long enough’ because it’s not valid for the right amount of time.

换句话说，当`x`存在时，`y`对作用于来说是有效的。一旦`x`消失，对于引用他的来说将是失效的。正是正阳，错误告我们借用“没有足够的存活时间”，因为它在此刻是失效的。

The same problem occurs when the reference is declared _before_ the variable it refers to:

相同的问题发生 _在_ 他引用的变量 _之前_ 声明地址引用时：

```rust,ignore
let y: &i32;
let x = 5;
y = &x;

println!("{}", y);
```

We get this error:

```text
error: `x` does not live long enough
y = &x;
     ^
note: reference must be valid for the block suffix following statement 0 at
2:16...
    let y: &i32;
    let x = 5;
    y = &x;
    
    println!("{}", y);
}

note: ...but borrowed value is only valid for the block suffix following
statement 1 at 3:14
    let x = 5;
    y = &x;
    
    println!("{}", y);
}
```
