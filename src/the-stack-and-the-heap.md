% The Stack and the Heap 栈和堆
 
As a systems language, Rust operates at a low level. If you’re coming from a
high-level language, there are some aspects of systems programming that you may
not be familiar with. The most important one is how memory works, with a stack
and a heap. If you’re familiar with how C-like languages use stack allocation,
this chapter will be a refresher. If you’re not, you’ll learn about this more
general concept, but with a Rust-y focus.

作为一门系统语言，rust在系统中一个比较低的层次运行。如果你来自于 高级语言，有几个系统编程方面的内容可能你不熟悉。最重要的事情是内存是如何通过栈和堆来进行工作的。如果你熟悉类似c语言的堆栈分配机制，本章将会重温一遍。如果你不熟悉，你将学到更多的通用知识，但是是站在rust语言的角度上

# Memory management 内存管理

These two terms are about memory management. The stack and the heap are
abstractions that help you determine when to allocate and deallocate memory.

有两种内存管理方式，栈和堆是抽象的，用于帮助你决定什么时候分配和释放内存。

Here’s a high-level comparison:  

这里有一个比较好的解释：

The stack is very fast, and is where memory is allocated in Rust by default.
But the allocation is local to a function call, and is limited in size. The
heap, on the other hand, is slower, and is explicitly allocated by your
program. But it’s effectively unlimited in size, and is globally accessible.

栈的管理是非常快的，尤其是在内存有rust语言默认进行内存分配的时候。但是这些分配是局部函数方法调用，且会收到占用空间大小的限制。另一个管理方式堆的管理相对比较就慢多了，并且由你的程序进行明确分配。优点是，它不受限于占用空间的大小，并且能够被全局访问。

# The Stack  栈

Let’s talk about this Rust program:

让我们看看下面的rust程序

```rust
fn main() {
    let x = 42;
}
```

This program has one variable binding, `x`. This memory needs to be allocated
from somewhere. Rust ‘stack allocates’ by default, which means that basic
values ‘go on the stack’. What does that mean?

这个程序有一个变量`x`.内存需要从某一处开始分配。在rust中默认的“栈分配”有一个基本的叫法‘入栈’。那这意味着什么呢？

Well, when a function gets called, some memory gets allocated for all of its local variables and some other information. This is called a ‘stack frame’, and for the purpose of this tutorial, we’re going to ignore the extra information and just consider the local variables we’re allocating. So in this case, when `main()` is run, we’ll allocate a single 32-bit integer for our stack frame.This is automatically handled for you, as you can see, we didn’t have to write any special Rust code or anything.

嗯，当一个函数被调用，一些内存就会被分配给所有的局部变量和一些其他信息。这被称作“栈帧”。介于本教程的意图，我们讲忽略掉一些额外的信息，只考虑所有的局部变量。因此，在这种情况下 当 `main()`运行的时候，我们将会为“栈帧”分配一个32位的整数。这个内存分配是自动处理的，正如你所见到的，我们不需要写任何特殊的Rust代码和做其他事情。

When the function is over, its stack frame gets deallocated. This happens
automatically, we didn’t have to do anything special here.

当一个函数执行结束，他的栈帧就会被回收，这也是自动发生的，我们不需要做任何特殊的操作。

That’s all there is for this simple program. The key thing to understand here is that stack allocation is very, very fast. Since we know all the local variables we have ahead of time, we can grab the memory all at once. And since we’ll throw them all away at the same time as well, we can get rid of it very fast too.

这就是这个简单的程序的全部了。我们需要弄清楚的关键事情就是栈管理操作是非常非常快的。一旦我们提前知道所有的变量，我们就能够一次抓取到所有的内存信息。同样，一旦我们在这个过程中较好的丢掉他们，我们也就能够快速的清除掉它。

The downside is that we can’t keep values around if we need them for longer than a single function. We also haven’t talked about what that name, ‘stack’ means. To do that, we need a slightly more complicated example:

缺点是我们不能够保持变量存在，当我们不只在一个函数中需要他们。我们同样还没有讨论‘stack’到底是什么。为了说明这一个问题，我们需要一个更复杂些的例子。

```rust
fn foo() {
    let y = 5;
    let z = 100;
}

fn main() {
    let x = 42;

    foo();
}
```

