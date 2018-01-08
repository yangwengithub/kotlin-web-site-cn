---
type: doc
layout: reference
category: "Syntax"
title: "返回与跳转：break 与 continue"
---

# 返回和跳转

Kotlin 有三种结构化跳转表达式：

* *return*{: .keyword }。默认从最直接包围它的函数或者[匿名函数](lambdas.html#匿名函数)返回。
* *break*{: .keyword }。终止最直接包围它的循环。
* *continue*{: .keyword }。继续下一次最直接包围它的循环。

所有这些表达式都可以用作更大表达式的一部分：

``` kotlin
val s = person.name ?: return
```

这些表达式的类型是 [Nothing 类型](exceptions.html#nothing-类型)。

## Break 与 Continue 标签

在 Kotlin 中任何表达式都可以用标签（*label*{: .keyword }）来标记。
标签的格式为标识符后跟 `@` 符号，例如：`abc@`、`fooBar@`都是有效的标签（参见[语法](grammar.html#labelReference)）。
要为一个表达式加标签，我们只要在其前加标签即可。

``` kotlin
loop@ for (i in 1..100) {
    // ……
}
```

现在，我们可以用标签限制 *break*{: .keyword } 或者*continue*{: .keyword }：

``` kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (……) break@loop
    }
}
```

标签限制的 break 跳转到刚好位于该标签指定的循环后面的执行点。
*continue*{: .keyword } 继续标签指定的循环的下一次迭代。


## 标签处返回

Kotlin 有函数字面量、局部函数和对象表达式。因此 Kotlin 的函数可以被嵌套。
标签限制的 *return*{: .keyword } 允许我们从外层函数返回。
最重要的一个用途就是从 lambda 表达式中返回。回想一下我们这么写的时候：

``` kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return  // nonlocal return from inside lambda directly to the caller of foo()
        print(it)
    }
}
```

这个 *return*{: .keyword } 表达式从最直接包围它的函数即 `foo` 中返回。
（注意，这种非局部的返回只支持传给[内联函数](inline-functions.html)的 lambda 表达式。）
如果我们需要从 lambda 表达式中返回，我们必须给它加标签并用以限制 *return*{: .keyword }。

``` kotlin
fun foo() {
    ints.forEach lit@{
        if (it == 0) return@lit
        print(it)
    }
}
```

现在，它只会从 lambda 表达式中返回。通常情况下使用隐式标签更方便。
该标签与接受该 lambda 的函数同名。

``` kotlin
fun foo() {
    ints.forEach {
        if (it == 0) return@forEach
        print(it)
    }
}
```

或者，我们用一个[匿名函数](lambdas.html#匿名函数)替代 lambda 表达式。
匿名函数内部的 *return*{: .keyword } 语句将从该匿名函数自身返回

``` kotlin
fun foo() {
    ints.forEach(fun(value: Int) {
        if (value == 0) return  // local return to the caller of the anonymous fun, i.e. the forEach loop
        print(value)
    })
}
```

当要返一个回值的时候，解析器优先选用标签限制的 return，即

``` kotlin
return@a 1
```

意为“从标签 `@a` 返回 1”，而不是“返回一个标签标注的表达式 `(@a 1)`”。
