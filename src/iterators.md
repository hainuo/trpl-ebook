% Iterators 迭代器

Let's talk about loops.

下面我们来说说循环。

Remember Rust's `for` loop? Here's an example:

还记得Rust语言的`for`循环吗？这里有个例子：

```rust
for x in 0..10 {
    println!("{}", x);
}
```

Now that you know more Rust, we can talk in detail about how this works.Ranges (the `0..10`) are 'iterators'. An iterator is something that we can call the `.next()` method on repeatedly, and it gives us a sequence of things.

现在你了解到更多的Rust知识，我们可以讨论一下循环是怎么运行的了。`0..10`就是`iterators 迭代器`。迭代器就是我们能够不断重复代用的`.next()`，并且它给了我们一个事物的序列。

Like this:

就像这样：

```rust
let mut range = 0..10;

loop {
    match range.next() {
        Some(x) => {
            println!("{}", x);
        },
        None => { break }
    }
}
```

We make a mutable binding to the range, which is our iterator. We then `loop`,with an inner `match`. This `match` is used on the result of `range.next()`,which gives us a reference to the next value of the iterator. `next` returns an `Option<i32>`, in this case, which will be `Some(i32)` when we have a value and `None` once we run out. If we get `Some(i32)`, we print it out, and if we get `None`, we `break` out of the loop.

我们设置了一个范围的变量，它就是我们的迭代器。使用一个内置的`match`方法进行循环.`match`被用在`range.next()`的值上，他给我们迭代器下一个值的地址引用。`netx`返回一个`Option<i32>`（译者：就是整型值），在本案例中，当我们有一个值时，他就是`Some(i32)`,或者运行一次`None`。如果我们的匹配值是`Some(u=i32)`，我们就打印出来，如果我们匹配的值是`None`，我们`break打断`这个循环，跳出来。

This code sample is basically the same as our `for` loop version. The `for` loop is just a handy way to write this `loop`/`match`/`break` construct.

这段代码基本与我们的`for`循环相同。该`for`循环只是用了一个方便的方式来写这个`loop 循环`/`match匹配`/`break跳出`结构。

`for` loops aren't the only thing that uses iterators, however. Writing your own iterator involves implementing the `Iterator` trait. While doing that is outside of the scope of this guide, Rust provides a number of useful iterators to accomplish various tasks. Before we talk about those, we should talk about a Rust anti-pattern. And that's using ranges like this.

然而，`for`循环并不是使用迭代器的唯一方式。编写你自己的的迭代器，涉及到了`Iterator`特性。然而这样做超出了本教程的范围，Rust 提供了许多有用的迭代器来完成歌中任务。在我们谈论这些之前，我们来讨论下Rust语言的反射模式。它们行这样使用序列。

Yes, we just talked about how ranges are cool. But ranges are also very
primitive. For example, if you needed to iterate over the contents of a vector,you may be tempted to write this:

是的，我们只谈论下为什么序列是非常酷的。但是，他同样是很原始的。举个例子，如果遍历一个响亮的内容，你可能会这样写：

```rust
let nums = vec![1, 2, 3];

for i in 0..nums.len() {
    println!("{}", nums[i]);
}
```

This is strictly worse than using an actual iterator. You can iterate over vectors directly, so write this:

这比使用实际的迭代器都要糟糕。你可以直接遍历向量，所以应该这么写：

```rust
let nums = vec![1, 2, 3];

for num in &nums {
    println!("{}", num);
}
```

There are two reasons for this. First, this more directly expresses what we mean. We iterate through the entire vector, rather than iterating through indexes, and then indexing the vector. Second, this version is more efficient:the first version will have extra bounds checking because it used indexing,`nums[i]`. But since we yield a reference to each element of the vector in turn with the iterator, there's no bounds checking in the second example. This is very common with iterators: we can ignore unnecessary bounds checks, but still know that we're safe.

这样做有两个原因。首先，这更能直接表达处我们的意思。我们遍历整个向量，并且索引向量，而不是遍历索引。第二点原因就是这样做更加高效：第一版的代码需要额外的边界检查，因为他用的是索引-`nums[i]`。但是由于我们使用迭代器时得到了向量循环的每一个元素的索引，所以在第二版的代码中不需要边界检查。这是迭代器的通用做法：我们忽略不必要的边界检查，却仍然保持边界安全。

There's another detail here that's not 100% clear because of how `println!` works. `num` is actually of type `&i32`. That is, it's a reference to an `i32`,not an `i32` itself. `println!` handles the dereferencing for us, so we don't see it. This code works fine too:

这里有另一个细节，因为`println!`的执行方式，不是100%的清楚。`num`是真实的`&i32`类型。这是说，他是一个`i32`的地址引用，而不是`i32`自身。`println!`自动为我们处理了这个引用，所以，我们看不到它。也因为这样，代码能够很好的运行：

```rust
let nums = vec![1, 2, 3];

for num in &nums {
    println!("{}", *num);
}
```

Now we're explicitly dereferencing `num`. Why does `&nums` give us references?Firstly, because we explicitly asked it to with `&`. Secondly, if it gave us the data itself, we would have to be its owner, which would involve making a copy of the data and giving us the copy. With references, we're just borrowing a reference to the data, and so it's just passing a reference, without needing to do the move.

现在我们明确解除`num`的地址引用。为什么`&nums`给我们的是一个地址引用呢？首先，因为我们使用`&`来明确的请求它。第二如果他给我们他们身的数据，我们不得不程伟他的拥有者，这将涉及到制作数据副本，并给我们这个副本。使用地址引用，我们只需要引用数据的一个地址，并且他只是传递一个地址，不需要做其他事情。

So, now that we've established that ranges are often not what you want, let's talk about what you do want instead.

所以，现在我们已经确定了序列常常不是我们所需要的，让我们讨论下，你想要的是什么吧。

There are three broad classes of things that are relevant here: iterators,*iterator adapters*, and *consumers*. Here's some definitions:
 
一共有三大种类：迭代器，*迭代器适配器*和*消费者*。这是他们的定义：

* *iterators* give you a sequence of values. *迭代器*  给你有一些值的序列。
* *iterator adapters* operate on an iterator, producing a new iterator with a different output sequence. *迭代器适配器* 操作一个迭代器，生成一个新的不同序列的迭代器。
* *consumers* operate on an iterator, producing some final set of values.*消费者* 操作一个迭代器，生成最后的一组值。

Let's talk about consumers first, since you've already seen an iterator, ranges.

让我们首先讨论消费者，因为你已经见过一个迭代器——序列。

## Consumers  消费者

A *consumer* operates on an iterator, returning some kind of value or values.The most common consumer is `collect()`. This code doesn't quite compile,but it shows the intention:

一个*consumer*操作一个迭代器时，返回某种值或者所有值。最通常使用的消费者是`collect()`.这段代码无法被编译，只是为了表明某种意图。

```{rust,ignore}
let one_to_one_hundred = (1..101).collect();
```

As you can see, we call `collect()` on our iterator. `collect()` takes as many values as the iterator will give it, and returns a collection of the results. So why won't this compile? Rust can't determine what type of things you want to collect, and so you need to let it know.Here's the version that does compile:

曾如你所见，我们在迭代器上调用了`collect()`。`collect()`收集尽可能躲得迭代器的值，并返回结果集。那么为什么这个代码不能够被编译呢？Rust语言无法决定你想要收集何种类型的内容，所以你需要让它知道。这是那段代码的编译版本：

```rust
let one_to_one_hundred = (1..101).collect::<Vec<i32>>();
```

If you remember, the `::<>` syntax allows us to give a type hint,and so we tell it that we want a vector of integers. You don't always need to use the whole type, though. Using a `_` will let you provide a partial hint:

如果你还记得，`::<>`语法允许我们给出一个类型提示，那么我们可以告诉它，我们需要一个整型的向量。然而你并不需要使用所有的类型。你可以通过使用`_`来给出部分提示：

```rust
let one_to_one_hundred = (1..101).collect::<Vec<_>>();
```

This says "Collect into a `Vec<T>`, please, but infer what the `T` is for me."`_` is sometimes called a "type placeholder" for this reason.

这是说“收集成一个`向量Vec<T>`,拜托，不要为我推断`T`。”因为这个原因，`_`有时候被称作占位符。

`collect()` is the most common consumer, but there are others too. `find()` is one:

`collect()`是最常用的消费者，还有一些其他方法也是。`find()`也是其中一个：

```rust
let greater_than_forty_two = (0..100)
                             .find(|x| *x > 42);

match greater_than_forty_two {
    Some(_) => println!("We got some numbers!"),
    None => println!("No numbers found :("),
}
```

`find` takes a closure, and works on a reference to each element of an iterator. This closure returns `true` if the element is the element we're looking for, and `false` otherwise. Because we might not find a matching element, `find` returns an `Option` rather than the element itself.

`find`使用一个闭包，运行是建立在迭代器每一个元素的地址引用基础上的。当元素时我们寻找的时，这段代码中的闭包会返回`true`,反之，返回`false`。因为我们可能不会找到匹配的元素，`find`会返回一个`Option` 而不是元素本身。

Another important consumer is `fold`. Here's what it looks like:

另一个重要的消费者是`fold`。这是它看起来的样子：