This program has three variables total: two in `foo()`, one in `main()`. Just as before, when `main()` is called, a single integer is allocated for its stack frame. But before we can show what happens when `foo()` is called, we need to visualize what’s going on with memory. Your operating system presents a view of memory to your program that’s pretty simple: a huge list of addresses, from 0 to a large number, representing how much RAM your computer has. For example, if you have a gigabyte of RAM, your addresses go from `0` to `1,073,741,824`. That number comes from 2<sup>30</sup>, the number of bytes in a gigabyte.

这个程序总共有三个变量：2个在`foo()`中，一个在`main()`中。就像签一个例子，当`main()`被执行时，一个整数被分配在栈帧上。但是，在我们展示当`foo()`被执行时发生什么前，我们需要想象一下在内存中发生了什么。你的操作系统呈现程序的内存视图是非常简单的：一个从0到一个很大的数字的很长的地址列表用来表示你的电脑中有多少内存空间。例如，如果你有一个1G的内存，你的地址列表将是从`0`到`1,073,741,824`。这个数字是2<sup>30</sup>，1GB所表示的字节数。

This memory is kind of like a giant array: addresses start at zero and go
up to the final number. So here’s a diagram of our first stack frame:

内存像是一个巨大的数组，地址从0开始走到最终数目。所以我们第一个栈帧的图表如下


| Address 地址 | Name 名称 | Value 值 |
|---------|------|-------|
| 0       | x    | 42    |

We’ve got `x` located at address `0`, with the value `42`.
我们在地址`0`处得到`x`,他的值是`42`.

When `foo()` is called, a new stack frame is allocated:

当`foo()`被执行时，一下新的栈帧被分配出来：

| Address 地址 | Name 名称 | Value 值 |
|---------|------|-------|
| 2       | z    | 100   |
| 1       | y    | 5     |
| 0       | x    | 42    |

Because `0` was taken by the first frame, `1` and `2` are used for `foo()`’s stack frame. It grows upward, the more functions we call.

因为`0`被第一帧占用，所以`1`和`2`被用于`foo()`的栈帧。我们调用的函数越多，他增长的越快。

There’s some important things we have to take note of here. The numbers 0, 1, and 2 are all solely for illustrative purposes, and bear no relationship to the actual numbers the computer will actually use. In particular, the series of addresses are in reality going to be separated by some number of bytes that separate each address, and that separation may even exceed the size of the value being stored.

这里有些特别重要的地方需要我们注意。数字0,1和2只是用来表名我们的意图，和电脑中实际使用的真实数目没有任何关系。特别是在实际中，系列中的地址是被一些字节数分离开的，并且有些分离甚至超过了数据被分配的大小。

After `foo()` is over, its frame is deallocated:

在`foo()`执行结束后，他的栈帧被释放了：

| Address | Name | Value |
|---------|------|-------|
| 0       | x    | 42    |

And then, after `main()`, even this last value goes away. Easy!

然后，`main()`执行结束后，连最终的数据都被清空了。真简单!

It’s called a ‘stack’ because it works like a stack of dinner plates: the first
plate you put down is the last plate to pick back up. Stacks are sometimes
called ‘last in, first out queues’ for this reason, as the last value you put
on the stack is the first one you retrieve from it.

它被称之为“栈”是因为它工作起来向是一摞餐盘

Let’s try a three-deep example:

让我们尝试一下三层嵌套的例子：

```rust
fn bar() {
    let i = 6;
}

fn foo() {
    let a = 5;
    let b = 100;
    let c = 1;

    bar();
}

fn main() {
    let x = 42;

    foo();
}
```

Okay, first, we call `main()`:

OK，首先我们执行`main()`:

| Address | Name | Value |
|---------|------|-------|
| 0       | x    | 42    |

Next up, `main()` calls `foo()`:

下一步，`mian()` 调用`foo()`:

| Address | Name | Value |
|---------|------|-------|
| 3       | c    | 1     |
| 2       | b    | 100   |
| 1       | a    | 5     |
| 0       | x    | 42    |

And then `foo()` calls `bar()`:

然后，`foo()` 调用`bar()`:

| Address | Name | Value |
|---------|------|-------|
| 4       | i    | 6     |
| 3       | c    | 1     |
| 2       | b    | 100   |
| 1       | a    | 5     |
| 0       | x    | 42    |

