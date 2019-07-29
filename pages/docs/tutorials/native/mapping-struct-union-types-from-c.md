---
type: tutorial
layout: tutorial
title:  "映射来自 C 语言的结构与联合类型"
description: "如何在 Kotlin/Native 中观察 C 语言的结构与联合类型"
authors: Eugene Petrenko，乔禹昂（翻译）
date: 2019-04-15
showAuthorInfo: true
issue: EVAN-5343
---

这是本系列的第二篇教程。本系列的第一篇教程是<!--
-->[映射来自 C 语言的原始数据类型](mapping-primitive-data-types-from-c.html)。
There are also the [Mapping Struct and Union Types from C](mapping-struct-union-types-from-c.html) and 
[映射来自 C 语言的字符串](mapping-strings-from-c.html)。

在本教程中我们将学习到
- [如何映射结构与联合类型](#映射-c-语言的结构与联合类型)
- [在 Kotlin 中如何使用结构与联合类型](#在-kotlin-中使用结构与联合类型)

我们需要在自己的机器上已经安装了 Kotlin 编译器。
这篇<!--
-->[基本 Kotlin 应用程序](basic-kotlin-native-app.html#obtaining-the-compiler)<!--
-->教程涵盖了这一步骤。
我们假设，我们拥有一个 `kotlinc-native`、`cinterop` 以及 `klib` 命令行工具都已经准备好控制台。

## 映射 C 语言的结构与联合类型

理解在 Kotlin 与 C 之间进行映射的最好方式是尝试编写一个小型<!--
-->示例。我们将在 C 语言中声明一个结构体与一个联合体，并以此来观察如何将它们映射到 Kotlin 中。

Kotlin/Native 附带 `cinterop` 工具，该工具可以生成 C 语言与 Kotlin 之间的绑定。
它使用一个 `.def` 文件指定一个 C 库来导入。更多的细节将在<!--
-->[与 C 库互操作](/docs/reference/native/c_interop.html)教程中讨论。
 
在[之前的教程](mapping-primitive-data-types-from-c.html)中我们创建了一个 `lib.h` 文件。这次，
在 `---` 分割行之后，我们将直接将这些声明导入到 `interop.def` 文件：

<div class="sample" markdown="1" mode="c" theme="idea" data-highlight-only="1" auto-indent="false">

```c

---

typedef struct {
  int a;
  double b;
} MyStruct;

void struct_by_value(MyStruct s) {}
void struct_by_pointer(MyStruct* s) {}

typedef union {
  int a;
  MyStruct b;
  float c;
} MyUnion;

void union_by_value(MyUnion u) {}
void union_by_pointer(MyUnion* u) {}

``` 
</div>

该 `interop.def` 文件足够用来编译并运行应用程序，或在 IDE 中打开它。
现在是时候创建工程文件，并在
[IntelliJ IDEA](https://jetbrains.com/idea) 中打开这个工程，然后运行它。

## 为 C 库检查生成的 Kotlin API

[[include pages-includes/docs/tutorials/native/mapping-primitive-data-types-gradle.md]]

让我们使用下面的内容创建一个 `src/nativeMain/kotlin/hello.kt` 存根文件，
以用来观察我们的 C 声明是如何在 Kotlin 中可见的：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
import interop.*

fun main() {
  println("Hello Kotlin/Native!")
  
  struct_by_value(/* fix me*/)
  struct_by_pointer(/* fix me*/)
  union_by_value(/* fix me*/)
  union_by_pointer(/* fix me*/)
}
```
</div>

现在我们已经准备好<!--
-->[在 IntelliJ IDEA 中打开这个工程](basic-kotlin-native-app.html#open-in-ide)<!--
-->并且看看如何修正这个示例工程。当我们做了这些之后，
我们将观察到 C 的原始类型已经被映射到了 Kotlin/Native。

## Kotlin 中的原始类型

通过 IntelliJ IDEA 的 _Goto Declaration_ 或
编译器错误的帮助，我们会看到如下的为 C  函数、`struct` 以及 `union` 生成的 API：

<div class="sample" markdown="1" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
fun struct_by_value(s: CValue<MyStruct>)
fun struct_by_pointer(s: CValuesRef<MyStruct>?)

fun union_by_value(u: CValue<MyUnion>)
fun union_by_pointer(u: CValuesRef<MyUnion>?)

class MyStruct constructor(rawPtr: NativePtr /* = NativePtr */) : CStructVar {
    var a: Int
    var b: Double
    companion object : CStructVar.Type
}

class MyUnion constructor(rawPtr: NativePtr /* = NativePtr */) : CStructVar {
    var a: Int
    val b: MyStruct
    var c: Float
    companion object : CStructVar.Type
}
```
</div>

我们看到 `cinterop` 为我们的 `struct` 与 `union` 类型生成了包装类型。
为在 C 中声明的 `MyStruct` 与 `MyUnion` 类型，我们分别为其<!--
-->生成了 Kotlin 类 `MyStruct` 与 `MyUnion`。
该包装器继承自 `CStructVar` 基类并将所有的字段声明为了 Kotlin 属性。
它使用 `CValue<T>` 来表示一个值类型的结构体参数并使用 `CValuesRef<T>?`
来表示传递一个结构体或共用体的指针。

从技术上讲，在 Kotlin 看来 `struct` 与 `union` 类型之间<!--
-->没有区别。我们应该注意，Kotlin 中 `MyUnion` 类的 `a`、`b` 以及 `c` 属性使用了<!--
-->相同的位置来进行读写值的操作，就像 C 语言中的 `union` 一样。

更多细节与高级用例将在 
[C 互操作文档](https://github.com/JetBrains/kotlin-native/blob/master/INTEROP.md#passing-and-receiving-structs-by-value)中介绍

## 在 Kotlin 中使用结构与联合类型

It is easy to use the generated wrapper classes for C `struct` and `union` types from Kotlin. Thanks to the generated
properties, it feels natural to use them in Kotlin code. The only question, so far, is how do we create a new instance on those
classes. As we see from the declarations of `MyStruct` and `MyUnion`, their constructors require a `NativePtr`.
Of course, we are not willing to deal with pointers manually. Instead, we can use Kotlin API to have those 
objects instantiated for us. 

Let's take a look at the generated functions that take our `MyStruct` and `MyUnion` as parameters. We see that 
by-value parameters are represented as `kotlinx.cinterop.CValue<T>`. And for typed pointer parameters we 
see `kotlinx.cinterop.CValuesRef<T>`.
Kotlin provides us with an API to deal with both types easily, let's try it and see.

### Creating a `CValue<T>`

`CValue<T>` type is used to pass by-value parameters to a C function call.
We use `cValue` function to create `CValue<T>` object instance. The function requires a
[lambda function with a receiver](../../reference/lambdas.html#带有接收者的函数字面值)
to initialize the underlying C type in-place. The function is declared as follows:
<div class="sample" markdown="1" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
fun <reified T : CStructVar> cValue(initialize: T.() -> Unit): CValue<T>
```
</div>

Now it is time to see how to use `cValue` and pass by-value parameters:
<div class="sample" markdown="1" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
fun callValue() {

  val cStruct = cValue<MyStruct> {
    a = 42
    b = 3.14
  }
  struct_by_value(cStruct)

  val cUnion = cValue<MyUnion> {
    b.a = 5
    b.b = 2.7182
  }

  union_by_value(cUnion)
}
```
</div>

### Creating Struct and Union as `CValuesRef<T>`

`CValuesRef<T>` type is used in Kotlin to pass a typed pointer parameter of a C 
function. First, we need an instance of 
`MyStruct` and `MyUnion` classes. This time we create them directly in the native memory. 
Let's use the    
<div class="sample" markdown="1" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
fun <reified T : kotlinx.cinterop.CVariable> alloc(): T   
```
</div>

extension function on `kotlinx.cinterop.NativePlacement`
type for this.

`NativePlacement` represents native memory with functions similar to `malloc` and `free`. 
There are several implementations of `NativePlacement`. The global one is called with `kotlinx.cinterop.nativeHeap`
and don't forget to call the `nativeHeap.free(..)` function to free the memory after use.
 
Another option is to use the
<div class="sample" markdown="1" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
fun <R> memScoped(block: kotlinx.cinterop.MemScope.() -> R): R    
```
</div>

function. It creates a short-lived memory allocation scope,
and all allocations will be cleaned up automatically at the end of the `block`.

Our code to call functions with pointers will look like this:
<div class="sample" markdown="1" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
fun callRef() {
  memScoped {
    val cStruct = alloc<MyStruct>()
    cStruct.a = 42
    cStruct.b = 3.14

    struct_by_pointer(cStruct.ptr)


    val cUnion = alloc<MyUnion>()
    cUnion.b.a = 5
    cUnion.b.b = 2.7182

    union_by_pointer(cUnion.ptr)
  }
}

```
</div>

Note, we use the extension property `ptr` which comes from a `memScoped` lambda receiver type, 
to turn `MyStruct` and `MyUnion` instances into native pointers.

The `MyStruct` and `MyUnion` classes have the pointer to the native memory underneath. The memory will be released
when a `memScoped` function ends, which is equal to the end of its `block`. Be careful to make sure that a
pointer is not used outside of the `memScoped` call. We may use `Arena()` or `nativeHeap` for pointers that 
should be available longer, or are cached inside a C library.  

### Conversion between `CValue<T>` and `CValuesRef<T>`

Of course, there are use cases, where we need to pass a struct as a value to one call, and then, to 
pass the same struct as a reference to another call. This is possible in Kotlin/Native too. A 
`NativePlacement` will be needed here. 

Let's see now `CValue<T>` is turned to a pointer first:
<div class="sample" markdown="1" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
fun callMix_ref() {
  val cStruct = cValue<MyStruct> {
    a = 42
    b = 3.14
  }
  
  memScoped { 
    struct_by_pointer(cStruct.ptr)
  }
}
```  
</div>

We use the extension property `ptr` which comes from `memScoped` lambda receiver type 
to turn `MyStruct` and `MyUnion` instances into native pointers. Those pointers are only valid
inside the `memScoped` block.

For the opposite conversion, to turn a pointer into a by-value variable, 
we call the `readValue()` extension function:

<div class="sample" markdown="1" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
fun callMix_value() {
  memScoped {
    val cStruct = alloc<MyStruct>()
    cStruct.a = 42
    cStruct.b = 3.14

    struct_by_value(cStruct.readValue())
  }
}
```
</div>

## Running the Code

Now we have learned how to use C declarations in our code, we are ready to try
it out on a real example. Let's fix our code and see how it runs by calling the 
`runDebugExecutableNative` Gradle task [in the IDE](basic-kotlin-native-app.html#run-in-ide)
or by using the following console command:
[[include pages-includes/docs/tutorials/native/runDebugExecutableNative.md]]


The final code in the `hello.kt` file may look like this:
 
<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
import interop.*
import kotlinx.cinterop.alloc
import kotlinx.cinterop.cValue
import kotlinx.cinterop.memScoped
import kotlinx.cinterop.ptr
import kotlinx.cinterop.readValue

fun main() {
  println("Hello Kotlin/Native!")

  val cUnion = cValue<MyUnion> {
    b.a = 5
    b.b = 2.7182
  }

  memScoped {
    union_by_value(cUnion)
    union_by_pointer(cUnion.ptr)
  }

  memScoped {
    val cStruct = alloc<MyStruct> {
      a = 42
      b = 3.14
    }

    struct_by_value(cStruct.readValue())
    struct_by_pointer(cStruct.ptr)
  }
}
```
</div>


## Next Steps

Join us to continue exploring the C language types and their representation in Kotlin/Native in the related tutorials:
- [Mapping Primitive Data Types from C](mapping-primitive-data-types-from-c.html)
- [Mapping Function Pointers from C](mapping-function-pointers-from-c.html)
- [Mapping Strings from C](mapping-strings-from-c.html)

The [C Interop documentation](https://github.com/JetBrains/kotlin-native/blob/master/INTEROP.md)
documentation covers more advanced scenarios of the interop

