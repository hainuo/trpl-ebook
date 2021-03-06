% The Rust Programming Language Rust编程语言

Welcome! This book will teach you about the [Rust Programming Language][rust].Rust is a systems programming language focused on three goals: safety, speed,and concurrency. It maintains these goals without having a garbage collector,making it a useful language for a number of use cases other languages aren’t good at: embedding in other languages, programs with specific space and time requirements, and writing low-level code, like device drivers and operating systems. It improves on current languages targeting this space by having a number of compile-time safety checks that produce no runtime overhead, while eliminating all data races. Rust also aims to achieve ‘zero-cost abstractions’ even though some of these abstractions feel like those of a high-level language. Even then, Rust still allows precise control like a low-level language would.

欢迎你了解Rust！这本书将教你关于[Rust编程语言][rust]的知识。Rust是一个系统编程语言，它的诞生是为了做到三个目标：安全、快速、和并发。它没有使用垃圾回收器，来实现这些目标；这使它变得非常有用，尤其是在其他语言并不擅长的用例上：在特殊空间和时间要求嵌入到其他语言中，和编写底层代码，比如设陪驱动程序和操作系统。在清除了所有数据races的同时，通过没有任何运行时开销的多个编译时的安全检查操作改善当前语言的性能，Rust同样致力于实现“零成本抽象概念”，尽管有些抽象概念像是高级语言。然而及时这样，Rust仍然向一个低级语言一样允许精确控制。

[rust]: http://rust-lang.org

“The Rust Programming Language” is split into seven sections. This introduction is the first. After this:

“Rust编程语言”分为7大部分，这个说明是第一部分，然后是：

* [Getting started 开始使用][gs] - Set up your computer for Rust development.配置你的Rust开发环境。
* [Learn Rust 学习Rust][lr] - Learn Rust programming through small projects.通过几个小例子来学习Rust编程。
* [Effective Rust 高效的Rust][er] - Higher-level concepts for writing excellent Rust code.编写优质Rust代码的高级理论。
* [Syntax and Semantics 语法与语义][ss] - Each bit of Rust, broken down into small chunks.Rust的每一部分，分解成一个个小块。
* [Nightly Rust 每日Rust构建][nr] - Cutting-edge features that aren’t in stable builds yet. 尚未稳定构建的顶端特性。
* [Glossary 术语表][gl] - A reference of terms used in the book. 本书中一些术语的引用
* [Academic Research 学术研究][ar] - Literature that influenced Rust.影响了Rust的文献。

[gs]: getting-started.html
[lr]: learn-rust.html
[er]: effective-rust.html
[ss]: syntax-and-semantics.html
[nr]: nightly-rust.html
[gl]: glossary.html
[ar]: academic-research.html

After reading this introduction, you’ll want to dive into either ‘Learn Rust’
or ‘Syntax and Semantics’, depending on your preference: ‘Learn Rust’ if you
want to dive in with a project, or ‘Syntax and Semantics’ if you prefer to
start small, and learn a single concept thoroughly before moving onto the next.Copious cross-linking connects these parts together.

在阅读完本介绍之后，你可以根据你的喜好，随意选择`Learn Rust 学习Rust`或者`Syntax and Semantics 语法与语义`章节：如果你想要从一个项目开始，那么选择`Learn Rust 学习Rust`，或者如果你喜欢从小事做起，通过移动到下一页，一个概念一个概念的学习，将这些广泛的关联的只是连接起来。

## Contributing 社区贡献

