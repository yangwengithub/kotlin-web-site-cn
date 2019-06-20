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
Linux（x86_64, arm64, MIPS），macOS（x86_64），Raspberry PI、SMT32、WASM。所有的目标平台列表<!--
-->我们可以在 [Kotlin/Native overview](/docs/reference/native-overview.html) 中查看。
虽然跨平台的编译是可能的，
(即，使用一个平台为另一个平台编译)，但在这个第一篇教程中<!--
-->我们仅仅为我们当前运行的系统进行编译。

使用 Kotlin/Native 编译器的最好的方式是去构建一个系统。
它有助于下载和缓存 Kotlin/Native 编译器二进制文件与库
传递依赖，并运行编译器以及测试。
它同样缓存编译结果。
IDE 还可以使用构建系统来了解项目布局。

Kotlin/Native 通过 [Gradle](https://gradle.org) 的
[kotlin-multiplatform](/docs/reference/building-mpp-with-gradle.html) 插件来使用构建系统。
我们将在下面介绍如何配置 Gradle 构建。
对于某些极端情况，Kotlin/Native 编译器仍然可以从
[GitHub Kotlin 发行版页面](https://github.com/JetBrains/kotlin/releases)手动获取（不推荐）。
在本教程中，我们将专注于使用 Gradle 进行构建。

虽然编译器的输出没有任何依赖项或虚拟机要求，
但编译器本身以及 Gradle 构建系统需要 Java 1.8 或 11 runtime。在网页
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
-->编译器手动生成一个 `hello.kexe`（Linux 以及 macOS）或 `hello.exe`（Windows）
二进制文件

```bash
kotlinc-native hello.kt -o hello
```

虽然从控制台编译似乎很简单，但我们应该注意到它<!--
-->对于包含数百个文件和库的大型项目来说，不能很好地扩展。
除此之外，命令行方法没有向 IDE 解释如何打开这样的项目，
源所在的位置，使用的依赖项，或依赖项的下载方式等。

<a name="create-gradle-project"></a>
## 创建一个 Kotlin/Native Gradle 工程

The _New Project_ wizard in IntelliJ IDEA can be used to start a new Kotlin/Native project with just one click.
Check out the _Kotlin_ section and select the _Native | Gradle_ option to generate the project.
For a better understanding and to explain what's happening, in this tutorial we'll create the project manually.

Let's first create a project folder. All the paths in this tutorial will be relative to this folder. Sometimes
the missing directories will have to be created before new files are added.

Gradle supports two languages for the build scripts. We have the following
options:
- Groovy scripts in `build.gradle` files
- Kotlin scripts in `build.gradle.kts` files

Groovy language is the oldest supported scripting language for Gradle,
it leverages the power of the dynamic typing and runtime features of the language.
Sometimes it can be harder to maintain Groovy build scripts. IDEs are struggling
to get through the dynamism of Groovy to provide helpful insights or
code completion.

Kotlin as a statically typed programming language plays well for writing
Gradle build scripts.
Thanks to the static type inference, the Kotlin compiler detects errors earlier and
shows important compilation error messages and warnings.
Both an IDE and the compiler can use the information about types to infer
the available functions and properties in a given scope.

We create
<span class="multi-language-span" data-lang="groovy">
`build.gradle`
</span>
<span class="multi-language-span" data-lang="kotlin">
`build.gradle.kts`
</span>
Gradle build file with the following contents:
[[include pages-includes/docs/tutorials/native/basic-kotlin-native-app-codeblocks-code.md]]

The prepared project sources can be directly downloaded from
[[include pages-includes/docs/tutorials/native/basic-kotlin-native-app-codeblocks-link.md]]

Now need to create an empty
<span class="multi-language-span" data-lang="kotlin">
`settings.gradle.kts`
</span><span class="multi-language-span" data-lang="groovy">
`settings.gradle`
</span>
file in the project root directory.

Depending on the target platform, we use different [functions](/docs/reference/building-mpp-with-gradle.html),
e.g. `macosX64`, `mingwX64`, `linuxX64`, `iosX64`,
to create the Kotlin target. The function name is the platform which we are compiling our code for.
These functions optionally take the target name as a parameter, which is `"native"` in our case.
The specified _target name_ is used to generate the source paths and task names in the project.

By the convention, all sources are located in the `src/<target name>[Main|Test]/kotlin` folders.
It creates _main_ and _test_ source sets for every target. Let's place the `hello.kt`
we previously created into the _main_ source set folder, which is `src/nativeMain/kotlin`.
The `nativeMain` folder comes from the `"native"` target name, which we specified in the build script above.

The project is ready. The next step is to open it in IntelliJ IDEA. For advanced build scenarios,
it is recommended to refer to the
[more detailed](/docs/reference/building-mpp-with-gradle.html#setting-up-a-multiplatform-project)
documentation.

Anyone wanting to continue on without an IDE, will need to download and install the
[Gradle](https://gradle.org) build tool.
Make sure to use the right version of Gradle (e.g. {{ site.data.releases.tutorials.native.gradle_version }} or newer).
Running the `gradle wrapper` command will complete the project creation.
[Getting Started with Gradle](https://docs.gradle.org/current/userguide/getting_started.html)
explains in detail how to start using Gradle projects.

<a name="open-in-ide"></a>
## 在 IDE 中打开这个工程

We are using [IntelliJ IDEA](https://jetbrains.com/idea) for this tutorial.
Both the [free and open source](https://www.jetbrains.com/idea/features/editions_comparison_matrix.html)
IntelliJ IDEA [Community Edition](https://www.jetbrains.com/idea/download) and
IntelliJ IDEA Ultimate Edition work for this tutorial.
We can download and install both of them from [https://jetbrains.com/idea/download](https://jetbrains.com/idea/download) if necessary.
The Kotlin plugin is included with IntelliJ IDEA by default, but still, we need to make sure the Kotlin plugin version
is {{ site.data.releases.latest.version }} (or newer) in the _Settings_ or _Preferences_ dialog, under
the Language & Frameworks | Kotlin section.


At this point, we should have a Gradle project that is ready to be opened in an IDE.
IntelliJ IDEA (CLion, AppCode, or AndroidStudio) helps us to generate the
[Gradle Wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html)
scripts for our project.

Now let's open the project in IntelliJ IDEA. For that we click on the File | Open... and select
our
<span class="multi-language-span" data-lang="kotlin">
`build.gradle.kts`
</span><span class="multi-language-span" data-lang="groovy">
`build.gradle`
</span>
project file.

![Open Project Dialog]({{ url_for('tutorial_img', filename='native/basic-kotlin-native/idea-open-as-project.png')}}){: width="70%"}

Confirm to open the file _as Project_.

![Gradle Import Dialog]({{ url_for('tutorial_img', filename='native/basic-kotlin-native/idea-import-gradle.png')}})

Select _Use gradle 'wrapper' task configuration_ option in the Gradle import dialog to complete the import.
For existing projects, which already have Gradle wrapper scripts, the _Use default Gradle wrapper_
option should be selected instead.

Use the path to the Java runtime version 1.8 or 11 for the _Gradle JVM_ field. Check out the
[https://jdk.java.net/11](https://jdk.java.net/11/) or [https://adoptopenjdk.net/](https://adoptopenjdk.net/)
for the best JRE, OpenJDK, or JDK distribution.

<a name="run-in-ide"></a>
## 运行应用程序

Usually, a native binary can be compiled as _debug_ with more debug information and fewer optimizations, and _release_
where optimizations are enabled and there is no (or at least less) debug information available.
The binary files are created in the `build/bin/native/debugExecutable` or `build/bin/native/releaseExecutable`
folders respectively. The file has a `.kexe` extension on Linux and macOS and an `.exe` extension on Windows. Use the following command
to instruct the build to produce binaries:

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

It's important to understand that this is now a native application, and no
runtime or virtual machine is required.
We can now run the compiled binary from the console:

```
Hello Kotlin/Native!
```

In addition to the build tasks, the Gradle build includes helpful
tasks to run the application directly via
`runDebugExecutableNative` and `runReleaseExecutableNative`.

The names of these tasks were created from the formula:
`run[Debug|Release]Executable<target name>`,
where `target name` is the capitalized target name that we specified in the
<span class="multi-language-span" data-lang="kotlin">
`build.gradle.kts`
</span><span class="multi-language-span" data-lang="groovy">
`build.gradle`
</span>
file out of our build, `"native"` in our case.
Let's run the task in the IDE. For that, let's open the Gradle Tool Window
and find the task in the list:
![Gradle Import Dialog]({{ url_for('tutorial_img', filename='native/basic-kotlin-native/idea-run-gradle-task.png')}})

Alternatively, we may call the following command from the console:
[[include pages-includes/docs/tutorials/native/runDebugExecutableNative.md]]

The output should be:

```
> Task :runDebugExecutableNative
Hello Kotlin/Native!

BUILD SUCCESSFUL
```

## 接下来

Kotlin/Native can be used for many
[targets](targeting-multiple-platforms.html) and applications,
including, but not limited to
macOS, Windows, Linux, and [iOS](/docs/tutorials/native/mpp-ios-android.html).

Calling C, Objective-C, or Swift from Kotlin/Native is easy. Take a look at
the [C Interop documentation](/docs/reference/native/c_interop.html) or
[Objective-C and Swift](/docs/reference/native/objc_interop.html) interop
documentation or check out one of our tutorials.

With Kotlin [multiplatform](/docs/reference/multiplatform.html) projects, it is possible to
share the same Kotlin code between all the supported platforms.
Check out the tutorial on [sharing Kotlin code between iOS and Android](/docs/tutorials/native/mpp-ios-android.html)
or have a look at how to build your own [multiplatform library](/docs/tutorials/multiplatform-library.html).
