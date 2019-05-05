---
type: tutorial
layout: tutorial
title:  "Using C Interop and libcurl for an App"
description: "Using C library from Kotlin/Native"
authors: Hadi Hariri 
date: 2019-04-15
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

An ideal scenario for interop is to call C functions as if we were calling Kotlin functions, that is, following the same signature and conventions. This is precisely what the
`cinterop` tool provides us with. It takes a C library and generates the corresponding Kotlin bindings for it, which then allows us
to use the library as if it were Kotlin code.

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

A few things are going on in this file, let's go through them one by one. The first entry is `headers` which is the list of header files that we want to generate
Kotlin stubs for. We can add multiple files to this entry, separating each one with a `\` on a new line. In our case we only want `curl.h`. The files we are referencing
need to be relative to the folder where the definition file is, or be available on the system path (in our case it would be `/usr/include/curl`).

The second line is the `headerFilter`. This is used to denote what exactly we want included. In C, when one file references another file with the `#include` directive,
all the headers are also included. Sometimes this may not be needed, and we can use this parameter, [using glob patterns](https://en.wikipedia.org/wiki/Glob_(programming)), to fine tune things.
Note, that `headerFilter` is an optional argument and mostly only used when the library we're using is being installed as a system library, and we do not want to fetch external dependencies
(such as system `stdint.h` header) into our interop library. It may be important for both optimizing the library size and fixing potential conflicts between the system and the Kotlin/Native provided compilation environment.

The next lines are about providing linker and compiler options, which can vary depending on different target platforms. In our case, we are defining it for macOS (the `.osx` suffix) and Linux (the `.linux` suffix).
Parameters without a suffix is also possible (e.g. `linkerOpts=`) and will be applied to all platforms.

惯例是每个库都有自己的定义文件，经常被命名为与库中的相同。关于所有 `cinterop`
的配置的更多信息，请查看[互操作文档](/docs/reference/native/c_interop.html)

Once we have the definition file ready, we can
create project files and open the project in an IDE.

[[include pages-includes/docs/tutorials/native/mapping-primitive-data-types-gradle.md]]


### 在 Windows 上 curl

You should have the `curl` library binaries on Windows to make the sample work.
You may build `curl` from [sources](https://curl.haxx.se/download.html) on Windows (you'll need Visual Studio or Windows SDK Commandline tools), for more
details, see the [related blog post](https://jonnyzzz.com/blog/2018/10/29/kn-libcurl-windows/).
Alternatively, you may also want to consider a [MinGW/MSYS2](https://www.msys2.org/) `curl` binary.

## 使用生成的 Kotlin API

Now we have our library and Kotlin stubs, we can consume them from our application. To keep things simple, in this tutorial we're going to convert one of the simplest
`libcurl` examples over to Kotlin.

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

第一件事情是我们将需要一个定义了 `main` 函数的 Kotlin 文件，并起名为 `src/nativeMain/kotlin/hello.kt` 接下来将继续翻译每一行

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
import interop.*
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

As we can see, we've eliminated the explicit variable declarations in the Kotlin version, but everything else is pretty much verbatim to the C version. All the calls we'd
expect in the `libcurl` library are available in their Kotlin equivalent.

注意，出于本教程的目的，我们对逐行进行了直译。显然，我们可以用更 Kotlin 的惯用方式来编写这个例子。

## 编译与链接库

The next step is to compile our application. We already covered the basics of compiling a Kotlin/Native application from the command line in the [A Basic Kotlin/Native application](basic-kotlin-native-app.html) tutorial.
The only difference in this case is that the `cinterop` generated part is implicitly included into the build:
Let's call the following command:
[[include pages-includes/docs/tutorials/native/runDebugExecutableNative.md]]

If there are no errors during compilation, we should see the result of the execution
of our program, which on execution should output
the contents of the site `http://example.com`

![Output]({{ url_for('tutorial_img', filename='native/cinterop/output.png')}})

我们看到实际输出的原因是因为调用 `curl_easy_perform` 将结果打印到标准输出。我们应该使用
`curl_easy_setopt` 隐藏它。

有关使用 `libcurl` 的更完整示例，[libcurl 在 Kotlin/Native 项目中的示例](https://github.com/JetBrains/kotlin-native/tree/master/samples/libcurl)展示了如何将代码抽象为 Kotlin
类以及显示标题。它还演示了如何通过将它们组合到 shell 脚本或 Gradle 构建脚本中来使步骤更简洁一些。我们将在后续教程中介绍这些主题的更多细节。

