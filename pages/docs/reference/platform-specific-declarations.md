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

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
package org.jetbrains.foo

expect class Foo(bar: String) {
    fun frob()
}

fun main() {
    Foo("Hello").frob()
}
```
</div>

而这是相应的 JVM 模块：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
package org.jetbrains.foo

actual class Foo actual constructor(val bar: String) {
    actual fun frob() {
        println("Frobbing the $bar")
    }
}
```
</div>

This illustrates several important points:

  * An expected declaration in the common module and its actual counterparts always
    have exactly the same fully qualified name.
  * An expected declaration is marked with the `expect` keyword; the actual declaration
    is marked with the `actual` keyword.
  * All actual declarations that match any part of an expected declaration need to be marked
    as `actual`.
  * Expected declarations never contain any implementation code.

Note that expected declarations are not restricted to interfaces and interface members.
In this example, the expected class has a constructor and can be created directly from common code.
You can apply the `expect` modifier to other declarations as well, including top-level declarations and
annotations:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
// Common
expect fun formatString(source: String, vararg args: Any): String

expect annotation class Test

// JVM
actual fun formatString(source: String, vararg args: Any) =
    String.format(source, *args)
    
actual typealias Test = org.junit.Test
```
</div>

The compiler ensures that every expected declaration has actual declarations in all platform
modules that implement the corresponding common module, and reports an error if any actual declarations are 
missing. The IDE provides tools that help you create the missing actual declarations.

If you have a platform-specific library that you want to use in common code while providing your own
implementation for another platform, you can provide a typealias to an existing class as the actual
declaration:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
expect class AtomicRef<V>(value: V) {
  fun get(): V
  fun set(value: V)
  fun getAndSet(value: V): V
  fun compareAndSet(expect: V, update: V): Boolean
}

actual typealias AtomicRef<V> = java.util.concurrent.atomic.AtomicReference<V>
```
</div>
