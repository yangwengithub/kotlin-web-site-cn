---
type: doc
layout: reference
category: "Syntax"
title: "枚举类"
---

# 枚举类

枚举类的最基本的用法是实现类型安全的枚举：

``` kotlin
enum class Direction {
    NORTH, SOUTH, WEST, EAST
}
```

每个枚举常量都是一个对象。枚举常量用逗号分隔。

## 初始化

因为每一个枚举都是枚举类的实例，所以他们可以是这样初始化过的：

``` kotlin
enum class Color(val rgb: Int) {
        RED(0xFF0000),
        GREEN(0x00FF00),
        BLUE(0x0000FF)
}
```

## 匿名类

枚举常量也可以声明自己的匿名类：

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

及相应的方法、以及覆盖基类的方法。注意，如果枚举类定义任何<!--
-->成员，要使用分号将成员定义中的枚举常量定义分隔开，就像<!--
-->在 Java 中一样。

枚举条目不能包含内部类以外的嵌套类型（已在 Kotlin 1.2 中弃用）。

## Implementing Interfaces in Enum Classes

An enum class may implement an interface (but not derive from a class), providing either a single interface members implementation for all of the entries, or separate ones for each entry within its anonymous class. This is done by adding the interfaces to the enum class declaration as follows:

<div class="sample" markdown="1">

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

``` kotlin
EnumClass.valueOf(value: String): EnumClass
EnumClass.values(): Array<EnumClass>
```

如果指定的名称与类中定义的任何枚举常量均不匹配，`valueOf()` 方法将抛出 `IllegalArgumentException` 异常。

自 Kotlin 1.1 起，可以使用 `enumValues<T>()` 和 `enumValueOf<T>()` 函数以泛型的方式访问枚举类中的常量
：

``` kotlin
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}

printAllValues<RGB>() // 输出 RED, GREEN, BLUE
```

每个枚举常量都具有在枚举类声明中获取其名称和位置的属性：

``` kotlin
val name: String
val ordinal: Int
```

枚举常量还实现了 [Comparable](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparable/index.html) 接口，
其中自然顺序是它们在枚举类中定义的顺序。