```rust
let sum = (1..4).fold(0, |sum, x| sum + x);
```

`fold()` is a consumer that looks like this:`fold(base, |accumulator, element| ...)`. It takes two arguments: the first is an element called the *base*. The second is a closure that itself takes two arguments: the first is called the *accumulator*, and the second is an *element*. Upon each iteration, the closure is called, and the result is the value of the accumulator on the next iteration. On the first iteration, the base is the value of the accumulator.

`fold()`是一个想这样子的消费者：`fold(base, |accumulator,element| ...)`。它有两个参数：一个是叫做*base*的元素，另一个是一个拥有两个参数的闭包（其中一个参数被叫做*accumulator*，另一个是*element*）。在每一次迭代中，闭包都被调用，结果是对下一个迭代累加器的值。在第一个迭代中，`base`是累加器的值。

Okay, that's a bit confusing. Let's examine the values of all of these things in this iterator:

好吧，这里有点混乱。让我们来看看在迭代器中，所有这些内容的值：

| base | accumulator | element | closure result |
|------|-------------|---------|----------------|
| 0    | 0           | 1       | 1              |
| 0    | 1           | 2       | 3              |
| 0    | 3           | 3       | 6              |

We called `fold()` with these arguments:

我们调用`fold()`使用这些参数：

```rust
# (1..4)
.fold(0, |sum, x| sum + x);
```

So, `0` is our base, `sum` is our accumulator, and `x` is our element.  On the first iteration, we set `sum` to `0`, and `x` is the first element of `nums`,`1`. We then add `sum` and `x`, which gives us `0 + 1 = 1`. On the second iteration, that value becomes our accumulator, `sum`, and the element is the second element of the array, `2`. `1 + 2 = 3`, and so that becomes the value of the accumulator for the last iteration. On that iteration,`x` is the last element, `3`, and `3 + 3 = 6`, which is our final result for our sum. `1 + 2 + 3 = 6`, and that's the result we got.

所以，`0`就是base，`num`是accumulator,`x`是element。在第一次迭代时，我们设置`sum`的值为0，`x`是`nums`的第一个元素——`1`.然后我们将`sum`和`x`相加，得到`0+1=1`。在第二次迭代中，这个值变成了accumulator——`sum`的值，element是数组的第二个元素——`2`.`1+2=3`，他有成为最后一个迭代时accumulator的值。在最后一次迭代时，`x`是最后一个元素——`3`，`3+3=6`,这也是最后的`sum`的值。`1+2+3=6`,这就是我们得到最终结果。

Whew. `fold` can be a bit strange the first few times you see it, but once it clicks, you can use it all over the place. Any time you have a list of things,and you want a single result, `fold` is appropriate.

`fold`在你最初看他的几回时可能是有点奇怪的，但是一旦它被激活，你可以用在所有的地方。任何时候，你有一个清单，需要返回单一结果时，`fold`是最合适的。

Consumers are important due to one additional property of iterators we haven't talked about yet: laziness. Let's talk some more about iterators, and you'll see why consumers matter.

消费者的重要性取决于我们还没有讨论的，迭代器的一个附加属性：laziness。让我们多讨论些迭代器，你讲明白消费者关系。

## Iterators  迭代器

As we've said before, an iterator is something that we can call the `.next()` method on repeatedly, and it gives us a sequence of things.Because you need to call the method, this means that iterators can be *lazy* and not generate all of the values upfront. This code,
for example, does not actually generate the numbers `1-100`, instead creating a value that merely represents the sequence:

正如我们之前所说，迭代器就是一个能够重复调用`.next()`方法的事物，它给我们一个事物的序列。因为我们需要调用这个方法，这意味着迭代器是*lazy*，而不是生成前期所有的值。例如，这段代码并不是直接生成`1-100`,而是，创建一个仅仅表示一个序列的值，

```rust
let nums = 1..100;
```

Since we didn't do anything with the range, it didn't generate the sequence.Let's add the consumer:

因为我们使用这个范围什么都不能做，他不能够生成序列。让我们加入一个消费者：

```rust
let nums = (1..100).collect::<Vec<i32>>();
```

Now, `collect()` will require that the range gives it some numbers, and so it will do the work of generating the sequence.

现在`collect()`将请求范围给它的一些数，所以，它能够做生成序列这个事情。

Ranges are one of two basic iterators that you'll see. The other is `iter()`.`iter()` can turn a vector into a simple iterator that gives you each element in turn:

Ranges是你即将看到的两个基本迭代器之一。另一个是`iter()`。`iter`能够将向量转换成一个简单的迭代器，依次给你每一个元素：

```rust
let nums = vec![1, 2, 3];

for num in nums.iter() {
   println!("{}", num);
}
```

