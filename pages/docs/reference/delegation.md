---
type: doc
layout: reference
category: "Syntax"
title: "委托"
---

# 委托

## Property Delegation

Delegated properties are described on a separate page: [Delegated Properties](delegated-properties.html).

## Implementation by Delegation

[委托模式](https://zh.wikipedia.org/wiki/%E5%A7%94%E6%89%98%E6%A8%A1%E5%BC%8F)已经证明是实现继承的一个很好的替代方式，
而 Kotlin 可以零样板代码地原生支持它。
A class `Derived` can implement an interface `Base` by delegating all of its public members to a specified object:

<div class="sample" markdown="1">

``` kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main(args: Array<String>) {
    val b = BaseImpl(10)
    Derived(b).print()
}
```
</div>

`Derived` 的超类型列表中的 *by*{: .keyword }-子句表示 `b` 将会在 `Derived` 中内部存储，
并且编译器将生成转发给 `b` 的所有 `Base` 的方法。

### Overriding a member of an interface implemented by delegation 

[Overrides](classes.html#overriding-methods) work as you might expect: the compiler will use your `override` 
implementations instead of those in the delegate object. If we were to add `override fun print() { print("abc") }` to 
`Derived`, the program would print "abc" instead of "10" when `print` is called:

<div class="sample" markdown="1">

``` kotlin
interface Base {
    fun printMessage()
    fun printMessageLine()
}

class BaseImpl(val x: Int) : Base {
    override fun printMessage() { print(x) }
    override fun printMessageLine() { println(x) }
}

class Derived(b: Base) : Base by b {
    override fun printMessage() { print("abc") }
}

fun main(args: Array<String>) {
    val b = BaseImpl(10)
    Derived(b).printMessage()
    Derived(b).printMessageLine()
}
```
</div>

Note, however, that members overridden in this way do not get called from the members of the 
delegate object, which can only access its own implementations of the interface members:

<div class="sample" markdown="1">

``` kotlin
interface Base {
    val message: String
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override val message = "BaseImpl: x = $x"
    override fun print() { println(message) }
}

class Derived(b: Base) : Base by b {
    // This property is not accessed from b's implementation of `print`
    override val message = "Message of Derived"
}

fun main(args: Array<String>) {
    val b = BaseImpl(10)
    val derived = Derived(b)
    derived.print()
    println(derived.message)
}
```
</div>