Whew! Our stack is growing tall.
噢，我们的栈长高了。

After `bar()` is over, its frame is deallocated, leaving just `foo()` and
`main()`:
当`bar()`调用结束，他的栈帧被释放，只留下`foo()`和`main()`：

| Address | Name | Value |
|---------|------|-------|
| 3       | c    | 1     |
| 2       | b    | 100   |
| 1       | a    | 5     |
| 0       | x    | 42    |

And then `foo()` ends, leaving just `main()`:

然后 `foo()`运行结束，只剩下`main()`：

| Address | Name | Value |
|---------|------|-------|
| 0       | x    | 42    |

And then we’re done. Getting the hang of it? It’s like piling up dishes: you add to the top, you take away from the top.

然后，运行结束。懂得它的窍门了吗？这就像我们堆放的零食：你放在顶部，然后从顶部拿走。

# The Heap 堆

Now, this works pretty well, but not everything can work like this. Sometimes,you need to pass some memory between different functions, or keep it alive for longer than a single function’s execution. For this, we can use the heap.

现在，它运行的很好，但是并不是所有的事情都能够像那样运行。有些时候，你需要在几个不同的函数中传递内存，或者保持它存在时间比函数执行时间还要长久。为达到这个目的，我们可以使用堆。

In Rust, you can allocate memory on the heap with the [`Box<T>` type][box].

在Rust语言中，你能够以盒模型([`Box<T>` type][box])的方式在堆上分配内存。

Here’s an example:

下面是一个例子

```rust
fn main() {
    let x = Box::new(5);
    let y = 42;
}
```

[box]: ../std/boxed/index.html

Here’s what happens in memory when `main()` is called:

当`main()`被执行时，内存发生如下变化:

| Address | Name | Value  |
|---------|------|--------|
| 1       | y    | 42     |
| 0       | x    | ?????? |

We allocate space for two variables on the stack. `y` is `42`, as it always has been, but what about `x`? Well, `x` is a `Box<i32>`, and boxes allocate memory on the heap. The actual value of the box is a structure which has a pointer to ‘the heap’. When we start executing the function, and `Box::new()` is called,it allocates some memory for the heap, and puts `5` there. The memory now lookslike this:

我们在栈上分配了两个变量的空间。`y`是`42`,它永远都是，但是`x`呢？好吧，`x`是一个盒模型`Box<i32>`, 盒模型分配的内存是在堆上的。盒模型实际上是一个结构体，它有一个指向“堆”的指针。当我们开始执行函数的时候，`Box::new()`被调用了，它在堆上分配了一些内存，并把`5`放在那些内存地址上。现在内存看起来像是这样：

| Address         | Name | Value          |
|-----------------|------|----------------|
| 2<sup>30</sup>  |      | 5              |
| ...             | ...  | ...            |
| 1               | y    | 42             |
| 0               | x    | 2<sup>30</sup> |

We have 2<sup>30</sup> in our hypothetical computer with 1GB of RAM. And since
our stack grows from zero, the easiest place to allocate memory is from the
other end. So our first value is at the highest place in memory. And the value
of the struct at `x` has a [raw pointer][rawpointer] to the place we’ve
allocated on the heap, so the value of `x` is 2<sup>30</sup>, the memory
location we’ve asked for.

在我们假定的电脑的1G内存上我们有2<sup>30</sup>个地址。一旦我们的栈从0开始增长，最容易的方式是从另一端分配内存。所以，我们的第一个值时在内存地址最大的地方。x结构体的值有一个[原始指针][rawpointer] 指向我们在堆上申请的地址，所以x结构体的值是我们申请的内存地址——2<sup>30</sup>。

[rawpointer]: raw-pointers.html

We haven’t really talked too much about what it actually means to allocate and deallocate memory in these contexts. Getting into very deep detail is out of the scope of this tutorial, but what’s important to point out here is that the heap isn’t just a stack that grows from the opposite end. We’ll have an example of this later in the book, but because the heap can be allocated and freed in any order, it can end up with ‘holes’. Here’s a diagram of the memory layout of a program which has been running for a while now:

