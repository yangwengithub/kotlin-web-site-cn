---
type: doc
layout: reference
category: FAQ
title: "与 Scala 比较【官方已删除】"
---

# 与 Scala 比较【官方已删除】

Kotlin 团队的主要目标是创建一种务实且高效的编程语言，而不是提高编程语言研究中的最新技术水平。
考虑到这一点，如果你对 Scala 感到满意，那你很可能不需要 Kotlin。

## Scala 有而 Kotlin没有的东西

* 隐式转换、参数……等等
    * 在 Scala 中，由于画面中有太多的隐式转换，有时不使用 debugger 会很难弄清代码中具体发生了什么
    * 在 Kotlin 中使用[扩展函数](extensions.html)来给类型扩充功能/函数（双关：functions）。
* 可覆盖的类型成员
* 路径依赖性类型
* 宏
* 存在类型
    * [类型投影](generics.html#类型投影)是一种非常特殊的情况
* 特性（trait）初始化的复杂逻辑
    * 参见[类与接口](classes.html)
* 自定义符号操作
    * 参见[操作符重载](operator-overloading.html)
* 结构类型
* 值类型
    * 我们计划支持[Project Valhalla](http://openjdk.java.net/projects/valhalla/)当它作为 JDK 一部分发布时。
* yield 操作符与 actor
    * 参见[协程](coroutines.html)
* 并行集合
    * Kotlin 支持 Java 8 streams，它提供了类似的功能

## Kotlin 有而 Scala 没有的东西

* [零开销空安全](null-safety.html)
    * Scala 有 Option，它是一个语法糖以及运行时的包装器
* [智能转换](typecasts.html)
* [Kotlin的内联函数便于非局部跳转](inline-functions.html#内联函数)
* [一等公民的委托](delegation.html)。也通过第三方插件 Autoproxy 实现
