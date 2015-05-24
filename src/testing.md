% Testing 测试

> Program testing can be a very effective way to show the presence of bugs, butit is hopelessly inadequate for showing their absence.Edsger W. Dijkstra, "The Humble Programmer" (1972)
> 
> “程序测试是表明存在故障的非常有效的方法，但对于证明没有故障，调试是很无能为力的。” 艾兹格·迪科斯彻  《谦卑的程序员》 1972

Let's talk about how to test Rust code. What we will not be talking about is the right way to test Rust code. There are many schools of thought regarding the right and wrong way to write tests. All of these approaches use the same basic tools, and so we'll show you the syntax for using them.

那么，我们来讨论下如何调试Rust代码。我们不会讨论调试Rust代码的正确方法。有很多在写测试代码的正确和错误方式方面有想法的学校。他们都在使用最基本的工具，所以我们将告诉你使用它们的方法。

# The `test` attribute `test`属性

At its simplest, a test in Rust is a function that's annotated with the `test` attribute. Let's make a new project with Cargo called `adder`:

简单说来，Rust语言中的测试代码是一个函数，它使用`test`属性作为注解。现在，让我们使用`Cargo new` 新建一个名为`adder`的项目：

```bash
$ cargo new adder
$ cd adder
```

Cargo will automatically generate a simple test when you make a new project.Here's the contents of `src/lib.rs`:

当你新建一个项目的时候，`Cargo` 将自动生成一个简单的测试. 下面是`src/lib.rs`文件的内容：

```rust
#[test]
fn it_works() {
}
```

Note the `#[test]`. This attribute indicates that this is a test function. It currently has no body. That's good enough to pass! We can run the tests with `cargo test`:

大家注意下`#[test]`，这个属性表明这是一个测试函数。现在，它还没有内容。但是已经足够我们进行测试了。我们可以使用`cargo test`命令来运行这些测试用例：

```bash
$ cargo test
   Compiling adder v0.0.1 (file:///home/you/projects/adder)
     Running target/adder-91b3e234d4ed382a

running 1 test  运行一条测试
test it_works ... ok   运行测试方法it_works成功

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured 
测试结果：成功。1通过；0失败；0略过；0达标

   Doc-tests adder   文档测试

running 0 tests     运行0条测试

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
测试结果：成功。0通过；0失败；0略过；0达标
```

Cargo compiled and ran our tests. There are two sets of output here: one for the test we wrote, and another for documentation tests. We'll talk about those later. For now, see this line:

Cargo编译并运行测试。这里有两个输出结果：一个是我们写的测试的，另一个是文档测试。我们稍后谈论他们。现在，看着一行代码：

```text
test it_works ... ok
```

Note the `it_works`. This comes from the name of our function:

注意这个`it_works`,他来自于我们定义的方法名

```rust
fn it_works() {
# }
```

We also get a summary line:

我们还得到概要信息：

```text
test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
```

So why does our do-nothing test pass? Any test which doesn't `panic!` passes,
and any test that does `panic!` fails. Let's make our test fail:

为什么我们什么都没做，然后测试就通过了呢？任何没有`panic!`的测试通过，任何有`pannic!`的测试失败。让我们来做一个失败情况的测试吧：

```rust
#[test]
fn it_works() {
    assert!(false);
}
```

`assert!` is a macro provided by Rust which takes one argument: if the argument is `true`, nothing happens. If the argument is `false`, it `panic!`s.Let's run our tests again:

`assert!` 是一个Rust内置的需要一个参数的宏：如果这个参数是`true`，那么什么都不会发生。如果参数是`false`,那么他就`panic!`.我们再运行一下测试：

```bash
$ cargo test
   Compiling adder v0.0.1 (file:///home/you/projects/adder)
     Running target/adder-91b3e234d4ed382a

running 1 test
test it_works ... FAILED

failures:

---- it_works stdout ----
        thread 'it_works' panicked at 'assertion failed: false', /home/steve/tmp/adder/src/lib.rs:3



failures:
    it_works

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured

thread '<main>' panicked at 'Some tests failed', /home/steve/src/rust/src/libtest/lib.rs:247
```

Rust indicates that our test failed:

Rust命令行显示我们的测试失败了：

```text
test it_works ... FAILED
```

And that's reflected in the summary line:

并且反映在概要信息中：

```text
test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured
```

We also get a non-zero status code:

我们还可以得到了一个非零状态码：