我们还没有真正探讨实际意义上的上下文内存分配和释放问题。谈论更深层次的细节超出了本书的内容范围，但是需要指出的是，堆并不总是从另一端开始的栈。稍后，本书中会有一个关于这个的例子，但是由于堆可以以任何顺序分配和释放，他可能产生漏洞。这里有一个正在运行的程序的内存层的视图：

| Address              | Name | Value                |
|----------------------|------|----------------------|
| 2<sup>30</sup>       |      | 5                    |
| (2<sup>30</sup>) - 1 |      |                      |
| (2<sup>30</sup>) - 2 |      |                      |
| (2<sup>30</sup>) - 3 |      | 42                   |
| ...                  | ...  | ...                  |
| 3                    | y    | (2<sup>30</sup>) - 3 |
| 2                    | y    | 42                   |
| 1                    | y    | 42                   |
| 0                    | x    | 2<sup>30</sup>       |

In this case, we’ve allocated four things on the heap, but deallocated two of them. There’s a gap between 2<sup>30</sup> and (2<sup>30</sup>) - 3 which isn’t currently being used. The specific details of how and why this happens depends on what kind of strategy you use to manage the heap. Different programs can use different ‘memory allocators’, which are libraries that manage this for you.Rust programs use [jemalloc][jemalloc] for this purpose.

在这种情况下，我们分配了四个结构体在堆上，并且释放了他们中的两个。在2<sup>30</sup> 和 (2<sup>30</sup>) - 3之间有一个间隙，在当前的内存中没有被使用。为什么会产生这么特殊的情况取决于你用何种策略来管理堆。不同的程序使用不同“内存分配器”——就是为你管理堆的库。Rust程序使用[jemalloc][jemalloc] 来达到这一目的。

[jemalloc]: http://www.canonware.com/jemalloc/

Anyway, back to our example. Since this memory is on the heap, it can stay
alive longer than the function which allocates the box. In this case, however,
it doesn’t [^moving] When the function is over, we need to free the stack frame
for `main()`. `Box<T>`, though, has a trick up its sleeve: [Drop][drop]. The
implementation of `Drop` for `Box` deallocates the memory that was allocated
when it was created. Great! So when `x` goes away, it first frees the memory
allocated on the heap:

让我们回到我们的例子中来。一旦内存在堆上，它就能够比分配盒模型空间的函数存在时间更长。然而在这种情况下，在函数运行结束的时候，它不能够移除，我们需要为`main()`.`Box<T>`释放栈帧，不过有一个诀窍来解决他的遗留： [Drop][drop]。当为`Box`执行`Drop`的操作时，释放了在它被创建时分配的内存。强大！所以，当`x`被注销时，首先释放的是在堆上分配的内存空间：

| Address | Name | Value  |
|---------|------|--------|
| 1       | y    | 42     |
| 0       | x    | ?????? |

[drop]: drop.html
[moving]: We can make the memory live longer by transferring ownership,sometimes called ‘moving out of the box’. More complex examples will be covered later.通过转移所有者，我们可以保持更长的内存存活时间，这个有时候被称作“迁出”。稍后会有更多复杂的例子被涉及到。


And then the stack frame goes away, freeing all of our memory.

然后栈帧被移除了，所有的内存都被释放了。

# Arguments and borrowing  参数和引用

We’ve got some basic examples with the stack and the heap going, but what about function arguments and borrowing? Here’s a small Rust program:

我们已经理解了栈和堆管理的基本例子，但是对函数的参数和引用了解到了什么？这里有一个简单的Rust程序：


```rust
fn foo(i: &i32) {
    let z = 42;
}

fn main() {
    let x = 5;
    let y = &x;

    foo(y);
}
```

When we enter `main()`, memory looks like this:

当我们进入`main()`,内存视图看起来是这样子的：

| Address | Name | Value |
|---------|------|-------|
| 1       | y    | 0     |
| 0       | x    | 5     |

`x` is a plain old `5`, and `y` is a reference to `x`. So its value is the memory location that `x` lives at, which in this case is `0`.

`x`被解释为5，`y`是`x`的一个引用。所以`y`的值是`x`存在时在内存中的地址——在本例子中是`0`。

What about when we call `foo()`, passing `y` as an argument?

当我们调用了`foo()`后，将`y`作为参数传递进去后，又是一种什么情况？

