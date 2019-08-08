---
type: tutorial
layout: tutorial
title:  "基本 Kotlin/Native 应用程序"
description: "来看看如何编译我们的第一个 Kotlin/Native 应用程序并且在 IDE 中打开它"
authors:
  - Hadi Hariri
  - Eugene Petrenko，乔禹昂（翻译）
date: 2019-04-15
---


在本教程中，我们将看到如何：

* [获取 Kotlin/Native 编译器](#获取编译器)
* [编写应用程序](#创建-hello-kotlin)
* [创建构建文件](#创建一个-kotlinnative-gradle-工程)
* [配置 IDE](#在-ide-中打开这个工程)
* [运行应用程序](#运行应用程序)


## 获取编译器

Kotlin/Native 编译器可以被用于 macOS、Linux 以及 Windows。它支持<!--
-->不同的目标平台包括 iOS（arm32、arm64、simulator x86_64），Windows（mingw32 以及 x86_64），
Linux（x86_64、arm64、MIPS），macOS（x86_64），Raspberry PI、SMT32、WASM。所有的目标平台列表<!--
-->我们可以在 [Kotlin/Native 总览](/docs/reference/native-overview.html) 中查看。
虽然跨平台编译是可行的，
(即，使用一个平台为另一个平台编译)，但在这个第一篇教程中<!--
-->我们仅仅为当前运行的系统进行编译。

使用 Kotlin/Native 编译器的最好的方式是去构建一个系统。
它有助于下载与缓存 Kotlin/Native 编译器二进制文件与库<!--
-->传递依赖，并运行编译器以及测试。
它同样也缓存编译结果。
IDE 还可以使用构建系统来了解项目布局。

Kotlin/Native 通过 [Gradle](https://gradle.org) 的
[kotlin-multiplatform](/docs/reference/building-mpp-with-gradle.html) 插件来使用构建系统。
我们将在下面介绍如何配置 Gradle 构建。
对于某些极端情况，Kotlin/Native 编译器仍然可以从
[GitHub Kotlin 发行版页面](https://github.com/JetBrains/kotlin/releases)手动获取（不推荐）。
在本教程中，我们将专注于使用 Gradle 进行构建。

虽然编译器的输出没有任何依赖项或虚拟机要求，
但编译器本身以及 Gradle 构建系统需要 Java 1.8 或 11 运行时。在网页
[https://jdk.java.net/11](https://jdk.java.net/11/) 或另外的源<!--
-->检查，JRE、OpenJDK 或 JDK 发行版。

## 创建 Hello Kotlin

我们的第一个应用程序只简单的使用标准输出打印一些文本。在我们的案例中文本是“Hello Kotlin/Native”。
我们可以打开自己最喜欢的 IDE 或编辑器并在名为 `hello.kt` 的文件中编写下面的代码：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun main() {
  println("Hello Kotlin/Native!")
}
```
</div>

## 使用命令行编译代码

通过[下载](https://github.com/JetBrains/kotlin/releases)的<!--
-->编译器手动生成一个 `hello.kexe`（Linux 与 macOS）或 `hello.exe`（Windows）<!--
-->二进制文件

```bash
kotlinc-native hello.kt -o hello
```

虽然从控制台编译似乎很简单，但我们应该注意到它<!--
-->对于包含数百个文件与库的大型项目来说，不能很好地扩展。
除此之外，命令行方法没有向 IDE 解释如何打开这样的项目、
源所在的位置、使用的依赖项、或依赖项的下载方式等。

<a name="create-gradle-project"></a>
## 创建一个 Kotlin/Native Gradle 工程

我们只需点击 IntelliJ IDEA 中的 _New Project_ 引导项，就可以创建一个新的 Kotlin/Native 工程。
点击 _Kotlin_ 选项，并选择 _Native | Gradle_ 配置项来生成工程。
为了更好地理解与解释正在发生的事情，在本教程中我们将手动创建工程。

让我们首先创建一个工程文件夹。本教程中的所有路径都与此文件夹相关。有时<!--
-->在添加新文件之前，必须创建缺少的目录。

Gradle 的构建脚本支持两种语言。我们有以下的两种<!--
-->选项：
- `build.gradle` 文件中的 Groovy 脚本
- `build.gradle.kts` 文件中的 Kotlin 脚本

Groovy 语言是被 Gradle 脚本支持时间最长的语言，
它利用了语言的动态类型以及运行时特性的强大功能。
有时维护 Groovy 构建脚本可能会更困难。IDE 正在苦苦挣扎于<!--
-->通过 Groovy 的动态特性来提供有用的预测或<!--
-->代码补全。

Kotlin 作为一种静态类型的编程语言非常适合编写
Gradle 构建脚本。
由于静态类型推断，Kotlin 编译器可以更早地检测到错误<!--
-->并显示重要的编译错误消息和警告。
IDE 与编译器都可以使用有关类型的信息进行推断<!--
-->给定范围内的可用函数与属性。

我们创建
<span class="multi-language-span" data-lang="groovy">
`build.gradle`
</span>
<span class="multi-language-span" data-lang="kotlin">
`build.gradle.kts`
</span>
Gradle 构建文件中添加下面的内容：
[[include pages-includes/docs/tutorials/native/basic-kotlin-native-app-codeblocks-code.md]]

准备好的工程源可以直接下载
[[include pages-includes/docs/tutorials/native/basic-kotlin-native-app-codeblocks-link.md]]

现在需要在工程根目录创建一个空的
<span class="multi-language-span" data-lang="kotlin">
`settings.gradle.kts`
</span><span class="multi-language-span" data-lang="groovy">
`settings.gradle`
</span>
文件

根据目标平台，我们使用不同的[函数](/docs/reference/building-mpp-with-gradle.html)，
例如 `macosX64`、`mingwX64`、`linuxX64`、`iosX64`，
来创建一个 Kotlin 目标。函数名称是我们编译代码的平台。
这些函数可选地将目标名称作为参数，在我们的例子中是 `"native"`。
指定的 _目标平台名称_ 用于在项目中生成源路径以及任务名称。

按照惯例，所有的源都位于 `src/<target name>[Main|Test]/kotlin` 文件夹。
它为每个目标平台都创建了 _main_ 以及 _test_ 源集。让我们将 `hello.kt` 放置到<!--
-->我们之前在 `src/nativeMain/kotlin` 中创建的 _main_ 源集文件夹中。
该 `nativeMain` 文件夹名称来源于我们在上面的构建脚本中指明的 `"native"` 目标平台名称。

工程已经准备好了。下一步是在 IntelliJ IDEA 中打开它。关于高级的构建场景，
建议参考<!--
-->[更多细节](/docs/reference/building-mpp-with-gradle.html#setting-up-a-multiplatform-project)<!--
-->文档。

想要在没有 IDE 的情况下继续使用都需要下载并安装
[Gradle](https://gradle.org) 构建工具。
确保使用正确的 Gradle 版本（例如 {{ site.data.releases.tutorials.native.gradle_version }} 或者更新）。
当工程完成创建时运行 `gradle wrapper` 命令。
[从 Gradle 开始](https://docs.gradle.org/current/userguide/getting_started.html)
有更多关于如何开始使用 Gradle 工程的说明。

<a name="open-in-ide"></a>
## 在 IDE 中打开这个工程

在本教程中我们使用 [IntelliJ IDEA](https://jetbrains.com/idea)。
同时包括[自由与开源](https://www.jetbrains.com/idea/features/editions_comparison_matrix.html)的
IntelliJ IDEA [社区版本](https://www.jetbrains.com/idea/download)以及
IntelliJ IDEA 终极版。
如果有必要的话我们可以在 [https://jetbrains.com/idea/download](https://jetbrains.com/idea/download) 同时下载并安装它们两者。
在 IntelliJ IDEA 中 Kotlin 插件已经被默认包含，但是我们仍然需要确保在 Language & Frameworks | Kotlin 选项下的
_Settings_ 或 _Preferences_ 的弹窗中 Kotlin 插件的版本是
{{ site.data.releases.latest.version }}（或者更新）。


此时，我们应该有一个可以在 IDE 中打开的 Gradle 工程。
IntelliJ IDEA（CLion、AppCode 或 AndroidStudio）将帮助我们的工程生成一个
[Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html)
脚本。

现在让我们在 IntelliJ IDEA 中打开这个工程。为此我们点击 File | Open... 并选择<!--
-->我们的
<span class="multi-language-span" data-lang="kotlin">
`build.gradle.kts`
</span><span class="multi-language-span" data-lang="groovy">
`build.gradle`
</span>
工程文件。

![打开工程弹窗]({{ url_for('tutorial_img', filename='native/basic-kotlin-native/idea-open-as-project.png')}}){: width="70%"}

确认 _作为工程_ 打开文件。

![Gradle 导入弹窗]({{ url_for('tutorial_img', filename='native/basic-kotlin-native/idea-import-gradle.png')}})

在 Gradle 导入弹窗中选择 _Use gradle 'wrapper' task configuration_ 选项来完成导入。
对于已有 Gradle wrapper 脚本的现有项目，则应该选择 _Use default Gradle wrapper_
选项替代之。

为 _Gradle JVM_ 属性使用 Java 运行时版本 1.8 或 11 的路径。查看
[https://jdk.java.net/11](https://jdk.java.net/11/) 或者 [https://adoptopenjdk.net/](https://adoptopenjdk.net/)
来确认最优的 JRE、OpenJDK 或 JDK 发行版的版本。

<a name="run-in-ide"></a>
## 运行应用程序

通常，一个原生二进制文件可以被编译为 _debug_ 版本，这时它拥有更多的调试信息以及更少的优化，而 _release_
版本会有更多的优化被启用并且没有（或者至少是更少的）调试信息。
二进制文件会在 `build/bin/native/debugExecutable` 以及 `build/bin/native/releaseExecutable`
目录下分别被创建。该文件在 Linux 以及 macOS 上会拥有一个 `.kexe` 扩展名，而在 Windows 上的扩展名则是 `.exe`。使用以下命令<!--
-->指示构建生成二进制文件：

<div class="multi-language-sample" data-os="linux">
<div class="sample" markdown="1" theme="idea" mode='bash' data-highlight-only>

```bash
./gradlew build
```
</div>
</div>

<div class="multi-language-sample" data-os="macos">
<div class="sample" markdown="1" theme="idea" mode='bash' data-highlight-only>

```bash
./gradlew build
```
</div>
</div>

<div class="multi-language-sample" data-os="windows">
<div class="sample" markdown="1" theme="idea" mode='bash' data-highlight-only>

```bash
gradlew.bat build
```
</div>
</div>

重要的是要了解这是一个原生应用程序，所以不需要<!--
-->运行时环境或虚拟机。
我们现在可以从控制台运行已编译的二进制文件：

```
Hello Kotlin/Native!
```

除了构建任务之外，Gradle 构建还包括帮助<!--
-->通过直接运行应用程序的任务
`runDebugExecutableNative` 以及 `runReleaseExecutableNative`。

这些任务的名称是根据公式创建的：
`run[Debug|Release]Executable<target name>`，
其中 `target name` 我们在构建脚本中指定的大写目标名称：
<span class="multi-language-span" data-lang="kotlin">
`build.gradle.kts`
</span><span class="multi-language-span" data-lang="groovy">
`build.gradle`
</span>
在我们的案例中，`"native"` 是构建后输出的文件。
让我们在 IDE 中运行任务。为此，让我们打开 Gradle Tool 窗口<!--
-->并在列表中找到任务：
![Gradle Import Dialog]({{ url_for('tutorial_img', filename='native/basic-kotlin-native/idea-run-gradle-task.png')}})

或者，我们可以从控制台调用以下命令：
[[include pages-includes/docs/tutorials/native/runDebugExecutableNative.md]]

输出应该是：

```
> Task :runDebugExecutableNative
Hello Kotlin/Native!

BUILD SUCCESSFUL
```

## 接下来

Kotlin/Native 可以被用于许多<!--
-->[目标平台](targeting-multiple-platforms.html)或应用程序，
包括但不限于
macOS、Windows、Linux 以及 [iOS](/docs/tutorials/native/mpp-ios-android.html)。

调用 C、Objective-C 或 Swift 在 Kotlin/Native 中是非常简单的。只需要查看<!--
-->我们的文档中的 [C 互操作文档](/docs/reference/native/c_interop.html)或
[Objective-C 与 Swift](/docs/reference/native/objc_interop.html) 互操作<!--
-->文档。

至于 Kotlin [多平台](/docs/reference/multiplatform.html)项目，它可以用于<!--
-->将 Kotlin 代码共享至所有支持的平台。
查看文档[在 iOS 与 Android 之间共享 Kotlin 代码](/docs/tutorials/native/mpp-ios-android.html)<!--
-->或者查看如何构建自己的[多平台库](/docs/tutorials/multiplatform-library.html)。
