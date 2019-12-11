---
type: doc
layout: reference
category: "Other"
title: "多平台程序设计"
---

# 多平台程序设计

> 多平台项目是 Kotlin 1.2 与 1.3 中的实验性特性。本文档中描述的所有的语言<!--
-->与工具特性在未来的版本中都可能会有所变化。
{:.note}

在所有平台上都能用是 Kotlin 的一个明确目标，但我们将其视为一个更重要的目标<!--
-->——在多个平台之间共享代码的前提。有了对 JVM、Android、JavaScript、iOS、Linux、Windows、
Mac 甚至像 STM32 这样的嵌入式系统的支持，Kotlin 可以处理现代应用程序的任何组件与所有组件。
这为代码与专业知识的复用带来了宝贵的收益，节省了工作量去完成更具<!--
-->挑战任务，而不是将所有东西都实现两次或多次。

## 它是如何工作的

总得来说，多平台并不是为所有平台编译全部代码。这个模型有其明显的<!--
-->局限性，我们知道现代应用程序需要访问<!--
-->其所运行平台的独有特性。Kotlin 并不会限制你只使用其中所有 API 的公共子集。
每个组件都可以根据需要与其他组件共享尽可能多的代码，
而通过语言所提供的 [`expect`/`actual` 机制](platform-specific-declarations.html)可以随时访问平台 API。

以下是在极简版日志框架中公共逻辑与平台逻辑之间代码共享与交互的<!--
-->示例。其公共代码如下所示：

<div style="display:flex">
<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
enum class LogLevel {
    DEBUG, WARN, ERROR
}

internal expect fun writeLogMessage(message: String, logLevel: LogLevel)

fun logDebug(message: String) = writeLogMessage(message, LogLevel.DEBUG)
fun logWarn(message: String) = writeLogMessage(message, LogLevel.WARN)
fun logError(message: String) = writeLogMessage(message, LogLevel.ERROR)
```

</div>
<div style="margin-left: 5px;white-space: pre-line; line-height: 18px; font-family: Tahoma;">
    <div style="display:flex">├<i style="margin-left:5px">为所有平台编译</i></div>
    <div style="display:flex">├<i style="margin-left:5px">预期的平台相关 API</i></div>
    <div style="display:flex">├<i style="margin-left:5px">可在公共代码中使用预期的 API</i></div>
</div>
</div>

它期待目标平台为 `writeLogMessage` 提供平台相关实现，然后公共代码<!--
-->就可以使用此声明而无需考虑它是如何实现的。

在 JVM 上，可以提供一个将日志写到标准输出的实现：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
internal actual fun writeLogMessage(message: String, logLevel: LogLevel) {
    println("[$logLevel]: $message")
}
```

</div>

在 JavaScript 世界中可用的是一组完全不同的 API，
因此可以实现为将日志记录到控制台：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
internal actual fun writeLogMessage(message: String, logLevel: LogLevel) {
    when (logLevel) {
        LogLevel.DEBUG -> console.log(message)
        LogLevel.WARN -> console.warn(message)
        LogLevel.ERROR -> console.error(message)
    }
}
```

</div>

在 1.3 中我们重新设计了整个多平台模型。我们用于描述多平台 Gradle 项目的[新版 DSL](building-mpp-with-gradle.html)
更加灵活，我们会继续努力使项目配置更加简单。

## 多平台库

公共代码可以依赖于一组涵盖日常任务的库，例如 [HTTP](https://ktor.kotlincn.net/clients/http-client/multiplatform.html)、 [serialization](https://github.com/Kotlin/kotlinx.serialization) 以及[协程<!--
-->管理](https://github.com/Kotlin/kotlinx.coroutines)。此外，丰富的标准库在所有平台上都可用。

你可以随时编写<!--
-->自己的库，提供一个公共的 API，而在每个平台上以不同的方式实现。

## 使用场景

### Android——iOS

移动平台之间共享代码是 Kotlin 多平台的主要使用场景之一，现在<!--
-->可以通过在 Android 与 iOS 之间共享部分代码（如业务逻辑、连接等）
来构建移动应用。

See: 
- [Mobile Multiplatform features, case studies and examples](https://www.jetbrains.com/lp/mobilecrossplatform/)
- [Setting up a Mobile Multiplatform Project](/docs/tutorials/native/mpp-ios-android.html)

### 客户端——服务端

代码共享可以带来收益的另一个场景是互联应用，其中的逻辑可以<!--
-->在服务器与运行在浏览器中的客户端中复用。Kotlin 多平台也覆盖了<!--
-->这个场景。

[Ktor 框架](https://ktor.kotlincn.net/)适用于在互联系统中构建异步的服务器与客户端。

## 如何开始

<div style="display: flex; align-items: center; margin-bottom: 20px">
    <img src="{{ url_for('asset', path='images/landing/native/book.png') }}" height="38p" width="55" style="margin-right: 10px;">
    <b>教程与文档</b>
</div>

Kotlin 新手？可以看看[入门][Getting Started](basic-syntax.html)页。

建议的文档页：
- [搭建一个多平台项目](building-mpp-with-gradle.html#搭建一个多平台项目)
- [平台相关声明](platform-specific-declarations.html)

推荐的教程：
- [多平台 Kotlin 库](https://www.kotlincn.net/docs/tutorials/mpp/multiplatform-library.html)
- [多平台项目：iOS 与 Android](https://www.kotlincn.net/docs/tutorials/native/mpp-ios-android.html)

<div style="display: flex; align-items: center; margin-bottom: 10px;">
    <img src="{{ url_for('asset', path='images/landing/native/try.png') }}" height="38p" width="55" style="margin-right: 10px;">
    <b>示例项目</b>
</div>

- [KotlinConf app](https://github.com/JetBrains/kotlinconf-app)
- [KotlinConf Spinner app](https://github.com/jetbrains/kotlinconf-spinner)

在 [GitHub](https://github.com/JetBrains/kotlin-examples) 上还有更多示例