These two basic iterators should serve you well. There are some more advanced iterators, including ones that are infinite.

这两个基本的迭代器将很好的服务你。还有一些高级迭代器，包括哪些无穷大的。

That's enough about iterators. Iterator adapters are the last concept we need to talk about with regards to iterators. Let's get to it!

迭代器的东西已经足够多了。迭代器适配器是我们需要讨论的最后一个概念，让我们开始吧！

## Iterator adapters 迭代器适配器

*Iterator adapters* take an iterator and modify it somehow, producing a new iterator. The simplest one is called `map`:

*Iterator adapters* 获取一个迭代器，并在某些时候修改它，生成一个新的迭代器。最基本的一个叫做`map`:

```{rust,ignore}
(1..100).map(|x| x + 1);
```

`map` is called upon another iterator, and produces a new iterator where each element reference has the closure it's been given as an argument called on it.So this would give us the numbers from `2-100`. Well, almost! If you compile the example, you'll get a warning:

`map`被另一个迭代器调用，并生成一个新的迭代器，在每一个元素引用 拥有一个闭包座位参数被他调用的地方。所以，这将给我们从`2-100`的数字。好了，差不多了！如果你编译这个例子，你会收到一个敬告：


```text
warning: unused result which must be used: iterator adaptors are lazy and
         do nothing unless consumed, #[warn(unused_must_use)] on by default
(1..100).map(|x| x + 1);
 ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

Laziness strikes again! That closure will never execute. This example doesn't print any numbers:

laziness再次罢工了！闭包讲不会被执行。这个例子没有输出任何数字：


```{rust,ignore}
(1..100).map(|x| println!("{}", x));
```

If you are trying to execute a closure on an iterator for its side effects,just use `for` instead.

如果你想自一个迭代器的副本上执行一个闭包，请使用`for`。

There are tons of interesting iterator adapters. `take(n)` will return an iterator over the next `n` elements of the original iterator. Note that this has no side effect on the original iterator. Let's try it out with our infinite iterator from before:

有成千上万的迭代器适配器。`take(n)`将反悔一个覆盖了原来第n个元素迭代器的迭代器。注意，这对原来的迭代器无副作用。让我们尝试一下之前说过的无限迭代器：

```rust
# #![feature(step_by)]
for i in (1..).step_by(5).take(5) {
    println!("{}", i);
}
```

This will print

这将会打印

```text
1
6
11
16
21
```

`filter()` is an adapter that takes a closure as an argument. This closure returns `true` or `false`. The new iterator `filter()` produces only the elements that that closure returns `true` for:

`filter()` 是一个适配器，它使用一个闭包座位参数。这个闭包反悔`true`或者`false`。新的迭代器`filter()`只生成一个在闭包返回`true`时的元素：

```rust
for i in (1..100).filter(|&x| x % 2 == 0) {
    println!("{}", i);
}
```

This will print all of the even numbers between one and a hundred.(Note that because `filter` doesn't consume the elements that are being iterated over, it is passed a reference to each element, and thus the filter predicate uses the `&x` pattern to extract the integer
itself.)

这将打印0到100的所有的每一个偶数。（注意，`filter`并不消耗正在迭代的元素，他只是传递一个美格元素的地址引用，并且filter 使用`&x`方式来提取整数本身。）

You can chain all three things together: start with an iterator, adapt it a few times, and then consume the result. Check it out:

你可以将三个事物关联在一起了，首先是迭代器，适应它一段时间，然后是产生一个结果。看看下面的代码

```rust
(1..1000)
    .filter(|&x| x % 2 == 0)
    .filter(|&x| x % 3 == 0)
    .take(5)
    .collect::<Vec<i32>>();
```

This will give you a vector containing `6`, `12`, `18`, `24`, and `30`.

这将给我们一个包含有`6`，`12`，`18`，`24`和`30`的向量。

This is just a small taste of what iterators, iterator adapters, and consumers can help you with. There are a number of really useful iterators, and you can write your own as well. Iterators provide a safe, efficient way to manipulate all kinds of lists. They're a little unusual at first, but if you play with them, you'll get hooked. For a full list of the different iterators and consumers, check out the [iterator module documentation](../std/iter/index.html).

这只是一个关于迭代器，迭代器适配器和消费者能够帮你什么忙的小尝试。有太多有用的迭代器，同样，你可以写你自己的迭代器。迭代器提供一个安全、高效的方式来操纵各种各样的清单列表。起先，它们是有点不寻常，当你多使用他们，你就会上瘾。对于一个完整的迭代器和消费者之间不同之处，可以点击链接[iterator module documentation 迭代器模块文档](../std/iter/index.html)查看。
