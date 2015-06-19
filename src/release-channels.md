% Release Channels   发行通道

The Rust project uses a concept called ‘release channels’ to manage releases. It’s important to understand this process to choose which version of Rust your project should use.

Rust项目使用了一个叫做‘发行通道’的概念来管理发布。这对于理解你的项目应该应该选择哪一个版本的Rust来使用的过程很重要。

# Overview  概述

There are three channels for Rust releases:

rust发行版一共有三个通道：

* Nightly 每日版本
* Beta   测试版本
* Stable  稳定版本

New nightly releases are created once a day. Every six weeks, the latest nightly release is promoted to ‘Beta’. At that point, it will only receive patches to fix serious errors. Six weeks later, the beta is promoted to ‘Stable’, and becomes the next release of `1.x`.

每日版本是一天创建一次。每6个星期，最后一个每日版本被提升为测试版本。在这一个点上，它将只接受修复验证错误的补丁。6周后，测试版本被晋升为正式版，并成为`1.x`的下一个版本。

This process happens in parallel. So every six weeks, on the same day, nightly goes to beta, beta goes to stable. When `1.x` is released, at the same time, `1.(x + 1)-beta` is released, and the nightly becomes the first version of `1.(x + 2)-nightly`.

这个过程是平行发生的。所以每6周，在同一天，每日版本变成测试版，测试版本变成稳定版。当`1.x`被发布了，在同时`1.(x+1)-beta`也被发布，并且，每日版本变成了`1.(x+2)-nightly`的第一个版本。

# Choosing a version 选择版本

Generally speaking, unless you have a specific reason, you should be using the stable release channel. These releases are intended for a general audience.

一般说来，除非你有一个特殊原因，你应该使用稳定版本通道。这些版本都是面向普通用户的。

However, depending on your interest in Rust, you may choose to use nightly instead. The basic tradeoff is this: in the nightly channel, you can use unstable, new Rust features. However, unstable features are subject to change, and so any new nightly release may break your code. If you use the stable release, you cannot use experimental features, but the next release of Rust will not cause significant issues through breaking changes.

然而，取决于你对Rust的兴趣，你可以攒泽使用每日版本。基本的权衡是这样的：在每日版本通道中，你可以使用不稳定的，新的Rust特性。但是，不稳定的特性可能会产生改变，因此任何新的每日版本都可以破坏你的代码。如果你使用了稳定版本，则不能够使用实验性特性，但是Rust的下一个版本将不会因为阻止变化产生明显的问题。

# Helping the ecosystem through CI  通过CI来帮助生态链

What about beta? We encourage all Rust users who use the stable release channel to also test against the beta channel in their continuous integration systems.
This will help alert the team in case there’s an accidental regression.

为什么要测试？我们鼓励所有的使用稳定版本通道的Rust用户同样在他们的持续集成系统中尝试使用测试版本。

Additionally, testing against nightly can catch regressions even sooner, and so if you don’t mind a third build, we’d appreciate testing against all channels.

此外，针对每日版本的测试能够更早的抓住回归，如果你不介意第三个版本，我们非常感谢你测试所有的通道。