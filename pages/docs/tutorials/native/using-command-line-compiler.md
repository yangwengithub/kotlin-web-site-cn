---
type: tutorial
layout: tutorial
title: "使用命令行编译器的 Hello Kotlin/Native"
description: "了解如何使用命令行编译器编译本地应用程序"
authors: 
  - Hadi Hariri
date: 2020-01-15
---

<!--- To become a How-To. Need to change type to new "HowTo" --->


## 获取编译器

Kotlin/Native 编译器适用于 macOS、Linux 及 Windows。它是一个命令行工具，作为标准 Kotlin 发行版的一部分提供，可以从 [GitHub 发行版]({{site.data.releases.latest.url }})下载。它支持
不同的目标包括 iOS (arm32、arm64、simulator x86_64)、Windows (mingw32 及 x86_64)，
Linux (x86_64、arm64、MIPS)、macOS (x86_64)、Raspberry PI、SMT32、WASM。有关目标的完整列表，请参见 [Kotlin/Native 概述](/docs/reference/native-overview.html)。

尽管可以进行跨平台编译，这意味着可以使用一个平台针对另一个平台进行编译，
但在这种情况下，我们将针对正在编译的平台。

尽管编译器的输出没有任何依赖性或虚拟机要求，
但编译器本身需要 [Java 1.8 或 更高的运行时](https://jdk.java.net/11/)。

## 创建 Hello Kotlin/Native

该应用程序将在标准输出上打印 "Hello Kotlin/Native"。在选择的工作目录中，创建一个名为
`hello.kt`,并输入以下内容:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun main() {
  println("Hello Kotlin/Native!")
}
```
</div>

## 从控制台编译代码

要编译这个应用程序，请使用[下载项](https://github.com/JetBrains/kotlin/releases)

编译器执行以下命令:

```bash
kotlinc-native hello.kt -o hello
```

`-o` 选项的值指定了输出文件的名称，所以这个调用应该生成一个 `hello.kexe` (Linux 及 macOS) 或 `hello.exe` (Windows) 二进制文件。
有关可用编译器选项的完整列表，请参见[编译器选项参考](/docs/reference/compiler-reference.html)。

虽然从控制台编译看起来简单明了，但它
对于包含数百个文件和库的大型项目来说，这种方法不太适用。对于现实项目，建议这样做
使用[构建系统](using-gradle.html)和[集成开发环境](using-intellij-idea.html)。
