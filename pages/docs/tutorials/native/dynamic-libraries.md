---
type: tutorial
layout: tutorial
title:  "Kotlin/Native 开发动态库"
description: "将 Kotlin/Native 编译为动态库"
authors: Eugene Petrenko，乔禹昂（翻译）
date: 2019-04-15
showAuthorInfo: true
issue: EVAN-5371
---

在本教程中，我们将看到如何使用位于已经存在的<!--
-->原生应用程序或库中的代码。为此，我们需要将
Kotlin 代码编译为动态库，例如 `.so`、`.dylib` 以及 `.dll`。

Kotlin/Native 也可以与 Apple 技术紧密集成。
这篇 [Kotlin/Native 开发 Apple Framework](apple-framework.html)
教程解释了如何将代码编译成 Swift 与 Objective-C framework。

在这篇教程中，我们将：
 - [将 Kotlin 代码编译为动态库](#创建-kotlin-库)
 - [Examine generated C headers](#generated-headers-file)
 - [在 C 中调用 Kotlin 动态库](#using-generated-headers-from-c)
 - 将示例代码编译并运行于 [Linux 与 Mac](#compiling-and-running-the-example-on-linux-and-macos)
   以及 [Windows](#compiling-and-running-the-example-on-windows)
  
## 创建 Kotlin 库

Kotlin/Native 编译器可以将我们的 Kotlin
代码编译为一个动态库。
动态库常常带有一个头文件，即 `.h` 文件，
我们将通过它来调用编译后的 C 代码。

理解这些技术的最佳方法是尝试它们。
让我们创建第一个小型的 Kotlin 库，并在 C 程序中使用它。

我们可以先在 Kotlin 中创建一个库文件，然后将其保存为 `hello.kt`：

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
package example

object Object {
  val field = "A"
}

class Clazz {
  fun memberFunction(p: Int): ULong = 42UL
}

fun forIntegers(b: Byte, s: Short, i: UInt, l: Long) { }
fun forFloats(f: Float, d: Double) { }

fun strings(str: String) : String? {
  return "That is '$str' from C"
}

val globalString = "A global String"
```
</div>

[[include pages-includes/docs/tutorials/native/lets-create-gradle-build.md]]
[[include pages-includes/docs/tutorials/native/dynamic-library-code.md]]

这个已经准备好的工程源文件可以从这里直接下载：
[[include pages-includes/docs/tutorials/native/dynamic-library-link.md]]

让我们移除工程目录下的源文件移动到 `src/nativeMain/kotlin`
文件夹下。当使用 [kotlin 多平台](/docs/reference/building-mpp-with-gradle.html)<!--
-->插件的时候，对于源文件的位置，
这就是默认路径。我们使用以下代码块来指导和配置工程<!--
-->为我们生成动态或共享库：

<div class="multi-language-sample" data-os="macos">
<div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>

```kotlin
binaries {
  sharedLib {
    baseName = "native"
  }  
}
```
</div>
</div>
<div class="multi-language-sample" data-os="linux">
<div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>

```kotlin
binaries {
  sharedLib {
    baseName = "native"
  }  
}
```
</div>
</div>
<div class="multi-language-sample" data-os="windows">
<div class="sample" markdown="1" theme="idea" mode="kotlin" data-highlight-only>

```kotlin
binaries {
  sharedLib {
    baseName = "libnative"
  }  
}
```
</div>
</div>

`libnative` 被用作库名，即生成的<!--
-->头文件名前缀。 它也是前缀中的所有声明的前缀<!--
-->头文件。

现在我们已经准备好<!--
-->[在 IntelliJ IDEA 中打开这个工程](basic-kotlin-native-app.html#open-in-ide)<!--
-->并且可以看到如何修正这个示例工程。在我们这样做的时候，
我们将会研究 C 函数如何映射为 Kotlin/Native 声明。

让我们运行这个 `linkNative` Gradle 任务来<!--
-->[在 IDE 中](basic-kotlin-native-app.html#run-in-ide)构建该库。
或者运行下面这行控制台命令：
[[include pages-includes/docs/tutorials/native/linkNative.md]]

构建将会在 `build/bin/native/debugShared` 文件夹下生成
以下文件，并取决于目标操作系统：
- macOS: `libnative_api.h` 与 `libnative.dylib`
- Linux: `libnative_api.h` 与 `libnative.so`
- Windows: `libnative_api.h`、`libnative_symbols.def` 以及 `libnative.dll`

相似的规则被 Kotlin/Native 编译器用于在<!--
-->所有的平台上生成 `.h` 文件。  
来看看我们的 Kotlin 库的 C 语言 API。

## 生成头文件

在 `libnative_api.h` 中，我们将发现如下代码。
我们将探讨其中部分代码，以便更容易理解。

注意，Kotlin/Native 的外部符号如有变更，将不会另外通知。

第一部分包含了标准的 C/C++ 头文件的首尾：

<div class="sample" markdown="1" mode="C" theme="idea" data-highlight-only="1" auto-indent="false">

```c
#ifndef KONAN_DEMO_H
#define KONAN_DEMO_H
#ifdef __cplusplus
extern "C" {
#endif

/// 生成的代码的其余部分在这里

#ifdef __cplusplus
}  /* extern "C" */
#endif
#endif  /* KONAN_DEMO_H */
```
</div>

在 `libnative_api.h` 中的上述内容之后，我们就拥有了一个声明通用类型定义的块：

<div class="sample" markdown="1" mode="C" theme="idea" data-highlight-only="1" auto-indent="false">

```c
#ifdef __cplusplus
typedef bool            libnative_KBoolean;
#else
typedef _Bool           libnative_KBoolean;
#endif
typedef unsigned short     libnative_KChar;
typedef signed char        libnative_KByte;
typedef short              libnative_KShort;
typedef int                libnative_KInt;
typedef long long          libnative_KLong;
typedef unsigned char      libnative_KUByte;
typedef unsigned short     libnative_KUShort;
typedef unsigned int       libnative_KUInt;
typedef unsigned long long libnative_KULong;
typedef float              libnative_KFloat;
typedef double             libnative_KDouble;
typedef void*              libnative_KNativePtr;
``` 
</div>

Kotlin 在被创建的 `libnative_api.h` 文件中为<!--
-->所有的声明都添加了 `libnative_` 前缀。让我们以<!--
-->更容易阅读的方式来查看类型的映射：


|Kotlin Define          | C Type               |
|-----------------------|----------------------|
|`libnative_KBoolean`   | `bool` 或 `_Bool`    |
|`libnative_KChar`      |  `unsigned short`    |
|`libnative_KByte`      |  `signed char`       |
|`libnative_KShort`     |  `short`             |
|`libnative_KInt`       |  `int`               |
|`libnative_KLong`      |  `long long`         |
|`libnative_KUByte`     |  `unsigned char`     |
|`libnative_KUShort`    |  `unsigned short`    |
|`libnative_KUInt`      |  `unsigned int`      |
|`libnative_KULong`     |  `unsigned long long`|
|`libnative_KFloat`     |  `float`             |
|`libnative_KDouble`    |  `double`            |
|`libnative_KNativePtr` |  `void*`             |
{:.zebra}

这个定义部分展示了如何将 Kotlin 的原始类型映射为 C 的原始类型。 
我们在这篇[从 C 语言中映射原始类型](mapping-primitive-data-types-from-c.html)教程中讨论了反向映射。

`libnative_api.h` 文件的下一个部分包含了在该库中<!--
-->使用的类型的定义：

<div class="sample" markdown="1" mode="C" theme="idea" data-highlight-only="1" auto-indent="false">

```c
struct libnative_KType;
typedef struct libnative_KType libnative_KType;

typedef struct {
  libnative_KNativePtr pinned;
} libnative_kref_example_Object;

typedef struct {
  libnative_KNativePtr pinned;
} libnative_kref_example_Clazz;
```
</div>

`typedef struct { .. } TYPE_NAME` 语法在 C 语言中被用于声明一个结构体。
[这个问题](https://stackoverflow.com/questions/1675351/typedef-struct-vs-struct-definitions)
提供了对该模式的更多解释。

我们可以看到其中定义了 Kotlin 的 `Object` 类被映射到了
`libnative_kref_example_Object`，并且 `Clazz` 被映射到了 `libnative_kref_example_Clazz`。
这两个结构体都没有包含任何东西，但是 `pinned` 字段是一个指针，该字段类型
`libnative_KNativePtr` 被定义为 `void*`。

C 语言中没有支持命名空间，所以 Kotlin/Native 编译器生成了<!--
-->长名称，以避免与现有原生工程中的其他符号发生任何可能的冲突。

定义的很大一部分位于 `libnative_api.h` 文件。
它包含了我们的 Kotlin/Native 库世界的定义：


<div class="sample" markdown="1" mode="C" theme="idea" data-highlight-only="1" auto-indent="false">

```c
typedef struct {
  /* Service functions. */
  void (*DisposeStablePointer)(libnative_KNativePtr ptr);
  void (*DisposeString)(const char* string);
  libnative_KBoolean (*IsInstance)(libnative_KNativePtr ref, const libnative_KType* type);

  /* User functions. */
  struct {
    struct {
      struct {
        void (*forIntegers)(libnative_KByte b, libnative_KShort s, libnative_KUInt i, libnative_KLong l);
        void (*forFloats)(libnative_KFloat f, libnative_KDouble d);
        const char* (*strings)(const char* str);
        const char* (*get_globalString)();
        struct {
          libnative_KType* (*_type)(void);
          libnative_kref_example_Object (*_instance)();
          const char* (*get_field)(libnative_kref_example_Object thiz);
        } Object;
        struct {
          libnative_KType* (*_type)(void);
          libnative_kref_example_Clazz (*Clazz)();
          libnative_KULong (*memberFunction)(libnative_kref_example_Clazz thiz, libnative_KInt p);
        } Clazz;
      } example;
    } root;
  } kotlin;
} libnative_ExportedSymbols;
```
</div>

这段代码使用了匿名的结构体定义。代码 `struct { .. } foo`
在结构体外部声明了一个<!--
-->匿名结构体类型，这个类型没有名字。

C 语言同样也不支持对象。人们使用函数指针来模仿<!--
-->对象语义。一个函数指针被声明在 `RETURN_TYPE (* FIELD_NAME)(PARAMETERS)` 后面。
它的阅读性很差，但我们应该能够从上面的结构中看到函数指针字段。

### 运行时函数

阅读下面的代码。我们拥有这个 `libnative_ExportedSymbols` 结构体，它定义了所有
Kotlin/Native 中的以及我们的库提供给我们的函数。它使用<!--
-->嵌套匿名结构，以模仿包。`libnative_` 前缀来源于<!--
-->库的名字。

`libnative_ExportedSymbols` 结构体包含了几个辅助函数：

<div class="sample" markdown="1" mode="C" theme="idea" data-highlight-only="1" auto-indent="false">

```c
void (*DisposeStablePointer)(libnative_KNativePtr ptr);
void (*DisposeString)(const char* string);
libnative_KBoolean (*IsInstance)(libnative_KNativePtr ref, const libnative_KType* type);
```
</div>

这些函数被用于处理 Kotlin/Native 的对象。调用
`DisposeStablePointer` 来释放一个 Kotlin 对象，而 `DisposeString` 被用于释放一个 Kotlin String, 
which has the `char*` type in C. `IsInstance` 函数可以被用于检查一个
Kotlin 类型或者一个 `libnative_KNativePtr` 是否是某个类型的实例。实际的<!--
-->生成操作取决于实际的使用情况。
 
Kotlin/Native 拥有垃圾回收机制，但是它不能帮助我们处理<!--
-->来源于 C 的 Kotlin 对象。Kotlin/Native 可以与 Objective-C 以及
Swift 进行互操作，并且结合了它们的引用计数。 
这篇 [Objective-C 互操作](/docs/reference/native/objc_interop.html)
包含了更多关于此内容的细节。当然，也可以参考<!--
-->这篇 [Kotlin/Native 开发 Apple Framework](apple-framework.html) 文档。

### Our Library Functions

Let's take a look at the `kotlin.root.example` field, it
mimics the package structure of our Kotlin code with a `kotlin.root.` prefix.

There is a `kotlin.root.example.Clazz` field that 
represents the `Clazz` from Kotlin. The `Clazz#memberFunction` is
accessible with the `memberFunction` field. The only difference is that 
the `memberFunction` accepts a `this` reference as the first parameter. 
The C language does not support objects, and this is the reason to pass a
`this` pointer explicitly.

There is a constructor in the `Clazz` field (aka `kotlin.root.example.Clazz.Clazz`),
which is the constructor function to create an instance of the `Clazz`.

Kotlin `object Object` is accessible as `kotlin.root.example.Object`. There is 
the `_instance` function to get the only instance of the object.

Properties are translated into functions. The `get_` and `set_` prefix
is used to name the getter and the setter functions respectively. For example, 
the read-only property `globalString` from Kotlin is turned 
into a `get_globalString` function in C. 

Global functions `forInts`, `forFloats`, or `strings` are turned into the functions pointers in
the `kotlin.root.example` anonymous struct.
 
### The Entry Point

We can see how the API is created. To start with, we need to initialize the 
`libnative_ExportedSymbols` structure. Let's take a look at the latest part 
of the `libnative_api.h` for this:

```c
extern libnative_ExportedSymbols* libnative_symbols(void);
```

The function `libnative_symbols` allows us to open the way from the native 
code to the Kotlin/Native library. This is the entry point we use. The 
library name is used as a prefix for the function name. 


Note, Kotlin/Native object references do not support multi-threaded access. 
Hosting the returned `libnative_ExportedSymbols*` pointer
per thread might be necessary.

## Using Generated Headers from C

The usage from C is straightforward and uncomplicated. We create a `main.c` file with the following 
code: 

<div class="sample" markdown="1" mode="C" theme="idea" data-highlight-only="1" auto-indent="false">

```c
#include "libnative_api.h"
#include "stdio.h"

int main(int argc, char** argv) {
  //obtain reference for calling Kotlin/Native functions
  libnative_ExportedSymbols* lib = libnative_symbols();

  lib->kotlin.root.example.forIntegers(1, 2, 3, 4);
  lib->kotlin.root.example.forFloats(1.0f, 2.0);

  //use C and Kotlin/Native strings
  const char* str = "Hello from Native!";
  const char* response = lib->kotlin.root.example.strings(str);
  printf("in: %s\nout:%s\n", str, response);
  lib->DisposeString(response);

  //create Kotlin object instance
  libnative_kref_example_Clazz newInstance = lib->kotlin.root.example.Clazz.Clazz();
  long x = lib->kotlin.root.example.Clazz.memberFunction(newInstance, 42);
  lib->DisposeStablePointer(newInstance.pinned);

  printf("DemoClazz returned %ld\n", x);

  return 0;
}
```
</div>

## Compiling and Running the Example on Linux and macOS

On macOS 10.13 with Xcode, we compile the C code and link it with the dynamic library
with the following command:
```bash
clang main.c libnative.dylib
```

On Linux we call a similar command: 
```bash
gcc main.c libnative.so
```

The compiler generates an executable called `a.out`. We need to run it to see in action the Kotlin code
being executed from C library. On Linux, we'll need to include `.` into the `LD_LIBRARY_PATH`
to let the application know to load the `libnative.so` library from the current folder.

## Compiling and Running the Example on Windows

To start with, we'll need a Microsoft Visual C++ compiler installed that supports a x64_64 
target. The easiest way to do this is to have a version of Microsoft Visual Studio installed on 
a Windows machine. 

We will be using the `x64 Native Tools Command Prompt <VERSION>` console. We'll see the 
shortcut to open the console in the start menu. It comes with a Microsoft Visual Studio
package.  

On Windows, Dynamic libraries are included either via a generated static library wrapper
or with manual code, which deals with the [LoadLibrary](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684175.aspx)
or similar Win32API functions. We will follow the first option and generate the static wrapper library
for the `libnative.dll` on our own.
 
We call `lib.exe` from the toolchain to generate the static library 
wrapper `libnative.lib` that automates the DLL usage from the code:
```bash
lib /def:libnative_symbols.def /out:libnative.lib
```

Now we are ready to compile our `main.c` into an executable. We include the generated `libnative.lib` into
the build command and start:
```bash
cl.exe main.c libnative.lib
```

The command produces the `main.exe` file, which we can run.
 

## Next Steps

Dynamic libraries are the main way to use Kotlin code from existing programs. 
We can use them to share our code with many platforms or languages, including JVM,
[Python](https://github.com/JetBrains/kotlin-native/blob/master/samples/python_extension/src/main/c/kotlin_bridge.c),
iOS, Android, and others.

Kotlin/Native also has tight integration with Objective-C and Swift.
It is covered in the [Kotlin/Native as an Apple Framework](apple-framework.html)
tutorial.