```bash
$ echo $?
101
```

This is useful if you want to integrate `cargo test` into other tooling.

如果你想集成`cargo test`到其他工具中，这是非常有用的。

We can invert our test's failure with another attribute: `should_panic`:

我们还可以通过使用另一个属性——`should_panic`——来将测试失败情况确认为成功情况：

```rust
#[test]
#[should_panic]
fn it_works() {
    assert!(false);
}
```

This test will now succeed if we `panic!` and fail if we complete. Let's try it:

如果我们`panic!`且执行结果为失败时，这个测试将会显示成功。让我们运行它：

```bash
$ cargo test
   Compiling adder v0.0.1 (file:///home/you/projects/adder)
     Running target/adder-91b3e234d4ed382a

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

Rust provides another macro, `assert_eq!`, that compares two arguments for equality:

Rust提供了另一个宏——`assert_eq!`——它是用来比较两个参数是否相等：

```rust
#[test]
#[should_panic]
fn it_works() {
    assert_eq!("Hello", "world");
}
```

Does this test pass or fail? Because of the `should_panic` attribute, it passes:

现在测试是否成功呢？因为它拥有`should_panic`属性，所以他测试通过：

```bash
$ cargo test
   Compiling adder v0.0.1 (file:///home/you/projects/adder)
     Running target/adder-91b3e234d4ed382a

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

`should_panic` tests can be fragile, as it's hard to guarantee that the test didn't fail for an unexpected reason. To help with this, an optional `expected` parameter can be added to the `should_panic` attribute. The test harness will make sure that the failure message contains the provided text. A safer version of the example above would be:

`should_panic`测试是很不可靠的，因为它很难保证测试不会因为意外原因而运行失败。为了确保测试不会因为意外原因运行失败，一个可选的`expected`参数被加入到`should_panic`属性中。测试套件将确保失败信息中包含着提供的文本。上面例子的安全版本应当如下：

```
#[test]
#[should_panic(expected = "assertion failed")]
fn it_works() {
    assert_eq!("Hello", "world");
}
```

That's all there is to the basics! Let's write one 'real' test:

基础知识就是这些了！让我们写一个`真正`的测试吧：

```{rust,ignore}
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[test]
fn it_works() {
    assert_eq!(4, add_two(2));
}
```

This is a very common use of `assert_eq!`: call some function with some known arguments and compare it to the expected output.

这是一个`assert_eq!`的普通用法：调用一个使用可知参数的函数，并将他的运行结果与预期输出进行比较。

# The `tests` module `tests`单元

There is one way in which our existing example is not idiomatic: it's missing the `tests` module. The idiomatic way of writing our example looks like this:

在我们存在的例子中有一点是不符合通用习惯的：他没有`tests`单元。写例子的惯用方式应该像这样：

```{rust,ignore}
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::add_two;

    #[test]
    fn it_works() {
        assert_eq!(4, add_two(2));
    }
}
```

There's a few changes here. The first is the introduction of a `mod tests` with a `cfg` attribute. The module allows us to group all of our tests together, and to also define helper functions if needed, that don't become a part of the rest of our crate. The `cfg` attribute only compiles our test code if we're currently trying to run the tests. This can save compile time, and also ensures that our tests are entirely left out of a normal build.

这里有少许的不同。第一个变化是在声明一个`mod tests`之前需要一个`cfg`属性。 这样，这个模块允许我们将所有的测试放在一起，并且如果需要的话，还可以定义辅助函数——不会成为我们crate剩余的一部分。如果我们当前正在尝试运行测试，那么`cfg`属性只会编译我们的测试代码。这样就会节省编译时间，同样能够确定的是，我们的测试是完全被排除在普通构建之外的。

The second change is the `use` declaration. Because we're in an inner module,we need to bring our test function into scope. This can be annoying if you have a large module, and so this is a common use of the `glob` feature.Let's change our `src/lib.rs` to make use of it:

第二个变化是使用`use`声明。因为测试代码是在一个内部模块，我们需要将测试方法引入到当前的作用域内。如果你有一个庞大的模块，这会非常麻烦的，所以有一个通用的`glob`方法。那么让我们修改一下我们的代码`src/lib.rs`来使用这个测试方法。

```{rust,ignore}

pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(4, add_two(2));
    }
}
```

Note the different `use` line. Now we run our tests:

注意，这里有所不同的`use`语句，运行测试：

```bash
$ cargo test
    Updating registry `https://github.com/rust-lang/crates.io-index`
   Compiling adder v0.0.1 (file:///home/you/projects/adder)
     Running target/adder-91b3e234d4ed382a

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

It works! 

 它运行正常！

The current convention is to use the `tests` module to hold your "unit-style"
tests. Anything that just tests one small bit of functionality makes sense to
go here. But what about "integration-style" tests instead? For that, we have
the `tests` directory.

当前的约定是使用`test`单元作为单元式测试。，通常情况，他只能够测试一个功能的一小部分。如果是集成测试呢。为达到这一目的，我们设定了`tests`目录。

# The `tests` directory `tests`目录

To write an integration test, let's make a `tests` directory, and put a `tests/lib.rs` file inside, with this as its contents:

要编写集成测试，我们先建立一个`tests`目录，并且放入一个`tests/lib.rs`文件，文件内容如下：

```{rust,ignore}
extern crate adder;

#[test]
fn it_works() {
    assert_eq!(4, adder::add_two(2));
}
```

This looks similar to our previous tests, but slightly different. We now have an `extern crate adder` at the top. This is because the tests in the `tests` directory are an entirely separate crate, and so we need to import our library.This is also why `tests` is a suitable place to write integration-style tests:they use the library like any other consumer of it would.

看上去与之前的测试代码没啥不同，但还是有一些区别。我们现在在文件开始使用了`extern crate adder`。这是因为在`tests`目录中的测试代码是完全独立的包。所以我们需要导入我们的库文件。这样是为什么`tests`目录是一个适合些继承测试的地方：它们可以随意的使用库文件。

Let's run them:

运行测试：

```bash
$ cargo test
   Compiling adder v0.0.1 (file:///home/you/projects/adder)
     Running target/adder-91b3e234d4ed382a

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

     Running target/lib-c18e7d3494509e74

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

Now we have three sections: our previous test is also run, as well as our new
one.

现在，我们有三个部分：我们以前测试也运行，我们新的测试内容。

That's all there is to the `tests` directory. The `tests` module isn't needed
here, since the whole thing is focused on tests.

这就是`tests`目录的所有内容了。`tests`单元在这里是不需要的，因为这里所有的事情重点都在测试上。

Let's finally check out that third section: documentation tests.

最后让我们进入到第三部分：文档测试

# Documentation tests 文档测试

Nothing is better than documentation with examples. Nothing is worse than examples that don't actually work, because the code has changed since the documentation has been written. To this end, Rust supports automatically running examples in your documentation. Here's a fleshed-out `src/lib.rs` with examples:

没有什么比带例子的文档更好的。没有什么比不能如期运行的例子更差劲的了，因为文档一旦被写入，代码就被改变了。为此，Rust语言支持自动运行你在文档中的例子。这里有一个非常充实的有着使用例子的文件`src/lib.rs`:


```{rust,ignore}
//! The `adder` crate provides functions that add numbers to other numbers.
//!
//! # Examples
//!
//! ```
//! assert_eq!(4, adder::add_two(2));
//! ```

/// This function adds two to its argument.
///
/// # Examples
///
/// ```
/// use adder::add_two;
///
/// assert_eq!(4, add_two(2));
/// ```
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(4, add_two(2));
    }
}
```

Note the module-level documentation with `//!` and the function-level documentation with `///`. Rust's documentation supports Markdown in comments, and so triple graves mark code blocks. It is conventional to include the `# Examples` section, exactly like that, with examples following.

注意使用了`//!`的模块级文档和使用了`///`的函数级文档。Rust的文件支持MarkDown语法注释，所以有三种标记代码块。包含`# Examples`是常规做法，并且紧跟随着的是实例。

Let's run the tests again:

在此运行测试：

```bash
$ cargo test
   Compiling adder v0.0.1 (file:///home/steve/tmp/adder)
     Running target/adder-91b3e234d4ed382a

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

     Running target/lib-c18e7d3494509e74

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests adder

running 2 tests
test add_two_0 ... ok
test _0 ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured
```

Now we have all three kinds of tests running! Note the names of the documentation tests: the `_0` is generated for the module test, and `add_two_0` for the function test. These will auto increment with names like `add_two_1` as you add more examples.

现在我们掌握了全部的三种测试方式！注意文档测试的名称：`_0`是由模块测试生成的，`add_two_0`是有函数测试生成的。随着你增加更多的例子，`_*`将像`add_two_1`这样自动增加。
