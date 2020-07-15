---
type: doc
layout: reference
category: "Introduction"
title: "Kotlin 用于 JavaScript 开发"
---

# Kotlin JavaScript 概述

Kotlin 提供了 JavaScript 作为目标平台的能力。它通过将 Kotlin 转换为 JavaScript 来实现。目前的实现目标是 ECMAScript 5.1，但也有最终目标为 ECMAScript 2015 的计划。

当你选择 JavaScript 目标时，作为项目一部分的任何 Kotlin 代码以及 Kotlin 附带的标准库都会转换为 JavaScript。然而，这不包括使用的 JDK 和任何 JVM 或 Java 框架或库。任何不是 Kotlin 的文件会在编译期间忽略掉。

Kotlin 编译器努力遵循以下目标：

* 提供最佳大小的输出
* 提供可读的 JavaScript 输出
* 提供与现有模块系统的互操作性
* 在标准库中提供相同的功能，无论是 JavaScript 还是 JVM 目标（尽最大可能程度）。

## 如何使用

你可能希望在以下情景中将 Kotlin 编译为 JavaScript：

* 创建面向客户端 JavaScript 的 Kotlin 代码

    * **与 DOM 元素交互**。Kotlin 提供了一系列静态类型的接口来与文档对象模型（Document Object Model）交互，允许创建和更新 DOM 元素。

    * **与图形如 WebGL 交互**。你可以使用 Kotlin 在网页上用 WebGL 创建图形元素。

* 创建面向服务器端 JavaScript 的 Kotlin 代码

    * **使用服务器端技术**。你可以使用 Kotlin 与服务器端 JavaScript（如 Node.js）进行交互

Kotlin 可以与现有的第三方库和框架（如 jQuery 或 ReactJS）一起使用。要使用强类型
API 访问第三方框架，可以使用 [dukat](https://github.com/kotlin/dukat) 工具将 TypeScript 定义从 [Definitely Typed](http://definitelytyped.org/)
类型定义仓库转换为 Kotlin。或者，你可以使用<!--
-->[动态类型](dynamic-type.html)访问任何框架，而无需强类型。

Kotlin 兼容 CommonJS、AMD 和 UMD，直截了当[与不同的模块系统交互](https://www.kotlincn.net/docs/tutorials/javascript/working-with-modules/working-with-modules.html)。


## Kotlin/JS 入门

要了解如何开始使用 Kotlin 用于 JavaScript 开发，请参阅[搭建 Kotlin/JS 项目](/docs/reference/js-project-setup.html)。


## Hands-on labs for Kotlin/JS

Hands-on labs are long-form tutorials that help you get to know a technology by guiding you through a self-contained project related to a specific topic.

They include sample projects, which can serve as jumping-off points for your own projects, and contain useful snippets and patterns.

For Kotlin/JS, the following hands-on labs are currently available:

* [Building Web Applications with React and Kotlin/JS](https://play.kotlinlang.org/hands-on/Building%20Web%20Applications%20with%20React%20and%20Kotlin%20JS/01_Introduction) guides you through the process of building a simple web application using the React framework, shows how a typesafe Kotlin DSL for HTML makes it convenient to build reactive DOM elements, and illustrates how to use third-party React components, and how to obtain information from APIs, while writing the whole application logic in pure Kotlin/JS.

* [Building a Full Stack Web App with Kotlin Multiplatform](https://play.kotlinlang.org/hands-on/Full%20Stack%20Web%20App%20with%20Kotlin%20Multiplatform/01_Introduction) teaches the concepts behind building an application that targets Kotlin/JVM and Kotlin/JS by building a client-server application that makes use of common code, serialization, and other multiplatform paradigms. It also provides a brief introduction into working with Ktor both as a server- and client-side framework.


## Join Kotlin/JS community
You can also join [#javascript](https://kotlinlang.slack.com/archives/C0B8L3U69) channel in the official [Kotlin Slack](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and chat with the community and the team.