| Address | Name | Value |
|---------|------|-------|
| 3       | z    | 42    |
| 2       | i    | 0     |
| 1       | y    | 0     |
| 0       | x    | 5     |

Stack frames aren’t just for local bindings, they’re for arguments too. So in this case, we need to have both `i`, our argument, and `z`, our local variable binding. `i` is a copy of the argument, `y`. Since `y`’s value is `0`, so is `i`’s.

栈帧并不只是分配给局部变量的，它们也会为参数分配的。所以在本案例中，我们需要对`i`——参数，和`z`——本地局部变量分配内存栈帧

This is one reason why borrowing a variable doesn’t deallocate any memory: the value of a reference is just a pointer to a memory location. If we got rid of the underlying memory, things wouldn’t work very well.

这就是为什么引用一个变量不会释放任何内存的一个原因：引用的值只是指向一个内存地址的指针。如果我们脱离了基础的内存，程序将不能够很好的运行。

# A complex example  一个复杂的例子

Okay, let’s go through this complex program step-by-step:

好了，让我们一步步进入这个复杂的程序

```rust
fn foo(x: &i32) {
    let y = 10;
    let z = &y;

    baz(z);
    bar(x, z);
}

fn bar(a: &i32, b: &i32) {
    let c = 5;
    let d = Box::new(5);
    let e = &d;

    baz(e);
}

fn baz(f: &i32) {
    let g = 100;
}

fn main() {
    let h = 3;
    let i = Box::new(20);
    let j = &h;

    foo(j);
}
```

First, we call `main()`:

首先，我们调用`main()`：

| Address         | Name | Value          |
|-----------------|------|----------------|
| 2<sup>30</sup>  |      | 20             |
| ...             | ...  | ...            |
| 2               | j    | 0              |
| 1               | i    | 2<sup>30</sup> |
| 0               | h    | 3              |

We allocate memory for `j`, `i`, and `h`. `i` is on the heap, and so has a value pointing there.

我们为`j`,`i`和`h`分配了内存。`i`被分配在堆上，所以有一个指针在在那里。

Next, at the end of `main()`, `foo()` gets called:

下一步，在`main()`的最后,`foo()`被调用了：

| Address         | Name | Value          |
|-----------------|------|----------------|
| 2<sup>30</sup>  |      | 20             |
| ...             | ...  | ...            |
| 5               | z    | 4              |
| 4               | y    | 10             |
| 3               | x    | 0              |
| 2               | j    | 0              |
| 1               | i    | 2<sup>30</sup> |
| 0               | h    | 3              |

Space gets allocated for `x`, `y`, and `z`. The argument `x` has the same value as `j`, since that’s what we passed it in. It’s a pointer to the `0` address,since `j` points at `h`.

内存空间被分配给`x`,`y`和`z`。自我们将`j`传递进来后，参数`x`有一个与`j`相同的值。它是指向编号为`0`的内存地址，因为`j`指向`h`。

Next, `foo()` calls `baz()`, passing `z`:

然后，`foo()`调用`baz()`，传参`z`：

| Address         | Name | Value          |
|-----------------|------|----------------|
| 2<sup>30</sup>  |      | 20             |
| ...             | ...  | ...            |
| 7               | g    | 100            |
| 6               | f    | 4              |
| 5               | z    | 4              |
| 4               | y    | 10             |
| 3               | x    | 0              |
| 2               | j    | 0              |
| 1               | i    | 2<sup>30</sup> |
| 0               | h    | 3              |

We’ve allocated memory for `f` and `g`. `baz()` is very short, so when it’s over, we get rid of its stack frame:

我们为`f`和`g`分配了内存。函数`baz()`非常短，所以当他结束的时候，我们清除了它的栈帧。

| Address         | Name | Value          |
|-----------------|------|----------------|
| 2<sup>30</sup>  | ...  | 20             |
| ...             | ...  | ...            |
| 5               | z    | 4              |
| 4               | y    | 10             |
| 3               | x    | 0              |
| 2               | j    | 0              |
| 1               | i    | 2<sup>30</sup> |
| 0               | h    | 3              |

Next, `foo()` calls `bar()` with `x` and `z`:

然后是，`foo()`调用了`bar()`，使用`x`和`z`作为参数：

