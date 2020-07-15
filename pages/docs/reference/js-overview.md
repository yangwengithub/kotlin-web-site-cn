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


## Kotlin/JS 实践实验室

实践实验室是一种长形式的教程，通过一个与特定主题相关的独立项目来帮助了解一种技术。

它们包括示例项目，这些示例项目可用作自己的项目的起点，并包含有用的代码片段与模式。

对于 Kotlin/JS，当前有以下实践实验：

* [使用 React 与 Kotlin/JS 构建 Web 应用程序](https://play.kotlinlang.org/hands-on/Building%20Web%20Applications%20with%20React%20and%20Kotlin%20JS/01_Introduction)将指导完成使用 React 框架构建简单 Web 应用程序的过程，展示用于 HTML 的类型安全的 Kotlin DSL 如何使构建响应式 DOM 元素更加方便，并说明了如何使用第三方 React 组件，以及如何从 API 获取信息，同时使用纯 Kotlin/JS 编写整个应用程序逻辑。

* [使用 Kotlin Multiplatform 构建全栈 Web 应用](https://play.kotlinlang.org/hands-on/Full%20Stack%20Web%20App%20with%20Kotlin%20Multiplatform/01_Introduction)通过构建使用通用代码、序列化与其他多平台范式的客户端服务器应用程序，讲授了构建针对 Kotlin/JVM 与 Kotlin/JS 的应用程序的概念。它还简要介绍了如何将 Ktor 作为服务器与客户端框架使用。


## 加入 Kotlin/JS 社区
还可以在官方 [Kotlin Slack](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) 中加入 [#javascript](https://kotlinlang.slack.com/archives/C0B8L3U69) 频道，并与社区和团队聊天。
