---
type: tutorial
layout: tutorial
title: "使用命令行编译器的 Hello Kotlin/Native"
description: "了解如何使用命令行编译器编译 Kotlin/Native 应用程序"
authors: 
  - Hadi Hariri
  - Yue_plus（翻译）
date: 2020-01-15
---

<!--- To become a How-To. Need to change type to new "HowTo" --->


## 获取编译器

Kotlin/Native 编译器可用于 macOS、Linux 与 Windows。
它可以作为命令行工具使用，也可以作为标准 Kotlin 发行版的一部分提供，可以从 [GitHub Releases]({{ site.data.releases.latest.url }}) 下载。
它支持不同的目标，包括：iOS（arm32、arm64、模拟器 x86_64）、Windows（mingw32 与 x86_64），Linux（x86_64、arm64、MIPS）、macOS（x86_64）、<span title="Raspberry PI">树莓派</span>、SMT32、WASM。
有关目标的完整列表，请参阅[Kotlin/Native 用于原生开发](/docs/reference/native-overview.html)。

尽管可以进行跨平台编译，这意味着可以使用一个平台针对另一个平台进行编译，
在这种情况下，将针对正在编译的平台。

尽管编译器的输出没有任何依赖性或虚拟机要求，
但编译器本身需要 [Java 1.8 或更高版本来运行](https://jdk.java.net/11/)。

## 创建 Hello Kotlin/Native

该应用程序将在标准输出上打印 "Hello Kotlin/Native"。
在选择的工作目录中，创建一个名为 `hello.kt` 的文件，然后输入以下内容：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun main() {
  println("Hello Kotlin/Native!")
}
```
</div>

## 从控制台编译代码

要编译该应用程序，
请使用[下载](https://github.com/JetBrains/kotlin/releases)的编译器执行以下命令：

```bash
kotlinc-native hello.kt -o hello
```

`-o` 选项的值指定输出文件的名称，因此此调用应生成 `hello.kexe`（Linux 或 macOS）或 `hello.exe`（Windows）二进制文件。
有关可用的编译器选项的完整列表，请参阅 [Kotlin 编译器选项](/docs/reference/compiler-reference.html)。

尽管从控制台进行的编译看起来很简单明了，
但是对于具有数百个文件与库的大型项目而言，它的可拓展性并不理想。
对于实际项目，建议使用[构建系统](using-gradle.html)与 [IDE](using-intellij-idea.html)。
