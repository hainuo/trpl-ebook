# 'The Rust Programming Language' as EBook
# Rust编程语言 电子书

由 @hainuo 翻译成中文，由于第一次翻译科技文献，不太熟悉英语的表达方式，所以可能有不少地方不到位请大家见谅，并指出其中翻译错误地方，以便 @hainuo 进行修正；另外由于是根据个人兴趣进行翻译，可能你所需要的章节没有翻译，如果你想加入到翻译中来，请fork 并提交pull request

##翻译进度（已经完成章节列表）
+ 第四章  高效的Rust
  - 第一节  内存管理
  - 第二节  测试用例

  
This repository contains stuff to convert [this book](http://doc.rust-lang.org/book/) to HTML, EPUB and PDF.

本仓库主要是将doc.rust-lang.org/book 转换成html，epub和pdf。

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

## License  许可

The book content itself as well as any code I added as part of this repository is Copyright (c) 2015 The Rust Project Developers and licensed like Rust itself ([MIT](https://github.com/rust-lang/rust/blob/master/LICENSE-MIT) and [Apache](https://github.com/rust-lang/rust/blob/master/LICENSE-APACHE)).
本书内容及我所加入的本仓库中的任何代码都是归于rust 项目开发者所有，并且继承于Rust语言的许可MIT和Apache。
