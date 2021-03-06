# 'The Rust Programming Language' as EBook Rust编程语言电子书
  s

由 @hainuo 翻译成中文.由于第一次翻译科技文献，不太熟悉英语的表达方式，所以可能有不少地方不到位请大家见谅；同时请大家指出其中翻译错误地方，以便 @hainuo 进行修正。  
Ps：由于是根据个人兴趣进行翻译，可能你所需要的章节没有翻译，如果你想加入到翻译中来，请fork 并提交pull request

##翻译进度（已经完成章节列表）
+ 介绍说明 [readme](src/readme.nmd)
+ 第四章  [高效的Rust](src/effective-rust.md)
  - 第一节  [堆和栈](src/the-stack-and-the-heap.md)
  - 第二节  [测试用例](src/testing.md)
  - 第三节  [条件编译](src/conditional-compilation.md)
  - 第四节  [文档](src/documentation.md)
  - 第五节  [迭代器](src/iterators.md)
  - 第六节  [并发](src/concurrency.md)
  - 第七节  [错误处理](src/error-handling.md)
  - 第八节  [外部函数接口](src/ffi.md)
  - 第九节  [借用和引用](src/borrow-and-asref.md)
  - 第十节  [发行通道](src/release-channels.md)
+ 第五章 [语法与语义](src/syntax-and-semantics.md)
  - 第一节  [变量绑定](src/variable-bindings.md)
  - 第二节  [函数](src/functions.md)
  - 第三节  [注释](src/comments.md)
  - 第四节  [if条件语句](src/if.md)
  - 第五节  [for循环](src/for-loops.md)
  - 第六节  [while循环语句](src/while-loops.md)
  - 第七节  [所有权](src/ownership.md)
  - 第八节  [地址引用和借用](references-and-borrowing.md)
  
This repository contains stuff to convert [this book](http://doc.rust-lang.org/book/) to HTML, EPUB and PDF.

本仓库主要是将doc.rust-lang.org/book 转换成html，epub和pdf。你可在本项目的dist目录中获取到最新编译好的中英文对照版本。

[点击此处快速下载pdf格式的中英文对照最新版](dist/trpl-2015-05-15-a4.pdf)

英文版下载链接可以点击此处
**[Download Links](http://killercup.github.io/trpl-ebook/)**

## DIY  操作指南

Install: 安装：

- pandoc
- ruby
- XeLaTeX and probably some additional packages, I had to install (`sudo tlmgr install $pkg`) those:
    + framed
    + hyphenat
    + quotchap
- the DejaVu Sans Mono font: http://dejavu-fonts.org/
- the IPA font for Japanese Text: http://ipafont.ipa.go.jp/ipaexfont/download.html#en

Then run: 运行命令

```sh
$ ruby build.rb
```

Voilá!搞定！

ps:  (hainuo)
关于mactex大家可以通过这个地方来获取
[macText](http://tug.org/mactex/mactex-download.html)，前面的这个网址下载比较快速最新版本是20150613 大约2.5g

## License  许可

The book content itself as well as any code I added as part of this repository is Copyright (c) 2015 The Rust Project Developers and licensed like Rust itself ([MIT](https://github.com/rust-lang/rust/blob/master/LICENSE-MIT) and [Apache](https://github.com/rust-lang/rust/blob/master/LICENSE-APACHE)).
本书内容及我所加入的本仓库中的任何代码都是归于rust 项目开发者所有，并且继承于Rust语言的许可MIT和Apache。
