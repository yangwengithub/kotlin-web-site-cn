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
来看看我们的 Kotlin 库的 C API。

## 生成头文件

In the `libnative_api.h`, we'll find the following code. 
We will discuss the code in parts to make it easier to understand.

Note, the way Kotlin/Native exports symbols is subject to change without notice.

The very first part contains the standard C/C++ header and footer:

<div class="sample" markdown="1" mode="C" theme="idea" data-highlight-only="1" auto-indent="false">

```c
#ifndef KONAN_DEMO_H
#define KONAN_DEMO_H
#ifdef __cplusplus
extern "C" {
#endif

/// THE REST OF THE GENERATED CODE GOES HERE

#ifdef __cplusplus
}  /* extern "C" */
#endif
#endif  /* KONAN_DEMO_H */
```
</div>

After the rituals in the `libnative_api.h`, we have a block with the common type definitions:

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

Kotlin uses the `libnative_` prefix for all declarations
in the created `libnative_api.h` file. Let's present
the mapping of the types in a more readable way:


|Kotlin Define          | C Type               |
|-----------------------|----------------------|
|`libnative_KBoolean`   | `bool` or `_Bool`    |
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

The definitions part shows how Kotlin primitive types map into C primitive types. 
We discussed reverse mapping in the [Mapping Primitive Data Types from C](mapping-primitive-data-types-from-c.html) tutorial.

The next part of the `libnative_api.h` file contains definitions of the types
that are used in the library:

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

The `typedef struct { .. } TYPE_NAME` syntax is used in C language to declare a structure. 
[The thread](https://stackoverflow.com/questions/1675351/typedef-struct-vs-struct-definitions)
provides more explanations of that pattern.

We see from these definitions that the Kotlin object `Object` is mapped into
`libnative_kref_example_Object`, and `Clazz` is mapped into `libnative_kref_example_Clazz`.
Both structs contain nothing but the `pinned` field with a pointer, the field type 
`libnative_KNativePtr` is defined as `void*` above. 

There is no namespaces support in C, so the Kotlin/Native compiler generates 
long names to avoid any possible clashes with other symbols in the existing native project.

A significant part of the definitions goes in the `libnative_api.h` file.
It includes the definition of our Kotlin/Native library world:


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

The code uses anonymous structure declarations. The code `struct { .. } foo`
declares a field in the outer struct of that 
anonymous structure type, the type with no name. 

C does not support objects either. People use function pointers to mimic 
object semantics. A function pointer is declared as follows `RETURN_TYPE (* FIELD_NAME)(PARAMETERS)`.
It is tricky to read, but we should be able to see function pointer fields in the structures above. 

### Runtime Functions

The code reads as follows. We have the `libnative_ExportedSymbols` structure, which defines
all the functions that Kotlin/Native and our library provides us. It uses 
nested anonymous structures heavily to mimic packages. The `libnative_` prefix comes from the
library name.

The `libnative_ExportedSymbols` structure contains several helper functions:

<div class="sample" markdown="1" mode="C" theme="idea" data-highlight-only="1" auto-indent="false">

```c
void (*DisposeStablePointer)(libnative_KNativePtr ptr);
void (*DisposeString)(const char* string);
libnative_KBoolean (*IsInstance)(libnative_KNativePtr ref, const libnative_KType* type);
```
</div>

These functions deal with Kotlin/Native objects. Call the 
`DisposeStablePointer` to release a Kotlin object and `DisposeString` to release a Kotlin String, 
which has the `char*` type in C. It is possible to use the `IsInstance` function to check if a
Kotlin type or a `libnative_KNativePtr` is an instance of another type. The actual set of
operations generated depends on the actual usages.
 
Kotlin/Native has garbage collection, but it does not help us deal
with Kotlin objects from the C language. Kotlin/Native has interop with Objective-C and 
Swift and integrates with their reference counters. 
The [Objective-C Interop](/docs/reference/native/objc_interop.html)
documentation article contains more details on it. Also,
there is the tutorial [Kotlin/Native as an Apple Framework](apple-framework.html).

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
