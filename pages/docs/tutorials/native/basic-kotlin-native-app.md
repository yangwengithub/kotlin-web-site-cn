---
type: tutorial
layout: tutorial
title:  "基本 Kotlin/Native 应用程序"
description: "看看如何编译我们的第一个 Kotlin/Native 应用程序"
authors: Hadi Hariri 
date: 2017-12-03
showAuthorInfo: false
---


在本教程中我们将会学习到：

* [获取 Kotlin/Native 编译器](#obtaining-the-compiler)
* [编写应用程序](#creating-hello-kotlin)
* [编译和测试输出](#compiling-and-examining-output)
* [运行应用程序](#running-the-application)


## 获取编译器

Kotlin/Native 是可以运行在 macOS ， Linux 和 Windows 上的。 由于我们是工作在不同的操作系统上，因此需要正确地下载
对应的编译器。 虽然跨平台交叉编译是可行的（例如使用一个系统编译成另一个系统的可执行文件），在第一次的教程当中
我们将会编译出与当前运行的操作系统一致的结果。本例中使用的将会是 macOS 。

我们可以从 [GitHub releases page](https://github.com/JetBrains/kotlin-native/releases) 获取到最新版的编译器。

一旦下载完成，我们可以把它解压到任意文件夹中，例如放在 ~/kotlin-native 这个路径里。为了方便起见， 把 `bin` 文件夹放到系统的环境变量中是非常有用的， 这样我们就可以在任意地方调用 
编译器。 （如果我们把文件解压到 ~/kotlin-native ，那么路径应该是 ~/kotlin-native/bin ）

虽然编译器的输出不需要任何依赖，但是编译器本身是需要 Java 8 的，这也就意味着操作系统中应该要有对应的环境。如果我们使用的是 macOS ，那么我们就需要通过安装 Xcode 来安装 macOS SDK 。

## 创建 Hello Kotlin

我们的第一个应用程序将会是简单的输出一些文本。本例中，这些文本将会是“Hello Kotlin/Native”
 
我们可以打开自己喜欢的 IDE 或者编辑器，编写如下代码并把文件命名为： `hello.kt` 

<div class="sample" markdown="1" theme="idea">
```kotlin
fun main(args: Array<String>) {
    println("Hello Kotlin/Native!")
}
```
</div>

## 编译和测试输出

Kotlin 编译器使用的是一种叫 [LLVM](https://en.wikipedia.org/wiki/LLVM) 的技术来支持多个平台。 LLVM 需要中介码（或者被称为 IR ）作为输入。 IR 
是一种字节码的代表，该文件是比特流文件格式。

![Compiler Diagram]({{ url_for('tutorial_img', filename='native/basic-kotlin-native/llvm-diagram.png')}})


我们现在需要使用之前下载好的编辑器来进行编译工作。如果我们正确的把 `bin` 文件夹
添加到环境变量中， 那么我们应该可以直接使用如下命令来调用编译器：

```bash
kotlinc-native hello.kt
```

这是告诉编译器来编译 `hello.kt` 文件中的源码。

编译器第一次运行的时候会下载一些相关的依赖，因此第一次编译的时候会花费较长的时间。如果所有的步骤都正确执行了，那么输出的文件
应该会是 `program.kexe` 。

该文件是针对我们目标平台所实际生成的二进制文件。 编译器为我们提供了一系列的选项，其中一个
是用来定义输出的文件名。为了实现这个效果，我们可以使用 -output (or -o) 选项，如下：

```bash
kotlinc-native -o first hello.kt
```

这样生成的文件就应该是： `first.kexe` 。

后缀名是无法进行修改的，因为这取决于目标平台，但是我们仍然可以使用
常用的系统命令来把这些可执行文件重命名为任何喜欢的名字。

## 运行应用程序

为了运行程序，我们可以这样进行调用：

```bash
./program.kexe
```

有个非常重要的事情需要明白，那就是这是一个原生的应用程序，不需要额外的运行环境或者虚拟机来支持，输出的内容如下：

```bash
Hello Kotlin/Native!
```