The source files from which this book is generated can be found on Github:
[github.com/rust-lang/rust/tree/master/src/doc/trpl](https://github.com/rust-lang/rust/tree/master/src/doc/trpl)

生成本书的源代码的被放在github上：[github.com/rust-lang/rust/tree/master/src/doc/trpl](https://github.com/rust-lang/rust/tree/master/src/doc/trpl)

## A brief introduction to Rust Rust简单说明

Is Rust a language you might be interested in? Let’s examine a few small code
samples to show off a few of its strengths.

Rust是你可能感兴趣的语言吗？让我们看着一些小的代码例子，展示它的一些优势。

The main concept that makes Rust unique is called ‘ownership’. Consider this
small example:

使Rust独一无二的主要概念被称作“ownership 所有权”。参考如下例子：

```rust
fn main() {
    let mut x = vec!["Hello", "world"];
}
```

This program makes a [variable binding][var] named `x`. The value of this binding is a `Vec<T>`, a ‘vector’, that we create through a [macro][macro] defined in the standard library. This macro is called `vec`, and we invoke macros with a `!`. This follows a general principle of Rust: make things explicit. Macros can do significantly more complicated things than function calls, and so they’re visually distinct. The `!` also helps with parsing, making tooling easier to write, which is also important.

这段程序创建了一个叫做`x`的[variable binding变量绑定][var]。这个绑定的值是一个`Vec<T>`——一个向量`vector`,他是在标准库中通过一个[macro 宏][macro]来定义的。这个宏被称作`Vec`，我们使用`!`来强调宏。这遵循了Rust的一般原则：把事情明确。宏可以做比函数调用更加显著更加复杂的事情，这样使他们在视觉上区分开来。`!`同样帮助解析，是的工具更容易编写，所以也很重要。

We used `mut` to make `x` mutable: bindings are immutable by default in Rust.We’ll be mutating this vector later in the example.

在Rust语言中，默认变量是不可变的，我们通过使用`mut`使得变量`x`能够被改变。在稍后的例子中，我们将改变这个向量 。

It’s also worth noting that we didn’t need a type annotation here: while Rust is statically typed, we didn’t need to explicitly annotate the type. Rust has type inference to balance out the power of static typing with the verbosity of annotating types.

值得一提的是，我们不需要在这里声明一个类型，因为Rust是静态类型的，我们不需要明确声明一个类型。Rust拥有类型推断，能够平衡类型标记型静态类型的能量。

Rust prefers stack allocation to heap allocation: `x` is placed directly on the stack. However, the `Vec<T>` type allocates space for the elements of the vector on the heap. If you’re not familiar with this distinction, you can ignore it for now, or check out [‘The Stack and the Heap’][heap]. As a systems programming language, Rust gives you the ability to control how your memory is allocated, but when we’re getting started, it’s less of a big deal.

比起堆内存管理，Rust语言更喜欢栈内存管理：`x`变量直接被分配在栈上，然而`Vec<T>`类型因为向量的元素组成被分配在堆上。如果你不熟悉这个地方，现在可以忽略它，或者点击链接[‘The Stack and the Heap 栈和堆’][heap]了解它。作为一门系统编程语言，Rust给我们控制内存分配的能力，但是，当我们开始使用Rust时，它不再是一个大麻烦。

[var]: variable-bindings.html
[macro]: macros.html
[heap]: the-stack-and-the-heap.html

Earlier, we mentioned that ‘ownership’ is the key new concept in Rust. In Rust parlance, `x` is said to ‘own’ the vector. This means that when `x` goes out of scope, the vector’s memory will be de-allocated. This is done deterministically by the Rust compiler, rather than through a mechanism such as a garbage collector. In other words, in Rust, you don’t call functions like `malloc` and `free` yourself: the compiler statically determines when you need to allocate or deallocate memory, and inserts those calls itself. To err is to be human,but compilers never forget.

前面我们说过`ownership 所有权`在Rust语言中是一个关键的新概念。在Rust语言中，`x`可以被说成是`自己`的向量。也就是说，当`x`超出作用域范围后，向量的内存空间将被释放。这是由Rust语言编译器决定的，而不是通过一个像垃圾回收器之类的机制来决定。换句话说，在Rust中，你不需要像`malloc`和`free`自己的方式来调用函数：编译器明确的决定了什么时候你需要分配或者释放内存，并且自动插入这些调用本身。人会忘记，但是编译器永远不会。

Let’s add another line to our example:

让我们在例子中增加一行：

```rust
fn main() {
    let mut x = vec!["Hello", "world"];

    let y = &x[0];
}
```

We’ve introduced another binding, `y`. In this case, `y` is a ‘reference’ to the first element of the vector. Rust’s references are similar to pointers in other languages, but with additional compile-time safety checks. References interact with the ownership system by [‘borrowing’][borrowing] what they point to, rather than owning it. The difference is, when the reference goes out of scope, it will not deallocate the underlying memory. If it did, we’d de-allocate twice, which is bad!

我们声明了另一个变量，`y`。在案例中，`y`是向量第一个元素的一个`reference 地址引用`。Rust的引用跟其他语言的指针比较相似，但是增加了编译时安全检查。地址引用通过他们指向的[`borrowing 引用`][borrowing]来影响所有权系统，而不是通过拥有它来影响。 不同的是，当地址引用超出了作用域范围，他不会释放原本的内存。如果他做了，我们就会释放两次，这是很坏的事情！

[borrowing]: references-and-borrowing.html

Let’s add a third line. It looks innocent enough, but causes a compiler error:

现在增加第三行代码。他看起来很无辜，但是会引起编译错误：

```rust,ignore
fn main() {
    let mut x = vec!["Hello", "world"];

    let y = &x[0];

    x.push("foo");
}
```

`push` is a method on vectors that appends another element to the end of the vector. When we try to compile this program, we get an error:

`push` 是向量一个方法，向向量的末尾追加另一个元素。当我们试图编译程序时，我们得到错误信息：

```text
error: cannot borrow `x` as mutable because it is also borrowed as immutable
    x.push("foo");
    ^
note: previous borrow of `x` occurs here; the immutable borrow prevents
subsequent moves or mutable borrows of `x` until the borrow ends
    let y = &x[0];
             ^
note: previous borrow ends here
fn main() {

}
^
```

Whew! The Rust compiler gives quite detailed errors at times, and this is one of those times. As the error explains, while we made our binding mutable, we still cannot call `push`. This is because we already have a reference to an element of the vector, `y`. Mutating something while another reference exists is dangerous, because we may invalidate the reference. In this specific case, when we create the vector, we may have only allocated space for three elements.Adding a fourth would mean allocating a new chunk of memory for all those elements,copying the old values over, and updating the internal pointer to that memory.That all works just fine.The problem is that `y` wouldn’t get updated, and so we’d have a ‘dangling pointer’. That’s bad. Any use of `y` would be an error in
this case, and so the compiler has caught this for us.

哦！Rust编译器有时会给出非常详细的错误信息，这就是那些情况中的一种。错误说明，当我们创建了一个可变的变量时，我们仍然不能够调用`push`方法。这是因为我们已经有了一个向量元素的地址引用——`y`。当有元素进行地址引用时，改变一些内容是非常危险的，因为我们可能会废止地址引用。在特殊情况下，我们创建一个向量，我们只给他分配了三个元素的空间。增加第四个元素，将意味着分配一个新的内存块给所有的元素，将旧的的内存覆盖，并更新内部指针到那部分内存。它们做的刚刚好。问题是，`y`没有被更新，所以我们就会哟一个'空指针'。这是很坏的。在本案例中，`y`的任何使用都是错误的，所以编译器抓取这个信息给我们。

So how do we solve this problem? There are two approaches we can take. The first
is making a copy rather than using a reference:

```rust
fn main() {
    let mut x = vec!["Hello", "world"];

    let y = x[0].clone();

    x.push("foo");
}
```

Rust has [move semantics][move] by default, so if we want to make a copy of some data, we call the `clone()` method. In this example, `y` is no longer a reference to the vector stored in `x`, but a copy of its first element, `"Hello"`. Now that we don’t have a reference, our `push()` works just fine.

Rust默认有[move语义][move]，所以我们想要复制一些数据，我们可以调用`clone()`方法。在上述例子中，`y`不再是在`x`中保存的向量的一个地址引用，而是他第一个元素`"Hello"`的复制品。现在我们使用地址引用，所以`push()`方法运行良好。

[move]: #move-semantics

If we truly want a reference, we need the other option: ensure that our reference goes out of scope before we try to do the mutation. That looks like this:

如果我们真的想使用一个地址引用，我们需要另一个方式：在试图改变变量之前确定地址引用不在当前作用域内。就像下面的例子：

```rust
fn main() {
    let mut x = vec!["Hello", "world"];

    {
        let y = &x[0];
    }

    x.push("foo");
}
```

We created an inner scope with an additional set of curly braces. `y` will go out of scope before we call `push()`, and so we’re all good.

我们使用额外的一组花括号构建了一个内置作用域。在我们调用`push()`之前`y`已经超出了作用域范围，所以代码非常好。

This concept of ownership isn’t just good for preventing dangling pointers, but an entire set of related problems, like iterator invalidation, concurrency, and more.

所有权的概念不止是能够很好的避免空指针，还有一整套的相关难点，比如迭代器失效、并发等等。