| Address              | Name | Value                |
|----------------------|------|----------------------|
|  2<sup>30</sup>      | ...  | 20                   |
| (2<sup>30</sup>) - 1 | ...  | 5                    |
| ...                  | ...  | ...                  |
| 10                   | e    | 9                    |
| 9                    | d    | (2<sup>30</sup>) - 1 |
| 8                    | c    | 5                    |
| 7                    | b    | 4                    |
| 6                    | a    | 0                    |
| 5                    | z    | 4                    |
| 4                    | y    | 10                   |
| 3                    | x    | 0                    |
| 2                    | j    | 0                    |
| 1                    | i    | 2<sup>30</sup>       |
| 0                    | h    | 3                    |

We end up allocating another value on the heap, and so we have to subtract one from 2<sup>30</sup>. It’s easier to just write that than `1,073,741,823`. In any case, we set up the variables as usual.

我们不能够在堆上分配另一个值，所以我们必须在内存地址2<sup>30</sup>-1处分配。这么标记只是因为它比`1,073,741,823`好写。无论任何情况，我们像平常一样建立了变量。

At the end of `bar()`, it calls `baz()`:

在函数`bar()`的最后，它调用了函数`baz()`：

| Address              | Name | Value                |
|----------------------|------|----------------------|
|  2<sup>30</sup>      |      | 20                   |
| (2<sup>30</sup>) - 1 |      | 5                    |
| ...                  | ...  | ...                  |
| 12                   | g    | 100                  |
| 11                   | f    | 4                    |
| 10                   | e    | 9                    |
| 9                    | d    | (2<sup>30</sup>) - 1 |
| 8                    | c    | 5                    |
| 7                    | b    | 4                    |
| 6                    | a    | 0                    |
| 5                    | z    | 4                    |
| 4                    | y    | 10                   |
| 3                    | x    | 0                    |
| 2                    | j    | 0                    |
| 1                    | i    | 2<sup>30</sup>       |
| 0                    | h    | 3                    |

With this, we’re at our deepest point! Whew! Congrats for following along this far.

通过这个，我们到达了最深层！恭喜你跟随着走了这么远。

After `baz()` is over, we get rid of `f` and `g`:

`baz()`结束后，我们清除了`f`和`g`：

| Address              | Name | Value                |
|----------------------|------|----------------------|
|  2<sup>30</sup>      |      | 20                   |
| (2<sup>30</sup>) - 1 |      | 5                    |
| ...                  | ...  | ...                  |
| 10                   | e    | 9                    |
| 9                    | d    | (2<sup>30</sup>) - 1 |
| 8                    | c    | 5                    |
| 7                    | b    | 4                    |
| 6                    | a    | 0                    |
| 5                    | z    | 4                    |
| 4                    | y    | 10                   |
| 3                    | x    | 0                    |
| 2                    | j    | 0                    |
| 1                    | i    | 2<sup>30</sup>       |
| 0                    | h    | 3                    |

Next, we return from `bar()`. `d` in this case is a `Box<T>`, so it also frees
what it points to: (2<sup>30</sup>) - 1.

然后，我们返回到函数`bar()`。在本案例中`d` 是一个盒模型`Box<T>`,所以它指向的内存地址(2<sup>30</sup>) - 1 同样被释放了。

| Address         | Name | Value          |
|-----------------|------|----------------|
|  2<sup>30</sup> |      | 20             |
| ...             | ...  | ...            |
| 5               | z    | 4              |
| 4               | y    | 10             |
| 3               | x    | 0              |
| 2               | j    | 0              |
| 1               | i    | 2<sup>30</sup> |
| 0               | h    | 3              |

And after that, `foo()` returns:

之后，返回到了函数`foo()`：

| Address         | Name | Value          |
|-----------------|------|----------------|
|  2<sup>30</sup> |      | 20             |
| ...             | ...  | ...            |
| 2               | j    | 0              |
| 1               | i    | 2<sup>30</sup> |
| 0               | h    | 3              |

And then, finally, `main()`, which cleans the rest up. When `i` is `Drop`ped,it will clean up the last of the heap too.

最终，`main()`开始清理释放。当`i`被`Drop`的时候，它同样将清理掉最后的堆。

