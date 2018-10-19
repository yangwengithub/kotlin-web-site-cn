---
type: doc
layout: reference
title: "What's Coming in Kotlin 1.3"
---

# Kotlin 1.3 的新特性

## 目录

- [协程](#协程正式发布)
- [Kotlin/Native](#Kotlin/Native)
- [多平台项目](#多平台项目)
- [契约](#契约)
- [变量中捕获when语句主语](#在变量中捕获when语句主语)
- [接口伴生对象的@JvmStatic和@JvmField注解](#接口中的伴生对象的 @JvmStatic 和 @JvmField 注解)
- [注解类的嵌套声明](#注解类中的嵌套声明)
- [无参main方法](#无参main方法)
- [函数更多元](#更多元的函数)
- [渐进模式](#渐进式模式)
- [内联类](#内联类)
- [无符号整型](#无符号整型)
- [@JvmDefault](#@JvmDefault)
- [标准库](#标准库)
- [工具](#工具)





## 协程（正式发布）

历经了漫长而充足的的测试，协程 API 终于正式发布！这意味着从 Kotlin 1.3 版本起，协程 API 将与其他正式发布的功能一样保持稳定。查看全新的[协程概览](coroutines-overview.html)。

Kotlin 1.3 把挂起函数对 callable 引用支持和对协程的支持加入了 Reflection API。

## Kotlin/Native

Kotlin 1.3 将继续完善对 Native 平台的支持。详情查看[Kotlin/Native概览](native-overview.html)。

## 多平台项目

在 1.3 ，为了更具表现力和灵活性，同时也为了更简单的共享代码，我们彻底重写了多平台项目的模型。并且，Kotlin/Native 现在将作为支持目标之一！详情查看[这里](multiplatform.html)。

## 契约

Kotlin 编译器会做大量的静态分析工作，为了提供警告和降低模版代码。其中最重要的功能之一就是**类型智能转换**—提供了基于类型检查自动转换类型的能力。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun foo(s: String?) {
    if (s != null) s.length // Compiler automatically casts 's' to 'String'
}
```

</div>

然而，一旦这些检查被提取到一个单独的函数中，类型智能转换就会失效：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun String?.isNotNull(): Boolean = this != null

fun foo(s: String?) {
    if (s.isNotNull()) s.length // No smartcast :(
}
```

</div>

为了改善这种情景下的行为，Kotlin 1.3 引入了**实验性**的机制，称之为**契约**。

**契约**允许一个函数通过一种能让编译器理解的方式来，显示的来描述它的行为。

目前，支持两类常用的情景：

- 通过声明函数的结果和参数的值的关系来改善类型智能分析：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun require(condition: Boolean) {
    // This is a syntax form, which tells compiler:
    // "if this function returns successfully, then passed 'condition' is true"
    contract { returns() implies condition }
    if (!condition) throw IllegalArgumentException(...)
}

fun foo(s: String?) {
    require(s is String)
    // s is smartcasted to 'String' here, because otherwise
    // 'require' would have throw an exception
}
```

</div>

-  在存在高阶函数的情况下，可以改善变量的初始化分析：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun synchronize(lock: Any?, block: () -> Unit) {
    // It tells compiler:
    // "This function will invoke 'block' here and now, and exactly one time"
    contract { callsInPlace(block, EXACTLY_ONCE) }
}

fun foo() {
    val x: Int
    synchronize(lock) {
        x = 42 // Compiler knows that lambda passed to 'synchronize' is called
               // exactly once, so no reassignment is reported
    }
    println(x) // Compiler knows that lambda will be definitely called, performing
               // initialization, so 'x' is considered to be initialized here
}
```

</div>

###标准库中的契约

`stdlib（kotlin 标准库）`已经在利用**契约**这一特性，改善了上述情景中代码分析情况。这一部分的契约功能已经趋于稳定，意味着你不需要任何额外的操作就可以享用改善后的代码分析。

<div class="sample" data-min-compiler-version="1.3" markdown="1" theme="idea">

```kotlin
//sampleStart
fun bar(x: String?) {
    if (!x.isNullOrEmpty()) {
        println("length of '$x' is ${x.length}") // Yay, smartcasted to not-null!
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

你也可以为你自己的函数来自定义契约，但是这一功能目前还是**实验性**的，因为目前的契约语法尚处于早期的原型态，很有可能在后面的版本被修改。而且，值得注意的是，目前 kotlin 的编译器并没有对契约进行验证，所以需要开发者编写正确稳定的契约。

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

在 kotlin 1.3 中，允许你在变量中捕获`when` 语句的主语：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun Request.getBody() =
        when (val response = executeRequest()) {
            is Success -> response.body
            is HttpError -> throw HttpException(response.status)
        }
```

</div>

当然你也可以在 `when` 语句之前把变量提取出来，`when`语句中的`val`的作用域被限制在`when`语句的方法体中，借此防止了命名空间污染。点击[这里](control-flow.html#when-expression)查看 `when` 语句说明文档。

## 接口中的伴生对象的 @JvmStatic 和 @JvmField 注解

在 Kotlin 1.3 中，允许将作为接口成员的伴生对象使用 `@JvmStatic` 和 `@JvmField` 注解标记。在 .class 文件中，这类成员将会对应接口解除作为成员，而是标记为 `static`。

例如，以下 Kotlin 代码：

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

一般来说，一个 Kotlin 程序的执行入口的方法签名为`main(args: Array<String>)`，这里`args`表示传入的命令行参数。然而，并不是每个应用都支持命令行传参，所以这个参数通常并没有用到。

Kotlin 1.3 引入了一种简单的无参 `main`方法。现在 Kotlin 版的 `Hello,World`  缩短了19个字符！

<div class="sample" data-min-compiler-version="1.3" markdown="1" theme="idea">

```kotlin
fun main() {
    println("Hello, world!")
}
```

</div>

## 更多元的函数

在 Kotlin 中，函数类型用来表示带有许多参数的泛型类：`Function0<R>`, `Function1<P0, R>`, `Function2<P0, P1, R>`,... 这种方式有着列表是有限的问题，最多支持到`Function22`。

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

为了保证了代码安全和正确，我们相信，可以缩短许多用户修复编译器严峻问题的周期。所以，Kotlin 1.3 引入了*渐进式* 编译器模式，找个模式可以通过向编译器传递`-progressive` 来开启。

在渐进式模式中，一些语法上的问题能够及时的得到修复。所有的修复手段都包含两条重要的属性：

  — 保留源码对旧的编译器的兼容性支持，意味着所有对于**渐进式**编译器能编译的代码，也将能被**非渐进式**编译器编译。

 — 在某些场景，只会让代码更*安全* — 例如，一些不健壮的智能转换将会被禁止，自动生成的代码将会变的更加稳定、可预测，等。

开启渐进式模式可能需要你重新某些代码，但不会太多 — 所有的开启渐进式需要修改都是精挑细选、仔细琢磨后的，而且提供工具迁移帮助。我们期望对于那些一直保持最新代码版本的人，渐进式模式将会是一个非常棒的选择。

## 内联类

> 内联类从 Kotlin 1.3 开始支持，而且现阶段是**实验性**的。更多详情请看[这里](inline-classes.html#experimental-status-of-inline-classes)。
>
> {:.note}

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
    // In the next line no constructor call happens, and
    // at the runtime 'name' contains just string "Kotlin"
    val name = Name("Kotlin")
    println(name.s) 
}
//sampleEnd
```

</div>

点击[这里](inline-classes.html)查看内联类更多细节。

## 无符号整型

> 无符号整型从 Kotlin 1.3 开始支持，而且现阶段是**实验性**的。更多详情请看[这里](basic-types.html#experimental-status-of-unsigned-integers)。
>
> {:.note}

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
// You can define unsigned types using literal suffixes
val uint = 42u 
val ulong = 42uL
val ubyte: UByte = 255u

// You can convert signed types to unsigned and vice versa via stdlib extensions:
val int = uint.toInt()
val byte = ubyte.toByte()
val ulong2 = byte.toULong()

// Unsigned types support similar operators:
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

>`@JvmDefault`从 Kotlin 1.3 开始支持，而且现阶段是**实验性**的。更多详情请看[这里](/api/latest/jvm/stdlib/kotlin.jvm/-jvm-default/index.html)。
>
>{:.note}

Kotlin 兼容很多 Java 版本，包含接口中不允许方法默认实现的 Java6 和 Java7 。方便起见，Kotlin 编译器可以解决这个问题，但是这个解决方案将不会和 Java8 中的 `default` 方法兼容。

对于和 Java 的交互，这可能是个问题，所以 Kotlin 1.3 引入了 `@JvmDefalut` 注解。被该注解标记的方法将会被 JVM 编译为 `default` 方法。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
interface Foo {
    // Will be generated as 'default' method
    @JvmDefault
    fun foo(): Int = 42
}
```

</div>

> 注意！ 将你的 API 标记为 `@JvmDefault` 将会对二进制兼容性有严重影响。请确保你仔细阅读了[参考页面](/api/latest/jvm/stdlib/kotlin.jvm/-jvm-default/index.html)，在你的产品中使用`@Default`之前。
> {:.note}

# 标准库

##多平台随机数

在 Kotlin 1.3 之前，没有办法在所有平台上统一来生成随机数 — 我们只能采取依赖平台的特定解决方案，比如 JVM 上的`java.util.Random`。这个版本将会解决这个问题，通过引入`kotlin.random.Random`，这是个支持所有平台的类：

<div class="sample" data-min-compiler-version="1.3" markdown="1" theme="idea">

```kotlin
import kotlin.random.Random

fun main() {
//sampleStart
    val number = Random.nextInt(42)  // number is in range [0, limit)
    println(number)
//sampleEnd
}
```

</div>

## isNullOrEmpty 和 orEmpty 拓展

某些类型的 `isNullOrEmpty` 和 `orEmpty` 拓展函数已经存在于标准库中，对于前者，如果函数接受者是 `null`或者为空将会返回 `true` ；对于后者， 如果接收者是 `null` ，将会返回一个空实例。Kotlin 1.3 为 `collections`、`maps` 和对象数组提供了类似的拓展函数。

## 非空数组间拷贝元素

对于非空数组类型，包括无符号数组类型，函数 `array.copyInto(targetArray, targetOffset, startIndex, endIndex)` 使得数组拷贝在 Kotlin 中更加简单。

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

对于一个 list 一个非常常见的场景是，希望通过把 list 的每个元素作为 key，然后和某个 value 关联起来，来构建一个 map。这可以通过 `associate { it to getValue(it) }` 函数来完成，但是现在我们引入了一个更为效率和便捷的实现方式：`keys.associateWith { getValue(it) }`

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

##ifEmpty 和 ifBlank 拓展

Collections、maps、object arrays、char sequence 和 sequence 现在都有 `ifEmpty` 函数，用于指定一个默认值，当接收者为空时：

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

Char sequences 和 strings 还额外拥有 `ifBlank` 拓展，和  `ifEmpty` 功能类似，但是用于检查一个 string 是否全是空格。

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

- `Any?.hashCode()`  拓展函数会在 `null` 的时候返回 0。

- `Char` 类型现在提供了 `MIN_VALUE`/`MAX_VALUE` 常量。

- `SIZE_BYTES` 和  `SIZE_BITS` 会作为原生类型伴生对象常量。

# 工具

## IDE 中的代码风格支持

Kotlin 1.3 在 IDE 中引入了代码[推荐风格](coding-conventions.html)支持。具体迁移指南查看[这里](code-style-migration-guide.html)。

## kotlinx.serialization

[kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization) 是一个提供支持多平台的序列化/反序列化的库。之前作为一个独立的库，从 Kotlin 1.3 起，它将会和其他编译器插件一样，包含在 Kotlin 编译器发行版中。主要区别在于，你不在需要自行关注 IDE 的序列化插件和 Kotlin IDE 插件的兼容性问题：因为 Kotlin IDE 插件已经支持序列化！

具体详情查看[这里](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/eap13.md)。

> 值得注意的是，即便 kotlinx.serialization  包含在 Kotlin 编译器发行版中，它仍是一个**实验性**功能。
>
> {:.note}

## 脚本更新

> 注意，脚本是个**实验性**功能，对于现有的 API, 并没有任何兼容性的保障。
>
> {:.note}

Kotlin 1.3 将继续完善 script API，引入了一些自定义脚本实验性的支持，比如添加额外属性，提供静态或者动态依赖等。

更详细的细节，请参考[KEEP-75](https://github.com/Kotlin/KEEP/blob/master/proposals/scripting-support.md)。

## 草稿文件支持

Kotlin 1.3 引入了对可运行的**草稿文件**支持。**草稿文件**是一个以 .kts 拓展名结尾的 kotlin 脚本文件，你可以在编辑器里直接运行和获取计算结果。

更多细节请关注[Scratches documentation](https://www.jetbrains.com/help/idea/scratches.html)。





