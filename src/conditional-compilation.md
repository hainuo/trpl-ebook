% Conditional Compilation 条件编译

Rust has a special attribute, `#[cfg]`, which allows you to compile codebased on a flag passed to the compiler. It has two forms:

Rust语言有一个特殊的属性——`#[cfg]`——它允许你基于一个标记参数来编译代码。它有两种形式：

```rust
#[cfg(foo)]
# fn foo() {}

#[cfg(bar = "baz")]
# fn bar() {}
```

They also have some helpers:

他们有一些辅助项

```rust
#[cfg(any(unix, windows))]
# fn foo() {}

#[cfg(all(unix, target_pointer_width = "32"))]
# fn bar() {}

#[cfg(not(foo))]
# fn not_foo() {}
```

These can nest arbitrarily:

他们可以任意嵌套：

```rust
#[cfg(any(not(unix), all(target_os="macos", target_arch = "powerpc")))]
# fn foo() {}
```

As for how to enable or disable these switches, if you’re using Cargo,they get set in the [`[features]` section][features] of your `Cargo.toml`:

那么当你使用Cargo的时候，如何开启和禁用这些开关呢？他们被设定在你的`Cargo.toml`文件的[`[features]` section][features] 配置块中。 


[features]: http://doc.crates.io/manifest.html#the-[features]-section

```toml
[features]
# no features by default
default = []

# The “secure-password” feature depends on the bcrypt package.
secure-password = ["bcrypt"]
```

When you do this, Cargo passes along a flag to `rustc`:

当你做这么做时，Cargo传递给`rustc`一个标记：

```text
--cfg feature="${feature_name}"
```

The sum of these `cfg` flags will determine which ones get activated, and therefore, which code gets compiled. Let’s take this code:

这些`cfg`标记汇总起来将决定哪些代码能够被激活，因此那部分代码得以被编译。让我们看下面的代码：

```rust
#[cfg(feature = "foo")]
mod foo {
}
```

If we compile it with `cargo build --features "foo"`, it will send the `--cfg
feature="foo"` flag to `rustc`, and the output will have the `mod foo` in it.If we compile it with a regular `cargo build`, no extra flags get passed on,and so, no `foo` module will exist.

如果我们使用`cargo build --features "foo"`来编译它，它将会给`tustc`添加一个`--cfg
feature="foo"`的标记，并且在输出信息中有`mod foo`。如果我们使用普通的`cargo build`来编译它，那么就没有给`rustc`添加任何标记，因此也就没有`foo`存在。

# cfg_attr

You can also set another attribute based on a `cfg` variable with `cfg_attr`:

你同样可以使用`cfg_attr`给`cfg`变量设置额外的属性值。

```rust
#[cfg_attr(a, b)]
# fn foo() {}
```

Will be the same as `#[b]` if `a` is set by `cfg` attribute, and nothing otherwise.

如果`a`被`cfg`属性设置，`#[b]`也将同样被设计，否则反之。

# cfg!

The `cfg!` [syntax extension][compilerplugins] lets you use these kinds of flags elsewhere in your code, too:

`cfg!`[语法扩展][compilerplugins]也能让你在代码中其他地方使用这种标记：

```rust
if cfg!(target_os = "macos") || cfg!(target_os = "ios") {
    println!("Think Different!");
}
```

[compilerplugins]: compiler-plugins.html

These will be replaced by a `true` or `false` at compile-time, depending on the configuration settings.

根据不同的配置设置，在编译时，这些标记被`true`或者`false`替代。