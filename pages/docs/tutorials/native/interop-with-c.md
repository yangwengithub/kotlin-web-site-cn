---
type: tutorial
layout: tutorial
title: "与 C 语言库互操作"
description: "来看看如何与 C 语言库互操作"
authors: Hadi Hariri 
date: 2018-11-20
showAuthorInfo: false
---


当编写一个原生应用程序时，我们时常需要访问某些没有被包含在 Kotlin 标准库中的函数，<!--
-->例如发起 HTTP 请求，读写磁盘等等。

Kotlin/Native 给我们提供了操作 C 语言标准库的能力，这样就开放了存在于整个生态系统中<!--
-->几乎所有我们需要的功能。事实上，Kotlin/Native 已经预装了一套预制的[多平台库](https://github.com/JetBrains/kotlin-native/blob/master/PLATFORM_LIBS.md)<!--
-->来提供一些标准库不包含的通用功能。

然而在本教程中，我们将看到如何使用一些诸如 `libcurl` 这样的具体的库。我们将学到

* [创建 Kotlin 绑定器](#生成绑定)
* [使用生成的 Kotlin API](#使用生成的-kotlin-api)
* [在应用程序中连接库](#在应用程序中连接库)


## 生成绑定

互操作的理想方案是当我们在调用 C 函数的时候就像在调用 Kotlin 函数一样，也就是说，遵循相同的签名惯例。而这恰恰是
`cinterop` 工具提供给我们的功能。它需要一个 C 库并为它生成一个相应的 Kotlin 绑定，然后允许我们像<!--
-->使用 Kotlin 代码一样使用该库。

为了生成这些绑定，我们需要创建一个库定义 `.def` 文件，其中包含一些我们需要生成的头信息。在我们的案例中，我们想使用著名 `libcurl`
库来发起一些 HTTP 调用，所以我们将创建一个名为 `libcurl.def` 的文件，其中包含以下内容

<div class="sample" markdown="1" mode="c" theme="idea" data-highlight-only="1" auto-indent="false">

```c
headers = curl/curl.h
headerFilter = curl/*

compilerOpts.linux = -I/usr/include -I/usr/include/x86_64-linux-gnu
linkerOpts.osx = -L/opt/local/lib -L/usr/local/opt/curl/lib -lcurl
linkerOpts.linux = -L/usr/lib/x86_64-linux-gnu -lcurl
```
</div>

该文件中正在进行一些操作，让我们逐一查看它们。第一个条目是 `headers`，它是我们想要生成的头文件列表
Kotlin stub。我们可以在此条目中添加多个文件，在每个新行上使用 `\` 进行分隔。在我们的案例中我们仅仅想要 `curl.h`。我们引用的文件<!--
-->需要相对于定义文件所在的文件夹，或者在系统路径上可用（在我们的例子中是 `/usr/include/curl`）。

第二行是 `headerFilter`。这用于表示我们想要包含的内容。在 C 语言中，当一个文件使用 `#include` 指令引用另一个文件时，<!--
-->所有的头都将被应用。有时这也许是不需要的，我们可以使用这个参数，[使用 glob 模式](https://en.wikipedia.org/wiki/Glob_(programming))，来进行微调。<!--
-->注意，这个 `headerFilter` 是一个可选参数，并且主要仅在我们正在使用的库作为系统库安装时才使用，我们不想获取外部依赖项<!--
-->（比如系统 `stdint.h` 头）进入我们的互操作库。对于优化库大小和修复系统与 Kotlin/Native 提供的编译环境之间的潜在冲突可能都很重要。

接下来的几行是关于提供链接器和编译器选项，可以根据不同的目标平台而有所不同。在我们的例子中，我们定义平台为 macOS（`.osx` 后缀）和 Linux（`.linux` 后缀）。<!--
-->参数中没有后缀也是可行的（例如 `linkerOpts=`）并将适用于所有平台。 

惯例是每个库都有自己的定义文件，经常被命名为与库中的相同。关于所有 `cinterop`
的配置的更多信息，请查看[互操作文档](/docs/reference/native/c_interop.html)

一旦我们准备好了定义文件，我们就可以调用 `cinterop` 并让它为我们生成绑定

    cinterop -def libcurl.def -o build/c_interop/libcurl
    
输出应该是 Kotlin 编译的库（扩展 `.klib`）。我们也指示 `cinterop`
将内容输出到 `build/c_interop/libcurl` 文件夹，我们稍后将使用 `kotlinc-native` 作为输入。这个文件夹的创建非常重要，<!--
-->因为 `cinterop` 之前需要一个现有的文件夹。`c_interop` 是针对互操作库遵循的约定。

### 在 Windows 上 curl

你应该在 Windows 上使用 `curl` 库二进制文件来使示例工作。
你会在 Windows 从[源码](https://curl.haxx.se/download.html)构建 `curl`（你将需要 Visual Studio 或 Windows SDK 命令行工具），关于更多<!--
-->细节，请查看[相关博文](https://jonnyzzz.com/blog/2018/10/29/kn-libcurl-windows/)。<!--
-->或者，你也可以考虑使用 [MinGW/MSYS2](https://www.msys2.org/) `curl` 二进制文件。

你将需要提供 `curl` 源码路径给 `cinterop` 工具来调用： `-compilerOpts -I<curl include directory>` 以及一个链接器选项<!--
-->给 `kotlinc-native` 调用：`-linker-options "<curl binary directory>\libcurl.lib"`。<!--
-->注意。在 Windows 上，你总是使用 `.lib` 文件来链接，无论是使用*静态*还是*动态*库。对于*动态*链接，<!--
-->`.lib` 文件通常由工具链生成，并且它在内部使用 `.dll`。


## 使用生成的 Kotlin API

现在，我们有了一个库和相应的 Kotlin stub，我们可以在应用程序中使用它们。为了让事情简单一些，在本示例中我们将最简单的一个
`libcurl` 示例转换为 Kotlin。

有问题的代码来自[示例](https://curl.haxx.se/libcurl/c/simple.html)（为简洁起见删除了评论）

<div class="sample" markdown="1" theme="idea" mode="c">

```c
#include <stdio.h>
#include <curl/curl.h>
 
int main(void)
{
  CURL *curl;
  CURLcode res;
 
  curl = curl_easy_init();
  if(curl) {
    curl_easy_setopt(curl, CURLOPT_URL, "http://example.com");
    curl_easy_setopt(curl, CURLOPT_FOLLOWLOCATION, 1L);
 
    res = curl_easy_perform(curl);
    if(res != CURLE_OK)
      fprintf(stderr, "curl_easy_perform() failed: %s\n",
              curl_easy_strerror(res));
    curl_easy_cleanup(curl);
  }
  return 0;
}
```
</div>

第一件事情是我们将需要一个定义了 `main` 函数的 Kotlin 文件，并起名为 `Main.kt` 接下来将继续翻译每一行

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
import libcurl.*
import kotlinx.cinterop.*

fun main(args: Array<String>) {
    val curl = curl_easy_init()
    if (curl != null) {
        curl_easy_setopt(curl, CURLOPT_URL, "http://example.com")
        curl_easy_setopt(curl, CURLOPT_FOLLOWLOCATION, 1L)
        val res = curl_easy_perform(curl)
        if (res != CURLE_OK) {
            println("curl_easy_perform() failed ${curl_easy_strerror(res)?.toKString()}")
        }
        curl_easy_cleanup(curl)
    }
}
```
</div>

我们可以看到，我们已经消除了 Kotlin 版本中的显式变量声明，但是其它的一切都是 C 版本的逐字逐句翻译。我们可以调用<!--
-->所有 `libcurl` 库中的 API 的 Kotlin 版本。

注意，出于本教程的目的，我们对逐行进行了直译。显然，我们可以用更 Kotlin 的惯用方式来编写这个例子。

## 编译与链接库

下一步是编译我们的应用程序。我们已经在这篇[基本 Kotlin/Native 应用程序](basic-kotlin-native-app.html)教程中介绍了从命令行编译 Kotlin/Native 应用程序的基础知识。<!--
-->这个案例中的唯一不同之处是我们必须引入 `cinterop` 为我们生成的库。

```bash
kotlinc-native Main.kt -library build/c_interop/libcurl
```

我们可以看到作为 `library` 参数传递 `cinterop` 的输出路径。

如果编译期间没有错误，我们应该查看名为 `program.kexe` 的输出文件，执行时应该输出哪个
`http://example.com` 网站的内容

![Output]({{ url_for('tutorial_img', filename='native/cinterop/output.png')}})

我们看到实际输出的原因是因为调用 `curl_easy_perform` 将结果打印到标准输出。我们应该使用
`curl_easy_setopt` 隐藏它。 

有关使用 `libcurl` 的更完整示例，[libcurl 在 Kotlin/Native 项目中的示例](https://github.com/JetBrains/kotlin-native/tree/master/samples/libcurl)展示了如何将代码抽象为 Kotlin
类以及显示标题。它还演示了如何通过将它们组合到 shell 脚本或 Gradle 构建脚本中来使步骤更简洁一些。我们将在后续教程中介绍这些主题的更多细节。

