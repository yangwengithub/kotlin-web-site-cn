---
type: doc
layout: reference
category: "Introduction"
title: "协程概述"
---

# 用于异步编程等场景的协程

异步或非阻塞程序设计是新的现实。无论我们创建服务端应用、桌面应用还是移动端应用，都很重要的一点是，
我们提供的体检不仅是从用户角度看着流畅，而且还能在需要时伸缩（scalable，可扩充/缩减规模）。

这个问题有很多方法，在 Kotlin 中我们采用非常灵活的方法，在语言级提供[协程](https://en.wikipedia.org/wiki/Coroutine)支持，
而将大部分功能委托给库，这与 Kotlin 的理念非常一致。

额外收益是，协程不仅打开了异步编程的大门，还提供了大量其他的可能性，例如并发、参与者（actor）等。


## 如何开始

<div style="display: flex; align-items: center; margin-bottom: 20px">
    <img src="{{ url_for('asset', path='images/landing/native/book.png') }}" height="38p" width="55" style="margin-right: 10px;">
    <b>Tutorials and Documentation</b>
</div>

Kotlin 新手？可以看看[入门](/docs/reference/basic-syntax.html)页。

精选文档页：
- [协程指南](/docs/reference/coroutines/coroutines-guide.html)
- [基础](/docs/reference/coroutines/basics.html)
- [通道](/docs/reference/coroutines/channels.html)
- [协程上下文与调度器](/docs/reference/coroutines/coroutine-context-and-dispatchers.html)
- [共享的可变状态与并发](/docs/reference/coroutines/shared-mutable-state-and-concurrency.html)

推荐的教程：
- [你的第一个 Kotlin 协程程序](../tutorials/coroutines/coroutines-basic-jvm.html)
- [异步程序设计](../tutorials/coroutines/async-programming.html)

<div style="display: flex; align-items: center; margin-bottom: 10px;">
    <img src="{{ url_for('asset', path='images/landing/native/try.png') }}" height="38p" width="55" style="margin-right: 10px;">
    <b>Example Projects</b>
</div>

- [kotlinx.coroutines 示例与源代码](https://github.com/Kotlin/kotlin-coroutines/tree/master/examples)
- [KotlinConf app](https://github.com/JetBrains/kotlinconf-app)

在 [GitHub](https://github.com/JetBrains/kotlin-examples) 上还有更多示例
