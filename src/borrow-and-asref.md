% Borrow and AsRef   借用和引用

The [`Borrow`][borrow] and [`AsRef`][asref] traits are very similar, but
different. Here’s a quick refresher on what these two traits mean.

[`Borrow借用`][borrow]和[AsRef地址引用`][asref]特性是非常相似的，但是却又有不同。这里是对于这两个特性代表着什么意思。

[borrow]: ../std/borrow/trait.Borrow.html
[asref]: ../std/convert/trait.AsRef.html

# Borrow  借用

The `Borrow` trait is used when you’re writing a datastructure, and you want to use either an owned or borrowed type as synonymous for some purpose.

`Borrow借用`特性用  当你正在写一个数据结构，出于某种原因，你要么使用一个自有的类型要么使用一个借用类型作为代名词的时候，可以使用`Borrow借用`特性

For example, [`HashMap`][hashmap] has a [`get` method][get] which uses `Borrow`:

例如，[`HashMap`][hashmap]有一个使用了`Borrow借用`特性的[`get`方法][get]:

```rust,ignore
fn get<Q: ?Sized>(&self, k: &Q) -> Option<&V>
    where K: Borrow<Q>,
          Q: Hash + Eq
```

[hashmap]: ../std/collections/struct.HashMap.html
[get]: ../std/collections/struct.HashMap.html#method.get

This signature is pretty complicated. The `K` parameter is what we’re interested in here. It refers to a parameter of the `HashMap` itself:

这个标记是相当复杂的。`k`参数就是我们感兴趣的。它指向的是`HashMap`自身的一个参数：

```rust,ignore
struct HashMap<K, V, S = RandomState> {
```

The `K` parameter is the type of _key_ the `HashMap` uses. So, looking at
the signature of `get()` again, we can use `get()` when the key implements `Borrow<Q>`. That way, we can make a `HashMap` which uses `String` keys,but use `&str`s when we’re searching:

`K`参数就是 `HashMap`使用的 _Key_ 的类型。所以再次看一下`get()`的标记，当 _key_ 实现了`Borrow<Q>`后，我们可以使用`get()`方法。通过这种方式，我们可以做出一个`HashMap` ，它使用 的时`String字符串`键名，然而，当我们当我们查找时使用的时`&str`:

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
map.insert("Foo".to_string(), 42);

assert_eq!(map.get("Foo"), Some(&42));
```

This is because the standard library has `impl Borrow<str> for String`.

这是因为标准库拥有一个 `impl Borrow<str> for String`。

For most types, when you want to take an owned or borrowed type, a `&T` is enough. But one area where `Borrow` is effective is when there’s more than one kind of borrowed value. Slices are an area where this is especially true: you can have both an `&[T]` or a `&mut [T]`. If we wanted to accept both of these types, `Borrow` is up for it:

对于大多数类型来说，当我们想要使用一个自由类型或者借用类型时，一个`&T`就足够了。然而有一个地方使用`Borrow借用`是高效的，那就是不只是一种借用值的时候。分片是一个尤其如此地区：你可以使用`&[T]`或者`&mut [T]`这两种。如果我们想要接受这两种类型，`Borrow借用`弥补了这一点：

```
use std::borrow::Borrow;
use std::fmt::Display;

fn foo<T: Borrow<i32> + Display>(a: T) {
    println!("a is borrowed: {}", a);
}

let mut i = 5;

foo(&i);
foo(&mut i);
```

This will print out `a is borrowed: 5` twice.

这将会打印`a is borrowed: 5`两次。

# AsRef 地址引用

The `AsRef` trait is a conversion trait. It’s used for converting some value to a reference in generic code. Like this:

`AsRef`特性就是转换特性。它用于转换一些值到通用代码中的参考。就像着这样：

```rust
let s = "Hello".to_string();

fn foo<T: AsRef<str>>(s: T) {
    let slice = s.as_ref();
}
```

# Which should I use? 我应该用哪一个？

We can see how they’re kind of the same: they both deal with owned and borrowed versions of some type. However, they’re a bit different.

我们可以看到它们是多么的相似：它们都是处理一些类型的自有和借用版本。然而，它们有一点不同

Choose `Borrow` when you want to abstract over different kinds of borrowing, or when you’re building a datastructure that treats owned and borrowed values in equivalent ways, such as hashing and comparison.

当你想要抽象化不同种类的借用，后者当你正在创建一个将自有和借用值以同等方式对待的数据结构时，请选择`Borrow`，比如哈希和比较。

Choose `AsRef` when you want to convert something to a reference directly, and you’re writing generic code.

当你正在写通用代码，想要直接转换某些内容为参考时请使用`AsRef`。
