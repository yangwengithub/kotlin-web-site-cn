---
type: doc
layout: reference
category: "JavaScript"
title: "Kotlin 中调用 JavaScript"
---

# Kotlin 中调用 JavaScript

Kotlin 已被设计为能够与 Java 平台轻松互操作。它将 Java 类视为 Kotlin 类，并且
Java 也将 Kotlin 类视为 Java 类。但是，JavaScript 是一种动态类型语言，这意味着<!--
-->它不会在编译期检查类型。你可以通过[动态](dynamic-type.html)类型在
Kotlin 中自由地与 JavaScript 交流，但是如果你想要 Kotlin 类型系统的全部威力
，你可以为 JavaScript 库创建 Kotlin 头文件。


## 内联 JavaScript

你可以使用 [js("……")](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.js/js.html) 函数将一些 JavaScript 代码嵌入到 Kotlin 代码中。
例如：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun jsTypeOf(o: Any): String {
    return js("typeof o")
}
```
</div>

`js` 的参数必须是字符串常量。因此，以下代码是不正确的：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
fun jsTypeOf(o: Any): String {
    return js(getTypeof() + " o") // 此处报错
}
fun getTypeof() = "typeof"
```
</div>


## `external` 修饰符

要告诉 Kotlin 某个声明是用纯 JavaScript 编写的，你应该用 `external` 修饰符来标记它。
当编译器看到这样的声明时，它假定相应类、函数或<!--
-->属性的实现由开发人员提供，因此不会尝试从声明中生成任何 JavaScript 代码。
这意味着你应该省略 `external` 声明内容的代码体。例如：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
external fun alert(message: Any?): Unit

external class Node {
    val firstChild: Node

    fun append(child: Node): Node

    fun removeChild(child: Node): Node

    // 等等
}

external val window: Window
```
</div>

请注意，嵌套的声明会继承 `external` 修饰符，即在 `Node` 类中，我们在<!--
-->成员函数和属性之前并不放置 `external`。

`external` 修饰符只允许在包级声明中使用。 你不能声明一个非 `external` 类的 `external` 成员。


### 声明类的（静态）成员

在 JavaScript 中，你可以在原型或者类本身上定义成员。即：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` javascript
function MyClass() { …… }
MyClass.sharedMember = function() { /* 实现 */ };
MyClass.prototype.ownMember = function() { /* 实现 */ };
```
</div>

Kotlin 中没有这样的语法。然而，在 Kotlin 中我们有伴生（`companion`）对象。Kotlin 以特殊的方式处理
`external` 类的伴生对象：替代期待一个对象的是，它假定伴生对象的成员<!--
-->就是该类自身的成员。要描述来自上例中的 `MyClass`，你可以这样写：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
external class MyClass {
    companion object {
        fun sharedMember()
    }

    fun ownMember()
}
```
</div>


### 声明可选参数

一个外部函数可以有可选参数。
JavaScript 实现实际上如何计算这些参数的默认值，是 Kotlin 所不知道的，
因此在 Kotlin 中不可能使用通常的语法声明这些参数。
你应该使用以下语法：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
external fun myFunWithOptionalArgs(x: Int,
    y: String = definedExternally,
    z: Long = definedExternally)
```
</div>

这意味着你可以使用一个必需参数和两个可选参数来调用 `myFunWithOptionalArgs`（它们的<!--
-->默认值由一些 JavaScript 代码算出）。


### 扩展 JavaScript 类

你可以轻松扩展 JavaScript 类，因为它们是 Kotlin 类。只需定义一个 `external` 类并用<!--
-->非 `external` 类扩展它。例如：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
external open class HTMLElement : Element() {
    /* 成员 */
}

class CustomElement : HTMLElement() {
    fun foo() {
        alert("bar")
    }
}
```
</div>

有一些限制：

1. 当一个外部基类的函数被签名重载时，不能在派生类中覆盖它。
2. 不能覆盖一个使用默认参数的函数。

请注意，你无法用外部类扩展非外部类。


### `external` 接口

JavaScript 没有接口的概念。当函数期望其参数支持 `foo`
和 `bar` 方法时，只需传递实际具有这些方法的对象。
对于静态类型的 Kotlin，你可以使用接口来表达这点，例如：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
external interface HasFooAndBar {
    fun foo()

    fun bar()
}

external fun myFunction(p: HasFooAndBar)
```
</div>

外部接口的另一个使用场景是描述设置对象。例如：

<div class="sample" markdown="1" theme="idea" data-highlight-only auto-indent="false">
``` kotlin
external interface JQueryAjaxSettings {
    var async: Boolean

    var cache: Boolean

    var complete: (JQueryXHR, String) -> Unit

    // 等等
}

fun JQueryAjaxSettings(): JQueryAjaxSettings = js("{}")

external class JQuery {
    companion object {
        fun get(settings: JQueryAjaxSettings): JQueryXHR
    }
}

fun sendQuery() {
    JQuery.get(JQueryAjaxSettings().apply {
        complete = { (xhr, data) ->
            window.alert("Request complete")
        }
    })
}
```
</div>

外部接口有一些限制：

1. 它们不能在 `is` 检查的右侧使用。
2. `as` 转换为外部接口总是成功（并在编译时产生警告）。
3. 它们不能作为具体化类型参数传递。
4. 它们不能用在类的字面值表达式（即 `I::class`）中。
