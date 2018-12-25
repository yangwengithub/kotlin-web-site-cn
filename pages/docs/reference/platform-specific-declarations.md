---
type: doc
layout: reference
category: "Other"
title: "平台相关声明"
---

## 平台相关声明

> 多平台项目是 Kotlin 1.2 与 Kotlin 1.3 中的一个实验性特性。本文档中描述的所有语言<!--
-->与工具特性在未来的 Kotlin 版本中都可能会有所变化。
{:.note}

Kotlin 多平台代码的一个关键功能是让公共代码能够<!--
-->依赖平台相关声明的一种方式。在其他语言中，这通常可以通过<!--
-->在公共代码中构建一组接口并在平台相关模块中实现这些接口来完成<!--
-->。然而，当在其中某个平台上有一个<!--
-->实现所需功能的库，并且希望直接使用该库的 API
而无需额外包装器时，这种方法并不理想。此外，它需要以接口表示公共声明，这<!--
-->无法覆盖所有可能情况。

作为替代方案，Kotlin 提供了一种 _预期声明与实际声明_ 的机制。
利用这种机制，公共模块可以定义 _预期声明_ ，而平台模块<!--
-->可以提供与预期声明相对应的 _实际声明_ 。
为了了解其工作机制，我们先来看一个示例。此代码是公共模块的一部分：



```kotlin
package org.jetbrains.foo

expect class Foo(bar: String) {
    fun frob()
}

fun main() {
    Foo("Hello").frob()
}
```


而这是相应的 JVM 模块：



```kotlin
package org.jetbrains.foo

actual class Foo actual constructor(val bar: String) {
    actual fun frob() {
        println("Frobbing the $bar")
    }
}
```


这阐明了几个要点：

  * 公共模块中的预期声明与其对应的实际声明始终<!--
    -->具有完全相同的完整限定名。
  * 预期声明标有 `expect` 关键字；实际声明<!--
    -->标有 `actual` 关键字。
  * 与预期声明的任何部分匹配的所有实际声明都需要标记<!--
    -->为 `actual`。
  * 预期声明决不包含任何实现代码。

请注意，预期声明并不限于接口与接口成员。
在本例中，预期的类有一个构造函数，于是可以直接在公共代码中创建该类。
还可以将 `expect` 修饰符应用于其他声明，包括顶层声明以及<!--
-->注解：



```kotlin
// 公共
expect fun formatString(source: String, vararg args: Any): String

expect annotation class Test

// JVM
actual fun formatString(source: String, vararg args: Any) =
    String.format(source, *args)
    
actual typealias Test = org.junit.Test
```


编译器会确保每个预期声明在实现相应公共模块的所有平台模块中<!--
-->都具有实际声明，并且如果缺少任何实际声明，编译器都会报错<!--
-->。IDE 提供了帮助创建所缺失实际声明的工具。

如果有一个希望用在公共代码中的平台相关的库，同时为其他平台提供自己<!--
-->的实现，那么可以将现有类的别名作为实际<!--
-->声明：



```kotlin
expect class AtomicRef<V>(value: V) {
  fun get(): V
  fun set(value: V)
  fun getAndSet(value: V): V
  fun compareAndSet(expect: V, update: V): Boolean
}

actual typealias AtomicRef<V> = java.util.concurrent.atomic.AtomicReference<V>
```

