% for Loops  for循环语句

The `for` loop is used to loop a particular number of times. Rust’s `for` loops work a bit differently than in other systems languages, however. Rust’s `for` loop doesn’t look like this “C-style” `for` loop:

`for`循环被用来循环一个特定数字的次数。然而，Rust的`for`循环运行方式跟其他系统语言有一点不同。Rust的`for`遵化呢看起来并不是这样的`C风格`的`for`循环：

```c
for (x = 0; x < 10; x++) {
    printf( "%d\n", x );
}
```

Instead, it looks like this:

相反，它看起来像这样：

```rust
for x in 0..10 {
    println!("{}", x); // x: i32
}
```

In slightly more abstract terms,

抽象描述是这样的

```ignore
for var in expression {
    code
}
```

The expression is an [iterator][iterator]. The iterator gives back a series of elements. Each element is one iteration of the loop. That value is then bound to the name `var`, which is valid for the loop body. Once the body is over, the
next value is fetched from the iterator, and we loop another time. When there are no more values, the `for` loop is over.

表达式是一个[迭代器][iterator]。迭代器返回一个元素序列。每一个元素是循环的一个迭代。值被绑定到在循环体中有效的变量`var`上。一旦循环体结束，下一个值被从迭代器中取出，我们再一次循环。当没有更多值时，`for`循环结束。

[iterator]: iterators.html

In our example, `0..10` is an expression that takes a start and an end position,and gives an iterator over those values. The upper bound is exclusive, though,so our loop will print `0` through `9`, not `10`.

在我们的例子中，`0..10` 是一个表达式，有一个开始和一个结束位置，并且给处包含他们的迭代器。由于上线是排他性的，所以我们的循环将打印从`0`至`9`,不包含`10`.

Rust does not have the “C-style” `for` loop on purpose. Manually controlling each element of the loop is complicated and error prone, even for experienced C developers.

Rust不是用“C风格”的`for`循环是有意图的。手动控制循环的每一个元素是复杂的，并且易出错，即便是具有丰富经验的C开发者。