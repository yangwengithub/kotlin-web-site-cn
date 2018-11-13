---
type: doc
layout: reference
title: "Kotlin 1.3 的新特性"
---

# Kotlin 1.3 的新特性

## 协程正式发布

历经了漫长而充足的的测试，协程终于正式发布了！这意味着自 Kotlin 1.3 起，协程的语言支持与 API 已[完全稳定](evolution/components-stability.html)。参见新版[协程概述](coroutines-overview.html)。

Kotlin 1.3 引入了挂起函数的可调用引用以及在反射 API 中对协程的支持。

## Kotlin/Native

Kotlin 1.3 继续改进与完善原生平台。详情请参见 [Kotlin/Native 概述](native-overview.html)。

## 多平台项目

在 1.3 中，我们完全修改了多平台项目的模型，以提高表现力与灵活性，并使共享公共代码更加容易。此外，多平台项目现在也支持 Kotlin/Native！

与旧版模型的主要区别在于：

  * 在旧版模型中，需要将公共代码与平台相关代码分别放在独立的模块中，以 `expectedBy` 依赖项链接。
    现在，公共代码与平台相关代码放在相同模块的不同源根（source root）中，使项目更易于配置。
  * 对于不同的已支持平台，现在有大量的[预设的平台配置](building-mpp-with-gradle.html#supported-platforms)。
  * 更改了[依赖配置](building-mpp-with-gradle.html#adding-dependencies)；现在为每个源根分别指定依赖项。
  * 源集（source set）现在可以在任意平台子集之间共享（例如，在一个目标平台为 JS、Android 与 iOS 的模块中，可以有一个只在 Android 与 iOS 之间共享的源集）。
  * 现在支持[发布多平台库](building-mpp-with-gradle.html#publishing-a-multiplatform-library)了。

更多相关信息，请参考[多平台程序设计文档](multiplatform.html)。

## 契约

Kotlin 编译器会做大量的静态分析工作，以提供警告并减少模版代码。其中最显著的特性之一就是智能转换——能够根据类型检测自动转换类型。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun foo(s: String?) {
    if (s != null) s.length // 编译器自动将“s”转换为“String”
}
```

</div>

然而，一旦将这些检测提取到单独的函数中，所有智能转换都立即消失了：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun String?.isNotNull(): Boolean = this != null

fun foo(s: String?) {
    if (s.isNotNull()) s.length // 没有智能转换 :(
}
```

</div>

为了改善在此类场景中的行为，Kotlin 1.3 引入了称为 *契约* 的实验性机制。

*契约* 让一个函数能够以编译器理解的方式显式描述其行为。目前支持两大类场景：

- 通过声明函数调用的结果与所传参数值之间的关系来改进智能转换分析：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun require(condition: Boolean) {
    // 这是一种语法格式，告诉编译器：
    // “如果这个函数成功返回，那么传入的‘condition’为 true”
    contract { returns() implies condition }
    if (!condition) throw IllegalArgumentException(...)
}

fun foo(s: String?) {
    require(s is String)
    // s 在这里智能转换为“String”，因为否则
    // “require”会抛出异常
}
```

</div>

-  在存在高阶函数的情况下改进变量初始化的分析：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun synchronize(lock: Any?, block: () -> Unit) {
    // 告诉编译器：
    // “这个函数会在此时此处调用‘block’，并且刚好只调用一次”
    contract { callsInPlace(block, EXACTLY_ONCE) }
}

fun foo() {
    val x: Int
    synchronize(lock) {
        x = 42 // 编译器知道传给“synchronize”的 lambda 表达式刚好
               // 只调了一次，因此不会报重复赋值错
    }
    println(x) // 编译器知道一定会调用该 lambda 表达式而执行
               // 初始化操作，因此可以认为“x”在这里已初始化
}
```

</div>

### 标准库中的契约

`stdlib`（kotlin 标准库）已经利用契约带来了如上所述的对代码分析的改进。这部分契约是**稳定版的**，这意味着你现在就可以从改进的代码分析中受益，而无需任何额外的 opt-ins：

<div class="sample" data-min-compiler-version="1.3" markdown="1" theme="idea">

```kotlin
//sampleStart
fun bar(x: String?) {
    if (!x.isNullOrEmpty()) {
        println("length of '$x' is ${x.length}") // 哇，已经智能转换为非空！
    }
}
//sampleEnd
fun main() {
    bar(null)
    bar("42")
}
```

</div>

### 自定义契约

你也可以为你自己的函数来自定义契约，但是这一功能目前还是**实验性**的，因为目前的契约语法尚处于早期的原型态，很有可能在后面的版本被修改。而且，值得注意的是，目前 Kotlin 的编译器并没有对契约进行验证，所以需要开发者编写正确稳定的契约。

自定义契约通过调用标准库中的 `contract` 函数来引入，这个函数提供了 DSL：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun String?.isNullOrEmpty(): Boolean {
    contract {
        returns(false) implies (this@isNullOrEmpty != null)
    }
    return this == null || isEmpty()
}
```

</div>

更多语法细节和兼容性建议请查看[KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/kotlin-contracts.md)。

## 在变量中捕获`when`语句主语

在 Kotlin 1.3 中，允许你在变量中捕获 `when` 语句的主语：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun Request.getBody() =
        when (val response = executeRequest()) {
            is Success -> response.body
            is HttpError -> throw HttpException(response.status)
        }
```

</div>

当然你也可以在 `when` 语句之前把变量提取出来，`when` 语句中的 `val` 的作用域被限制在 `when` 语句的方法体中，借此防止了命名空间污染。点击[这里](control-flow.html#when-表达式)查看 `when` 语句说明文档。

## 接口中的伴生对象的 @JvmStatic 和 @JvmField 注解

在 Kotlin 1.3 中，允许将作为接口成员的伴生对象使用 `@JvmStatic` 和 `@JvmField` 注解标记。在 .class 文件中，这类成员将会对应接口解除作为成员，而是标记为 `static`。

例如，以下 Kotlin 代码：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
interface Foo {
    companion object {
        @JvmField
        val answer: Int = 42

        @JvmStatic
        fun sayHello() {
            println("Hello, world!")
        }
    }
}
```

</div>

和如下 Java 代码是相等的：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```java
interface Foo {
    public static int answer = 42;
    public static void sayHello() {
        // ...
    }
}
```

</div>

## 注解类中的嵌套声明

在 Kotlin 1.3 中，注解类允许包含嵌套的类、接口、对象和伴生对象：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
annotation class Foo {
    enum class Direction { UP, DOWN, LEFT, RIGHT }
    
    annotation class Bar

    companion object {
        fun foo(): Int = 42
        val bar: Int = 42
    }
}
```

</div>

## 无参`main`方法

一般来说，一个 Kotlin 程序的执行入口的方法签名为 `main(args: Array<String>)`，这里 `args` 表示传入的命令行参数。然而，并不是每个应用都支持命令行传参，所以这个参数通常并没有用到。

Kotlin 1.3 引入了一种简单的无参 `main`方法。现在 Kotlin 版的 `Hello,World`  缩短了19个字符！

<div class="sample" data-min-compiler-version="1.3" markdown="1" theme="idea">

```kotlin
fun main() {
    println("Hello, world!")
}
```

</div>

## 更多元的函数

在 Kotlin 中，函数类型用来表示带有许多参数的泛型类：`Function0<R>`, `Function1<P0, R>`, `Function2<P0, P1, R>`,... 这种方式有着列表是有限的问题，最多支持到 `Function22`。

Kotlin 1.3 放宽了限制，并且增加了函数更多元的支持：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun trueEnterpriseComesToKotlin(block: (Any, Any, ... /* 42 more */, Any) -> Any) {
    block(Any(), Any(), ..., Any())
}
```

</div>

## 渐进式模式

Kotlin 非常注重代码的稳定性和向后兼容性：Kotlin 的兼容策略为 “打破构建的变化”（既一个变化使得之前能够编译成功的代码，编译失败）只能在大版本中被引入（例如 1.2，1.3等）。

为了保证了代码安全和正确，我们相信，可以缩短许多用户修复编译器严峻问题的周期。所以，Kotlin 1.3 引入了*渐进式* 编译器模式，找个模式可以通过向编译器传递 `-progressive` 来开启。

在渐进式模式中，一些语法上的问题能够及时的得到修复。所有的修复手段都包含两条重要的属性：

— 保留源码对旧的编译器的兼容性支持，意味着所有对于**渐进式**编译器能编译的代码，也将能被**非渐进式**编译器编译。
— 在某些场景，只会让代码更*安全* — 例如，一些不健壮的智能转换将会被禁止，自动生成的代码将会变的更加稳定、可预测，等。

开启渐进式模式可能需要你重新某些代码，但不会太多 — 所有的开启渐进式需要修改都是精挑细选、仔细琢磨后的，而且提供工具迁移帮助。
我们期望对于那些一直保持最新代码版本的人，渐进式模式将会是一个非常棒的选择。

## 内联类

> 内联类从 Kotlin 1.3 开始支持，而且现阶段是**实验性**的。更多详情请看[这里](inline-classes.html#experimental-status-of-inline-classes)。
{:.note}


Kotlin 1.3 引入了一种新的声明方式 — `内联类`。内联类可视为常规类的限制版，值得一提的是，内联类必须有一个确切的属性：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
inline class Name(val s: String)
```

</div>

Kotlin 编译器将会利用这个限制，积极的提升内联类在运行时的表现，并且使用底层的属性值来替换内联类的实例，用于省略构造方法调用，减小 GC 压力，开启其他优化操作：

<div class="sample" data-min-compiler-version="1.3" markdown="1" theme="idea">

```kotlin
inline class Name(val s: String)
//sampleStart
fun main() {
    // 下一行不会调用构造方法
    // 而且在运行时，'name' 只会包含字符串 "Kotlin"
    val name = Name("Kotlin")
    println(name.s) 
}
//sampleEnd
```

</div>

点击[这里](inline-classes.html)查看内联类更多细节。

## 无符号整型

> 无符号整型从 Kotlin 1.3 开始支持，而且现阶段是**实验性**的。更多详情请看[这里](basic-types.html#experimental-status-of-unsigned-integers)。
{:.note}

Kotlin 1.3 引入了无符号整型类型：

- `kotlin.UByte`: an unsigned 8-bit integer, ranges from 0 to 255
- `kotlin.UShort`: an unsigned 16-bit integer, ranges from 0 to 65535
- `kotlin.UInt`: an unsigned 32-bit integer, ranges from 0 to 2^32 - 1
- `kotlin.ULong`: an unsigned 64-bit integer, ranges from 0 to 2^64 - 1

无符号类型也支持大多数有符号类型的功能：

<div class="sample" data-min-compiler-version="1.3" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
// 你可以使用字面前缀定义无符号类型：
val uint = 42u 
val ulong = 42uL
val ubyte: UByte = 255u

// 你可以把有符号类型通过标准库向无符号类型转换，反之亦然：
val int = uint.toInt()
val byte = ubyte.toByte()
val ulong2 = byte.toULong()

// 无符号类型支持同义操作符：
val x = 20u + 22u
val y = 1u shl 8
val z = "128".toUByte()
val range = 1u..5u
//sampleEnd
println("ubyte: $ubyte, byte: $byte, ulong2: $ulong2")
println("x: $x, y: $y, z: $z, range: $range")
}
```

</div>

点击[这里](basic-types.html#unsigned-integers)查看内联类更多细节。

## @JvmDefault

>`@JvmDefault` 从 Kotlin 1.3 开始支持，而且现阶段是**实验性**的。更多详情请看[这里](/api/latest/jvm/stdlib/kotlin.jvm/-jvm-default/index.html)。
{:.note}


Kotlin 兼容很多 Java 版本，包含接口中不允许方法默认实现的 Java6 和 Java7 。方便起见，Kotlin 编译器可以解决这个问题，但是这个解决方案将不会和 Java8 中的  `default`  方法兼容。

对于和 Java 的交互，这可能是个问题，所以 Kotlin 1.3 引入了 `@JvmDefalut` 注解。被该注解标记的方法将会被 JVM 编译为 `default` 方法。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
interface Foo {
    // 将会被编译为默认方法
    @JvmDefault
    fun foo(): Int = 42
}
```

</div>

> 注意！ 将你的 API 标记为 `@JvmDefault` 将会对二进制兼容性有严重影响。请确保你仔细阅读了[参考页面](/api/latest/jvm/stdlib/kotlin.jvm/-jvm-default/index.html)，在你的产品中使用`@Default`之前。
{:.note}

# 标准库

## 多平台随机数

在 Kotlin 1.3 之前，没有办法在所有平台上统一来生成随机数 — 我们只能采取依赖平台的特定解决方案，比如 JVM 上的 `java.util.Random`。这个版本将会解决这个问题，通过引入 `kotlin.random.Random`，这是个支持所有平台的类：

<div class="sample" data-min-compiler-version="1.3" markdown="1" theme="idea">

```kotlin
import kotlin.random.Random

fun main() {
//sampleStart
    val number = Random.nextInt(42)  // 数字区间 [0, limit)
    println(number)
//sampleEnd
}
```

</div>

## isNullOrEmpty 与 orEmpty

某些类型的 `isNullOrEmpty` 和 `orEmpty` 扩展函数已经存在于标准库中，对于前者，如果函数接受者是 `null`或者为空将会返回 `true` ；对于后者， 如果接收者是 `null` ，将会返回一个空实例。
Kotlin 1.3 为 `collections`、`maps` 和对象数组提供了类似的扩展函数。

## 非空数组间拷贝元素

对于非空数组类型，包括无符号数组类型，函数 `array.copyInto(targetArray, targetOffset, startIndex, endIndex)` 使得数组拷贝在 Kotlin 中更加简单。

<div class="sample" data-min-compiler-version="1.3" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val sourceArr = arrayOf("k", "o", "t", "l", "i", "n")
    val targetArr = sourceArr.copyInto(arrayOfNulls<String>(6), 3, startIndex = 3, endIndex = 6)
    println(targetArr.contentToString())
    
    sourceArr.copyInto(targetArr, startIndex = 0, endIndex = 3)
    println(targetArr.contentToString())
//sampleEnd
}
```

</div>

## associateWith

对于一个 list 一个非常常见的场景是，希望通过把 list 的每个元素作为 key，然后和某个 value 关联起来，来构建一个 map。这可以通过 `associate { it to getValue(it) }` 函数来完成，但是现在我们引入了一个更为效率和便捷的实现方式：`keys.associateWith { getValue(it) }`。

<div class="sample" data-min-compiler-version="1.3" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val keys = 'a'..'f'
    val map = keys.associateWith { it.toString().repeat(5).capitalize() }
    map.forEach { println(it) }
//sampleEnd
}
```

</div>

## ifEmpty 与 ifBlank

Collections, maps, object arrays, char sequence 和 sequence 现在都有 `ifEmpty` 函数，用于指定一个默认值，当接收者为空时：

<div class="sample" data-min-compiler-version="1.3" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    fun printAllUppercase(data: List<String>) {
        val result = data
        .filter { it.all { c -> c.isUpperCase() } }
            .ifEmpty { listOf("<no uppercase>") }
        result.forEach { println(it) }
    }
    
    printAllUppercase(listOf("foo", "Bar"))
    printAllUppercase(listOf("FOO", "BAR"))
//sampleEnd
}
```

</div>

Char sequences 和 strings 还额外拥有 `ifBlank` 扩展，和  `ifEmpty` 功能类似，但是用于检查一个 string 是否全是空格。

<div class="sample" data-min-compiler-version="1.3" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val s = "    \n"
    println(s.ifBlank { "<blank>" })
    println(s.ifBlank { null })
//sampleEnd
}
```

</div>

## 反射中的密封类

我们向 `kotlin-reflect` 添加了一个新的 API ： `KClass.sealedSubclasses`，用于列出所有密封类的子类型。

## 微小的改变

-  `Boolean` 类型现在支持伴生对象。
- `Any?.hashCode()`  扩展函数会在 `null` 的时候返回 0。
- `Char` 类型现在提供了 `MIN_VALUE`/`MAX_VALUE` 常量。
- `SIZE_BYTES` 和  `SIZE_BITS` 会作为原生类型伴生对象常量。

# 工具

## IDE 中的代码风格支持

Kotlin 1.3 在 IDE 中引入了代码[推荐风格](coding-conventions.html)支持。具体迁移指南查看[这里](code-style-migration-guide.html)。

## kotlinx.serialization

[kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization) 是一个提供支持多平台的序列化/反序列化的库。之前作为一个独立的库，从 Kotlin 1.3 起，它将会和其他编译器插件一样，包含在 Kotlin 编译器发行版中。主要区别在于，你不在需要自行关注 IDE 的序列化插件和 Kotlin IDE 插件的兼容性问题：因为 Kotlin IDE 插件已经支持序列化！

具体详情查看[这里](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/eap13.md)。

> 值得注意的是，即便 kotlinx.serialization  包含在 Kotlin 编译器发行版中，它仍是一个**实验性**功能。
{:.note}

## 脚本更新

> 注意，脚本是个**实验性**功能，对于现有的 API, 并没有任何兼容性的保障。
{:.note}

Kotlin 1.3 将继续完善 script API，引入了一些自定义脚本实验性的支持，比如添加额外属性，提供静态或者动态依赖等。

更详细的细节，请参考[KEEP-75](https://github.com/Kotlin/KEEP/blob/master/proposals/scripting-support.md)。

## 草稿文件支持

Kotlin 1.3 引入了对可运行的**草稿文件**支持。**草稿文件**是一个以 .kts 扩展名结尾的 kotlin 脚本文件，你可以在编辑器里直接运行和获取计算结果。

更多细节请关注[Scratches documentation](https://www.jetbrains.com/help/idea/scratches.html)。

