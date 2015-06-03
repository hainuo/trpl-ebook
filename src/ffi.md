% Foreign Function Interface  外部函数接口

# Introduction  说明

This guide will use the [snappy](https://github.com/google/snappy)
compression/decompression library as an introduction to writing bindings for foreign code. Rust is currently unable to call directly into a C++ library, but snappy includes a C interface (documented in
[`snappy-c.h`](https://github.com/google/snappy/blob/master/snappy-c.h)).

本书将使用[snappy](https://github.com/google/snappy)压缩发压缩库作为例子来编写外部代码的变量绑定。目前Rust语言不能够直接使用C++库，然而snappy含有一个c接口 (记录在
[`snappy-c.h`](https://github.com/google/snappy/blob/master/snappy-c.h)).

The following is a minimal example of calling a foreign function which will compile if snappy is installed:

下面就是一个简单的调用外部函数的例子，如果snappy已经被安装了，那么他将会被编译。

```no_run
# #![feature(libc)]
extern crate libc;
use libc::size_t;

#[link(name = "snappy")]
extern {
    fn snappy_max_compressed_length(source_length: size_t) -> size_t;
}

fn main() {
    let x = unsafe { snappy_max_compressed_length(100) };
    println!("max compressed length of a 100 byte buffer: {}", x);
}
```

The `extern` block is a list of function signatures in a foreign library, in this case with the platform's C ABI. The `#[link(...)]` attribute is used to instruct the linker to link against the snappy library so the symbols are resolved.

`extern`代码块是一个外部库的函数标记列表，在本案例中使用的是C平台 ABI。`#[link(...)]`属性是用来声明一个可以映射到snappy库的连接器，解决了符号链接

Foreign functions are assumed to be unsafe so calls to them need to be wrapped with `unsafe {}` as a promise to the compiler that everything contained within truly is safe. C libraries often expose interfaces that aren't thread-safe, and almost any function that takes a pointer argument isn't valid for all possible inputs since the pointer could be dangling, and raw pointers fall outside of Rust's safe memory model.

外部函数被认为是不安全的，所以需要使用`unsafe{}`来封装他们，作为向编译器的一个保证，来确保包含在内部的任何内容真的是安全的。C语言经常暴露一些非线程安全的接口，并且他们使用一个指针参数，因为指针可能为空指针，不能够验证所有可能的输入，从而导致原始指针落到Rust的安全内存模型之外的地方。

When declaring the argument types to a foreign function, the Rust compiler can not check if the declaration is correct, so specifying it correctly is part of keeping the binding correct at runtime.

当声明一个外部函数的参数类型时，Rust编译器不能够检查是否正确，所以正确地指定类型是保证在运行时保持绑定正确的一部分。

The `extern` block can be extended to cover the entire snappy API:

`extern`代码块能够被扩展来覆盖整个的snappy API：

```no_run
# #![feature(libc)]
extern crate libc;
use libc::{c_int, size_t};

#[link(name = "snappy")]
extern {
    fn snappy_compress(input: *const u8,
                       input_length: size_t,
                       compressed: *mut u8,
                       compressed_length: *mut size_t) -> c_int;
    fn snappy_uncompress(compressed: *const u8,
                         compressed_length: size_t,
                         uncompressed: *mut u8,
                         uncompressed_length: *mut size_t) -> c_int;
    fn snappy_max_compressed_length(source_length: size_t) -> size_t;
    fn snappy_uncompressed_length(compressed: *const u8,
                                  compressed_length: size_t,
                                  result: *mut size_t) -> c_int;
    fn snappy_validate_compressed_buffer(compressed: *const u8,
                                         compressed_length: size_t) -> c_int;
}
# fn main() {}
```

# Creating a safe interface 创建一个安全的接口

The raw C API needs to be wrapped to provide memory safety and make use of higher-level concepts like vectors. A library can choose to expose only the safe, high-level interface and hide the unsafe internal details.

原始的C API 需要提供安全内存和使用像向量这样的高级概念。一个库能够选择只暴露安全的、高级接口并隐藏内部不安全信息。

Wrapping the functions which expect buffers involves using the `slice::raw` module to manipulate Rust vectors as pointers to memory. Rust's vectors are guaranteed to be a contiguous block of memory. The length is number of elements currently contained, and the capacity is the total size in elements of the allocated memory. The length is less than or equal to the capacity.

封装具有缓冲区的方法包括使用`slice::raw`模块来作为内存指针来操纵Rust的向量。Rust的向量是被保护为一个连续的内存块。长度就是当前内部元素的数量，容量是所有分配内存元素大小的总和。长度是远远小于或者等于容量的

```
# #![feature(libc)]
# extern crate libc;
# use libc::{c_int, size_t};
# unsafe fn snappy_validate_compressed_buffer(_: *const u8, _: size_t) -> c_int { 0 }
# fn main() {}
pub fn validate_compressed_buffer(src: &[u8]) -> bool {
    unsafe {
        snappy_validate_compressed_buffer(src.as_ptr(), src.len() as size_t) == 0
    }
}
```

The `validate_compressed_buffer` wrapper above makes use of an `unsafe` block, but it makes the guarantee that calling it is safe for all inputs by leaving off `unsafe` from the function signature.

上面的`validate_compressed_buffer`封装使用了一个`unsafe`模块，然而，通过函数标记在离开`unsafe`封装时，它保证对于所有的输入操作，调用他是安全的。

The `snappy_compress` and `snappy_uncompress` functions are more complex, since a buffer has to be allocated to hold the output too.

`snappy_compress` 和 `snappy_uncompress`更加负责，因为一个缓冲区已经被分配用来保持输出。

The `snappy_max_compressed_length` function can be used to allocate a vector with the maximum required capacity to hold the compressed output. The vector can then be passed to the `snappy_compress` function as an output parameter. An output parameter is also passed to retrieve the true length after compression for setting the length.

 `snappy_max_compressed_length`函数用来分配一个使用所需要的最大容量的向量，来保证压缩输出。这个向量随后可以被传递给`snappy_compress`作为一个输出参数。输出参数同样被传递用来检查为设定长度的压缩后的真实长度。

```
# #![feature(libc)]
# extern crate libc;
# use libc::{size_t, c_int};
# unsafe fn snappy_compress(a: *const u8, b: size_t, c: *mut u8,
#                           d: *mut size_t) -> c_int { 0 }
# unsafe fn snappy_max_compressed_length(a: size_t) -> size_t { a }
# fn main() {}
pub fn compress(src: &[u8]) -> Vec<u8> {
    unsafe {
        let srclen = src.len() as size_t;
        let psrc = src.as_ptr();

        let mut dstlen = snappy_max_compressed_length(srclen);
        let mut dst = Vec::with_capacity(dstlen as usize);
        let pdst = dst.as_mut_ptr();

        snappy_compress(psrc, srclen, pdst, &mut dstlen);
        dst.set_len(dstlen as usize);
        dst
    }
}
```

Decompression is similar, because snappy stores the uncompressed size as part of the compression format and `snappy_uncompressed_length` will retrieve the exact buffer size required.

解压缩是类似的，因为snappy将未压缩大小作为压缩格式的一部分来存储，`snappy_uncompressed_length`会校验所需要的精确地缓冲区大小。

```
# #![feature(libc)]
# extern crate libc;
# use libc::{size_t, c_int};
# unsafe fn snappy_uncompress(compressed: *const u8,
#                             compressed_length: size_t,
#                             uncompressed: *mut u8,
#                             uncompressed_length: *mut size_t) -> c_int { 0 }
# unsafe fn snappy_uncompressed_length(compressed: *const u8,
#                                      compressed_length: size_t,
#                                      result: *mut size_t) -> c_int { 0 }
# fn main() {}
pub fn uncompress(src: &[u8]) -> Option<Vec<u8>> {
    unsafe {
        let srclen = src.len() as size_t;
        let psrc = src.as_ptr();

        let mut dstlen: size_t = 0;
        snappy_uncompressed_length(psrc, srclen, &mut dstlen);

        let mut dst = Vec::with_capacity(dstlen as usize);
        let pdst = dst.as_mut_ptr();

        if snappy_uncompress(psrc, srclen, pdst, &mut dstlen) == 0 {
            dst.set_len(dstlen as usize);
            Some(dst)
        } else {
            None // SNAPPY_INVALID_INPUT
        }
    }
}
```

For reference, the examples used here are also available as a [library on
GitHub](https://github.com/thestinger/rust-snappy).

这里的例子同样参考适用于在[github上的库][library on
GitHub](https://github.com/thestinger/rust-snappy)。

# Destructors  析构函数

Foreign libraries often hand off ownership of resources to the calling code.When this occurs, we must use Rust's destructors to provide safety and guarantee the release of these resources (especially in the case of panic).

外部库经常将资源的所有权交给调用带按摩。当这发生时，我们需要使用Rust语言的析构函数来保证安全和保障这些资源的额释放（尤其是在panic的情况下）。

For more about destructors, see the [Drop trait](../std/ops/trait.Drop.html).

想要了解更多析构函数相关的知识，可以参见[Drop trait丢弃特性](../std/ops/trait.Drop.html)。

# Callbacks from C code to Rust functions  从C代码到Rust函数的回调方法

Some external libraries require the usage of callbacks to report back their current state or intermediate data to the caller.It is possible to pass functions defined in Rust to an external library.The requirement for this is that the callback function is marked as `extern` with the correct calling convention to make it callable from C code.

一些外部库需要使用回调方法来汇报它们的当前状态或者是调用的中间数据。在Rust中定义一个函数传递到外部库是可能的。要求是回调函数必须使用正确的调用约定标记为`extern`，来确保他能够在C代码中调用到。

The callback function can then be sent through a registration call to the C library and afterwards be invoked from there.

回调函数能够被一个注册调用发送给C库，稍后从哪里被调用。

A basic example is:

基本栗子是这样子的：

Rust code:

这是Rust代码：

```no_run
extern fn callback(a: i32) {
    println!("I'm called from C with value {0}", a);
}

#[link(name = "extlib")]
extern {
   fn register_callback(cb: extern fn(i32)) -> i32;
   fn trigger_callback();
}

fn main() {
    unsafe {
        register_callback(callback);
        trigger_callback(); // Triggers the callback
    }
}
```

C code:

这是C代码：

```c
typedef void (*rust_callback)(int32_t);
rust_callback cb;

int32_t register_callback(rust_callback callback) {
    cb = callback;
    return 1;
}

void trigger_callback() {
  cb(7); // Will call callback(7) in Rust
}
```

In this example Rust's `main()` will call `trigger_callback()` in C,which would, in turn, call back to `callback()` in Rust.

在这个例子中，Rust的`main()`将调用C中的`trigger_callback()`,反过来，他将会回调Rust中的`callback()`

## Targeting callbacks to Rust objects 定位Rust对象的回调

The former example showed how a global function can be called from C code.However it is often desired that the callback is targeted to a special Rust object. This could be the object that represents the wrapper for the respective C object.

前面一个例子说明了，一个全局的函数是如何被C代码调用的。然而，定位回调给一个Rust对象是经常被期望的。这可能是一个表示着封装了各个C对象的对象。

This can be achieved by passing an unsafe pointer to the object down to the C library. The C library can then include the pointer to the Rust object in the notification. This will allow the callback to unsafely access the referenced Rust object.

这个能够通过传递一个不安全的指向这个对象的指针到C库中。然后C库将这个Rust对象的指针包含进声明中。这将允许回调对引用的Rust对象进行不安全地存取。

Rust code:

Rust代码：

```no_run
#[repr(C)]
struct RustObject {
    a: i32,
    // other members
}

extern "C" fn callback(target: *mut RustObject, a: i32) {
    println!("I'm called from C with value {0}", a);
    unsafe {
        // Update the value in RustObject with the value received from the callback
        (*target).a = a;
    }
}

#[link(name = "extlib")]
extern {
   fn register_callback(target: *mut RustObject,
                        cb: extern fn(*mut RustObject, i32)) -> i32;
   fn trigger_callback();
}

fn main() {
    // Create the object that will be referenced in the callback
    let mut rust_object = Box::new(RustObject { a: 5 });

    unsafe {
        register_callback(&mut *rust_object, callback);
        trigger_callback();
    }
}
```

C code:

C代码：

```c
typedef void (*rust_callback)(void*, int32_t);
void* cb_target;
rust_callback cb;

int32_t register_callback(void* callback_target, rust_callback callback) {
    cb_target = callback_target;
    cb = callback;
    return 1;
}

void trigger_callback() {
  cb(cb_target, 7); // Will call callback(&rustObject, 7) in Rust
}
```

## Asynchronous callbacks  异步回调

In the previously given examples the callbacks are invoked as a direct reaction to a function call to the external C library. The control over the current thread is switched from Rust to C to Rust for the execution of the callback, but in the end the callback is executed on the same thread that called the function which triggered the callback.

前面给出的例子中回调函数是作为一个调用外部C库的函数的直接反射来调用的。通过执行回调方法，当前线程的控制权从Rust转移到C中，然后又从C中转移到Rust中，然而直到最后，回调还是被执行在引发回调的函数所调用的同一个线程上。

Things get more complicated when the external library spawns its own threads and invokes callbacks from there. In these cases access to Rust data structures inside the callbacks is especially unsafe and proper synchronization mechanisms must be used. Besides classical synchronization mechanisms like mutexes, one possibility in Rust is to use channels (in `std::comm`) to forward data from the C thread that invoked the callback into a Rust thread.

当外部库派生出自己的线程，并从哪里调用回调时，事情变得更加复杂。在这种情况下，存取回调中Rust数据结构是特别不安全的，所以必须有一个适当的同步机制。除了经典的互斥变量同步机制，在Rust语言中有一个可能性是使用信道（在`std::comm`中）来转发数据到能够调用回到到一个Rust线程的C线程中。

If an asynchronous callback targets a special object in the Rust address space it is also absolutely necessary that no more callbacks are performed by the C library after the respective Rust object gets destroyed.This can be achieved by unregistering the callback in the object's destructor and designing the library in a way that guarantees that no callback will be performed after deregistration.

如果一个异步回调针对着Rust地址空间中的一个特殊对象，各自的Rust对象被销毁之后没有C库中不会执行任何回调时绝对必要的。这可以通过在对象析构函数中反注册回调 并将库设计为一种能够确保在反注册后没有任何回调会被执行的方式来实现。

# Linking

The `link` attribute on `extern` blocks provides the basic building block for
instructing rustc how it will link to native libraries. There are two accepted
forms of the link attribute today:

* `#[link(name = "foo")]`
* `#[link(name = "foo", kind = "bar")]`

In both of these cases, `foo` is the name of the native library that we're
linking to, and in the second case `bar` is the type of native library that the
compiler is linking to. There are currently three known types of native
libraries:

* Dynamic - `#[link(name = "readline")]`
* Static - `#[link(name = "my_build_dependency", kind = "static")]`
* Frameworks - `#[link(name = "CoreFoundation", kind = "framework")]`

Note that frameworks are only available on OSX targets.

The different `kind` values are meant to differentiate how the native library
participates in linkage. From a linkage perspective, the rust compiler creates
two flavors of artifacts: partial (rlib/staticlib) and final (dylib/binary).
Native dynamic libraries and frameworks are propagated to the final artifact
boundary, while static libraries are not propagated at all.

A few examples of how this model can be used are:

* A native build dependency. Sometimes some C/C++ glue is needed when writing
  some rust code, but distribution of the C/C++ code in a library format is just
  a burden. In this case, the code will be archived into `libfoo.a` and then the
  rust crate would declare a dependency via `#[link(name = "foo", kind =
  "static")]`.

  Regardless of the flavor of output for the crate, the native static library
  will be included in the output, meaning that distribution of the native static
  library is not necessary.

* A normal dynamic dependency. Common system libraries (like `readline`) are
  available on a large number of systems, and often a static copy of these
  libraries cannot be found. When this dependency is included in a rust crate,
  partial targets (like rlibs) will not link to the library, but when the rlib
  is included in a final target (like a binary), the native library will be
  linked in.

On OSX, frameworks behave with the same semantics as a dynamic library.

# Unsafe blocks  不安全代码块

Some operations, like dereferencing unsafe pointers or calling functions that have been marked
unsafe are only allowed inside unsafe blocks. Unsafe blocks isolate unsafety and are a promise to
the compiler that the unsafety does not leak out of the block.

Unsafe functions, on the other hand, advertise it to the world. An unsafe function is written like
this:

```
unsafe fn kaboom(ptr: *const i32) -> i32 { *ptr }
```

This function can only be called from an `unsafe` block or another `unsafe` function.

# Accessing foreign globals

Foreign APIs often export a global variable which could do something like track
global state. In order to access these variables, you declare them in `extern`
blocks with the `static` keyword:

```no_run
# #![feature(libc)]
extern crate libc;

#[link(name = "readline")]
extern {
    static rl_readline_version: libc::c_int;
}

fn main() {
    println!("You have readline version {} installed.",
             rl_readline_version as i32);
}
```

Alternatively, you may need to alter global state provided by a foreign
interface. To do this, statics can be declared with `mut` so we can mutate
them.

```no_run
# #![feature(libc)]
extern crate libc;

use std::ffi::CString;
use std::ptr;

#[link(name = "readline")]
extern {
    static mut rl_prompt: *const libc::c_char;
}

fn main() {
    let prompt = CString::new("[my-awesome-shell] $").unwrap();
    unsafe {
        rl_prompt = prompt.as_ptr();

        println!("{:?}", rl_prompt);

        rl_prompt = ptr::null();
    }
}
```

Note that all interaction with a `static mut` is unsafe, both reading and
writing. Dealing with global mutable state requires a great deal of care.

# Foreign calling conventions

Most foreign code exposes a C ABI, and Rust uses the platform's C calling convention by default when
calling foreign functions. Some foreign functions, most notably the Windows API, use other calling
conventions. Rust provides a way to tell the compiler which convention to use:

```
# #![feature(libc)]
extern crate libc;

#[cfg(all(target_os = "win32", target_arch = "x86"))]
#[link(name = "kernel32")]
#[allow(non_snake_case)]
extern "stdcall" {
    fn SetEnvironmentVariableA(n: *const u8, v: *const u8) -> libc::c_int;
}
# fn main() { }
```

This applies to the entire `extern` block. The list of supported ABI constraints
are:

* `stdcall`
* `aapcs`
* `cdecl`
* `fastcall`
* `Rust`
* `rust-intrinsic`
* `system`
* `C`
* `win64`

Most of the abis in this list are self-explanatory, but the `system` abi may
seem a little odd. This constraint selects whatever the appropriate ABI is for
interoperating with the target's libraries. For example, on win32 with a x86
architecture, this means that the abi used would be `stdcall`. On x86_64,
however, windows uses the `C` calling convention, so `C` would be used. This
means that in our previous example, we could have used `extern "system" { ... }`
to define a block for all windows systems, not just x86 ones.

# Interoperability with foreign code

Rust guarantees that the layout of a `struct` is compatible with the platform's
representation in C only if the `#[repr(C)]` attribute is applied to it.
`#[repr(C, packed)]` can be used to lay out struct members without padding.
`#[repr(C)]` can also be applied to an enum.

Rust's owned boxes (`Box<T>`) use non-nullable pointers as handles which point
to the contained object. However, they should not be manually created because
they are managed by internal allocators. References can safely be assumed to be
non-nullable pointers directly to the type.  However, breaking the borrow
checking or mutability rules is not guaranteed to be safe, so prefer using raw
pointers (`*`) if that's needed because the compiler can't make as many
assumptions about them.

Vectors and strings share the same basic memory layout, and utilities are
available in the `vec` and `str` modules for working with C APIs. However,
strings are not terminated with `\0`. If you need a NUL-terminated string for
interoperability with C, you should use the `CString` type in the `std::ffi`
module.

The standard library includes type aliases and function definitions for the C
standard library in the `libc` module, and Rust links against `libc` and `libm`
by default.

# The "nullable pointer optimization"

Certain types are defined to not be `null`. This includes references (`&T`,
`&mut T`), boxes (`Box<T>`), and function pointers (`extern "abi" fn()`).
When interfacing with C, pointers that might be null are often used.
As a special case, a generic `enum` that contains exactly two variants, one of
which contains no data and the other containing a single field, is eligible
for the "nullable pointer optimization". When such an enum is instantiated
with one of the non-nullable types, it is represented as a single pointer,
and the non-data variant is represented as the null pointer. So
`Option<extern "C" fn(c_int) -> c_int>` is how one represents a nullable
function pointer using the C ABI.

# Calling Rust code from C

You may wish to compile Rust code in a way so that it can be called from C. This is
fairly easy, but requires a few things:

```
#[no_mangle]
pub extern fn hello_rust() -> *const u8 {
    "Hello, world!\0".as_ptr()
}
# fn main() {}
```

The `extern` makes this function adhere to the C calling convention, as
discussed above in "[Foreign Calling
Conventions](ffi.html#foreign-calling-conventions)". The `no_mangle`
attribute turns off Rust's name mangling, so that it is easier to link to.
