---
type: doc
layout: reference
category: "Syntax"
title: "枚举类"
---

# 枚举类

枚举类的最基本的用法是实现类型安全的枚举：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```
</div>

每个枚举常量都是一个对象。枚举常量用逗号分隔。

## 初始化

因为每一个枚举都是枚举类的实例，所以他们可以是这样初始化过的：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
}
```
</div>

## 匿名类

枚举常量也可以声明自己的匿名类：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
enum class ProtocolState {
    WAITING {
        override fun signal() = TALKING
    },

    TALKING {
        override fun signal() = WAITING
    };

    abstract fun signal(): ProtocolState
}
```
</div>

及相应的方法、以及覆盖基类的方法。注意，如果枚举类定义任何<!--
-->成员，要使用分号将成员定义中的枚举常量定义分隔开，就像<!--
-->在 Java 中一样。

枚举条目不能包含内部类以外的嵌套类型（已在 Kotlin 1.2 中弃用）。

## 在枚举类中实现接口

一个枚举类可以实现接口（但不能从类继承），可以为所有条目提供统一的接口成员实现，也可以在相应匿名类中为每个条目提供各自的实现。只需将接口添加到枚举类声明中即可，如下所示：

<div class="sample" markdown="1" theme="idea">

``` kotlin
import java.util.function.BinaryOperator
import java.util.function.IntBinaryOperator

//sampleStart
enum class IntArithmetics : BinaryOperator<Int>, IntBinaryOperator {
    PLUS {
        override fun apply(t: Int, u: Int): Int = t + u
    },
    TIMES {
        override fun apply(t: Int, u: Int): Int = t * u
    };
    
    override fun applyAsInt(t: Int, u: Int) = apply(t, u)
}
//sampleEnd

fun main(args: Array<String>) {
    val a = 13
    val b = 31
    for (f in IntArithmetics.values()) {
        println("$f($a, $b) = ${f.apply(a, b)}")
    }
}
```
</div>

## 使用枚举常量

就像在 Java 中一样，Kotlin 中的枚举类也有合成方法允许列出<!--
-->定义的枚举常量以及通过名称获取枚举常量。这些方法的<!--
-->签名如下（假设枚举类的名称是 `EnumClass`）：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>
```
</div>

如果指定的名称与类中定义的任何枚举常量均不匹配，`valueOf()` 方法将抛出 `IllegalArgumentException` 异常。

自 Kotlin 1.1 起，可以使用 `enumValues<T>()` 与 `enumValueOf<T>()` 函数以泛型的方式访问枚举类中的常量
：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}

printAllValues<RGB>() // 输出 RED, GREEN, BLUE
```
</div>

每个枚举常量都具有在枚举类声明中获取其名称与位置的属性：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
val name: String
val ordinal: Int
```
</div>

枚举常量还实现了 [Comparable](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparable/index.html) 接口，
其中自然顺序是它们在枚举类中定义的顺序。