# What do other languages do?  其他的语言是怎么做的？

Most languages with a garbage collector heap-allocate by default. This means that every value is boxed. There are a number of reasons why this is done, but they’re out of scope for this tutorial. There are some possible optimizations that don’t make it true 100% of the time, too. Rather than relying on the stack and `Drop` to clean up memory, the garbage collector deals with the heap instead.

大多数的语言默认有一个垃圾回收器。这意味着，每一个值都是一个盒模型。这样做有许多原因，他们超出了本教程的范围，故不在此赘述。同样，这些可能的优化并不总是保证100%正确。但是垃圾回收器使用堆管理而不是依赖栈和“Drop”来清理内存。

# Which to use?  到底该使用哪一种

So if the stack is faster and easier to manage, why do we need the heap? A big reason is that Stack-allocation alone means you only have LIFO semantics for reclaiming storage. Heap-allocation is strictly more general, allowing storage to be taken from and returned to the pool in arbitrary order, but at a
complexity cost.

那么尽管栈操作比较快，且更容易管理，我们为什么还需要堆？一个很大方面的原因是栈分配意味着你只有先进后出，后进先出的方式来声明存储空间。堆分配更加通用一些，允许内存空间以任意顺序方式分配和返回到内存池，但是这需要付出一个复杂度大小的代价。

Generally, you should prefer stack allocation, and so, Rust stack-allocates by default. The LIFO model of the stack is simpler, at a fundamental level. This has two big impacts: runtime efficiency and semantic impact.

通常，我们选择栈分配，同样Rust语言也是默认使用栈分配。在系统基础层面上，栈的先进后出的方式是比较简单的。这会产生两个方面的影响：执行效率和语义对撞。

## Runtime Efficiency. 

Managing the memory for the stack is trivial: The machine just increments or decrements a single value, the so-called “stack pointer”.Managing memory for the heap is non-trivial: heap-allocated memory is freed at
arbitrary points, and each block of heap-allocated memory can be of arbitrary size, the memory manager must generally work much harder to identify memory for reuse.

栈的内存管理是琐碎的：机器只能够递增或者递减一个值，这个被称作“堆栈指针”。堆的内存管理是不琐碎的：堆分配内存可以被随意的释放，并且堆分配内存的每一块都可以是任意大小，内存管理者必须费力运行，以保证内存确认可用的内存被重新使用

If you’d like to dive into this topic in greater detail, [this paper][wilson] is a great introduction.

如果你想喜欢本话题的更深层次，[this paper][wilson] 是一个非常不错的介绍。

[wilson]: http://www.cs.northwestern.edu/~pdinda/icsclass/doc/dsa.pdf

## Semantic impact 

Stack-allocation impacts the Rust language itself, and thus the developer’s mental model. The LIFO semantics is what drives how the Rust language handles automatic memory management. Even the deallocation of a uniquely-owned heap-allocated box can be driven by the stack-based LIFO semantics, as discussed throughout this chapter. The flexibility (i.e. expressiveness) of non LIFO-semantics means that in general the compiler cannot automatically infer at compile-time where memory should be freed; it has to rely on dynamic protocols,potentially from outside the language itself, to drive deallocation (reference
counting, as used by `Rc<T>` and `Arc<T>`, is one example of this).

栈分配影响着Rust语言本身和开发者的心智模型。后进先出的哲学驱动着Rust语言如何自动进行内存管理。甚至于就像本章中探讨的，独有的堆分配也是被基于栈分配的后进先出的哲学来驱动的。非后进先出的灵活性（即表现）是指在编译器在编译过程中无法自动推断某些内存应该被释放时，它不得不依赖于来字语言本身之外的动态协议，来驱动释放操作——参考：`Rc<T>`和`Arc<T>`的用法就是一个关于这方面的例子。

When taken to the extreme, the increased expressive power of heap allocation comes at the cost of either significant runtime support (e.g. in the form of a garbage collector) or significant programmer effort (in the form of explicit memory management calls that require verification not provided by the Rust compiler).

当到了极致的时候，随着不断增加的堆分配的表现力来自于每一个重要运行时支持的成本（比如垃圾回收器的形式）或者重要成员（在Rust编译器提供的方法之外的明确的内存管理形式上）所花费的精力。