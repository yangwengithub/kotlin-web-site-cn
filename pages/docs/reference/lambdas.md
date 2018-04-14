---
type: doc
layout: reference
category: "Syntax"
title: "高阶函数与 lambda 表达式"
---

# 高阶函数与 lambda 表达式

Kotlin 函数都是[*头等的*](https://zh.wikipedia.org/wiki/%E5%A4%B4%E7%AD%89%E5%87%BD%E6%95%B0)，这意味着它们可以<!--
-->存储在变量与数据结构中、作为参数传递给其他<!--
-->[高阶函数](#高阶函数)以及从其他高阶函数返回。可以像操作任何其他<!--
-->非函数值一样操作函数。

为促成这点，作为一种静态类型编程语言的 Kotlin 使用一系列<!--
-->[函数类型](#函数类型)来表示函数并提供一组特定的语言结构，例如 [lambda 表达式](#lambda-表达式与匿名函数)。

## 高阶函数

高阶函数是将函数用作参数或返回值的函数。

一个不错的示例是集合的[函数式风格的 `fold`](https://en.wikipedia.org/wiki/Fold_(higher-order_function))，
它接受一个初始累积值与一个接合函数，并通过将当前累积值与每个集合元素连<!--
-->续接合起来代入累积值来构建返回值：

``` kotlin
fun <T, R> Collection<T>.fold(
    initial: R, 
    combine: (acc: R, nextElement: T) -> R
): R {
    var accumulator: R = initial
    for (element: T in this) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}
```

In the code above, the parameter `combine` has a [function type](#function-types) `(R, T) -> R`, so it accepts a function that 
takes two arguments of types `R` and `T` and returns a value of type `R`. 
It is [invoked](#invoking-a-function-type-instance) inside the *for*{: .keyword }-loop, and the return value is 
then assigned to `accumulator`.

To call `fold`, we need to pass it an [instance of the function type](#instantiating-a-function-type) as an argument, and lambda expressions ([described in more detail below](#lambda-expressions-and-anonymous-functions)) are widely used for 
this purpose at higher-order function call sites:

<div class="sample" markdown="1">

```kotlin
fun main(args: Array<String>) {
    //sampleStart
    val items = listOf(1, 2, 3, 4, 5)
    
    // Lambdas are code blocks enclosed in curly braces.
    items.fold(0, { 
        // When a lambda has parameters, they go first, followed by '->'
        acc: Int, i: Int -> 
        print("acc = $acc, i = $i, ") 
        val result = acc + i
        println("result = $result")
        // The last expression in a lambda is considered the return value:
        result
    })
    
    // Parameter types in a lambda are optional if they can be inferred:
    val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })
    
    // Function references can also be used for higher-order function calls:
    val product = items.fold(1, Int::times)
    //sampleEnd
    println("joinedToString = $joinedToString")
    println("product = $product")
}
```
</div>

The following sections explain in more detail the concepts mentioned so far.

## Function types

Kotlin uses a family of function types like `(Int) -> String` for declarations that deal with functions: `val onClick: () -> Unit = ...`.

These types have a special notation that corresponds to the signatures of the functions, i.e. their parameters and return values:

* All function types have a parenthesized parameter types list and a return type: `(A, B) -> C` denotes a type that
 represents functions taking two arguments of types `A` and `B` and returning a value of type `C`. 
 The parameter types list may be empty, as in `() -> A`. The [`Unit` return type](functions.html#unit-returning-functions) 
 cannot be omitted. 
 
* Function types can optionally have an additional *receiver* type, which is specified before a dot in the notation:
 the type `A.(B) -> C` represents functions that can be called on a receiver object of `A` with a parameter of `B` and
 return a value of `C`.
 [Function literals with receiver](#function-literals-with-receiver) are often used along with these types.
 
* [Suspending functions](coroutines.html#suspending-functions) belong to function types of a special kind, which have a *suspend*{: .keyword} modifier in the 
 notation, such as `suspend () -> Unit` or `suspend A.(B) -> C`.
 
The function type notation can optionally include names for the function parameters: `(x: Int, y: Int) -> Point`.
These names can be used for documenting the meaning of the parameters.

> To specify that a function type is [nullable](null-safety.html#nullable-types-and-non-null-types), use parentheses: `((Int, Int) -> Int)?`.
> 
> Function types can be combined using parentheses: `(Int) -> ((Int) -> Unit)`
>
> The arrow notation is right-associative, `(Int) -> (Int) -> Unit` is equivalent to the previous example, but not to 
`((Int) -> (Int)) -> Unit`.

You can also give a function type an alternative name by using [a type alias](type-aliases.html):

```kotlin
typealias ClickHandler = (Button, ClickEvent) -> Unit
```
 
### Instantiating a function type

There are several ways to obtain an instance of a function type:

* Using a code block within a function literal, in one of the forms: 
    * a [lambda expression](#lambda-expressions-and-anonymous-functions): `{ a, b -> a + b }`,
    * an [anonymous function](#anonymous-functions): `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`
    
   [Function literals with receiver](#function-literals-with-receiver) can be used as values of function types with receiver.
   
* Using a callable reference to an existing declaration:
    * a top-level, local, member, or extension [function](reflection.html#function-references): `::isOdd`, `String::toInt`,
    * a top-level, member, or extension [property](reflection.html#property-references): `List<Int>::size`,
    * a [constructor](reflection.html#constructor-references): `::Regex`
    
   These include [bound callable references](reflection.html#bound-function-and-property-references-since-11) that point to a member of a particular instance: `foo::toString`.
   
* Using instances of a custom class that implements a function type as an interface: 

    ```kotlin
    class IntTransformer: (Int) -> Int {
        override operator fun invoke(x: Int): Int = TODO()
    }
    
    val intFunction: (Int) -> Int = IntTransformer() 
    ```

The compiler can infer the function types for variables if there is enough information:

```kotlin
val a = { i: Int -> i + 1 } // The inferred type is (Int) -> Int
```

*Non-literal* values of function types with and without receiver are interchangeable, so that the receiver can stand in 
for the first parameter, and vice versa. For instance, a value of type `(A, B) -> C` can be passed or assigned 
 where a `A.(B) -> C` is expected and the other way around:
 
<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
    //sampleStart
    val repeat: String.(Int) -> String = { times -> repeat(times) }
    val twoParameters: (String, Int) -> String = repeat // OK
    
    fun runTransformation(f: (String, Int) -> String): String {
        return f("hello", 3)
    }
    val result = runTransformation(repeat) // OK
    //sampleEnd
    println("result = $result")
}
```
</div>

> Note that a function type with no receiver is inferred by default, even if a variable is initialized with a reference
> to an extension function. 
> To alter that, specify the variable type explicitly.

### Invoking a function type instance  

A value of a function type can be invoked by using its [`invoke(...)` operator](operator-overloading.html#invoke): `f.invoke(x)` or just `f(x)`.

If the value has a receiver type, the receiver object should be passed as the first argument.
Another way to invoke a value of a function type with receiver is to prepend it with the receiver object,
as if the value were an [extension function](extensions.html): `1.foo(2)`,

Example:

<div class="sample" markdown="1">

``` kotlin
fun main(args: Array<String>) {
    //sampleStart
    val stringPlus: (String, String) -> String = String::plus
    val intPlus: Int.(Int) -> Int = Int::plus
    
    println(stringPlus.invoke("<-", "->"))
    println(stringPlus("Hello, ", "world!")) 
    
    println(intPlus.invoke(1, 1))
    println(intPlus(1, 2))
    println(2.intPlus(3)) // extension-like call
    //sampleEnd
}
```
</div>

### Inline functions

Sometimes it is beneficial to use [inline functions](inline-functions.html), which provide flexible control flow,
for higher-order functions.

## Lambda 表达式与匿名函数

lambda 表达式与匿名函数是“函数字面值”，即未声明的函数，
但立即做为表达式传递。考虑下面的例子：

``` kotlin
max(strings, { a, b -> a.length < b.length })
```

函数 `max` 是一个高阶函数，它接受一个函数作为第二个参数。
其第二个参数是一个表达式，它本身是一个函数，即函数字面值，它等价于<!--
-->以下命名函数：

``` kotlin
fun compare(a: String, b: String): Boolean = a.length < b.length
```

### Lambda 表达式语法

Lambda 表达式的完整语法形式如下：

``` kotlin
val sum = { x: Int, y: Int -> x + y }
```

lambda 表达式总是括在花括号中，
完整语法形式的参数声明放在花括号内，并有可选的类型标注，
函数体跟在一个 `->` 符号之后。如果推断出的该 lambda 的返回类型不是 `Unit`，那么该 lambda 主体中的最后一个（或可能是单个）表达式会视为返回值。

如果我们把所有可选标注都留下，看起来如下：

``` kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
```

### Passing a lambda to the last parameter

In Kotlin, there is a convention that if the last parameter of a function accepts a function, a lambda expression that is 
passed as the corresponding argument can be placed outside the parentheses:

``` kotlin
val product = items.fold(1) { acc, e -> acc * e }
```

If the lambda is the only argument to that call, the parentheses can be omitted entirely: 

``` kotlin
run { println("...") }
```

{:#it单个参数的隐式名称}

### `it`：单个参数的隐式名称

一个 lambda 表达式只有一个参数是很常见的。

If the compiler can figure the signature out itself, it is allowed not to declare the only parameter and omit `->`. 
The parameter will be implicitly declared under the name `it`:

``` kotlin
ints.filter { it > 0 } // 这个字面值是“(it: Int) -> Boolean”类型的
```

### Returning a value from a lambda expression

我们可以使用[限定的返回](returns.html#标签处返回)语法从 lambda 显式返回一个值。
否则，将隐式返回最后一个表达式的值。

因此，以下两个片段是等价的：

``` kotlin
ints.filter {
    val shouldFilter = it > 0 
    shouldFilter
}

ints.filter {
    val shouldFilter = it > 0 
    return@filter shouldFilter
}
```

This convention, along with [passing a lambda expression outside parentheses](#passing-a-lambda-to-the-last-parameter), allows for 
[LINQ-style](http://msdn.microsoft.com/en-us/library/bb308959.aspx) code:

``` kotlin
strings.filter { it.length == 5 }.sortedBy { it }.map { it.toUpperCase() }
```

{:#下划线用于未使用的变量自-11-起}

### 下划线用于未使用的变量（自 1.1 起）

如果 lambda 表达式的参数未使用，那么可以用下划线取代其名称：

``` kotlin
map.forEach { _, value -> println("$value!") }
```

{:#在-lambda-表达式中解构自-11-起}

### 在 lambda 表达式中解构（自 1.1 起）

在 lambda 表达式中解构是作为[解构声明](multi-declarations.html#在-lambda-表达式中解构自-11-起)的一部分描述的。

### 匿名函数

上面提供的 lambda 表达式语法缺少的一个东西是指定函数的返回类型的<!--
-->能力。在大多数情况下，这是不必要的。因为返回类型可以自动推断出来。然而，如果<!--
-->确实需要显式指定，可以使用另一种语法： _匿名函数_ 。

``` kotlin
fun(x: Int, y: Int): Int = x + y
```

匿名函数看起来非常像一个常规函数声明，除了其名称省略了。其函数体<!--
-->可以是表达式（如上所示）或代码块：

``` kotlin
fun(x: Int, y: Int): Int {
    return x + y
}
```

参数和返回类型的指定方式与常规函数相同，除了<!--
-->能够从上下文推断出的参数类型可以省略：

``` kotlin
ints.filter(fun(item) = item > 0)
```

匿名函数的返回类型推断机制与正常函数一样：对于具有表达式函数体的匿名函数将自动<!--
-->推断返回类型，而具有代码块函数体的返回类型必须显式<!--
-->指定（或者已假定为 `Unit`）。

请注意，匿名函数参数总是在括号内传递。 允许将函数<!--
-->留在圆括号外的简写语法仅适用于 lambda 表达式。

Lambda表达式与匿名函数之间的另一个区别是<!--
-->[非局部返回](inline-functions.html#非局部返回)的行为。一个不带标签的 *return*{: .keyword } 语句<!--
-->总是在用 *fun*{: .keyword } 关键字声明的函数中返回。这意味着 lambda 表达式中的 *return*{: .keyword }
将从包含它的函数返回，而匿名函数中的 *return*{: .keyword }
将从匿名函数自身返回。

### 闭包

Lambda 表达式或者匿名函数（以及[局部函数](functions.html#局部函数)和[对象表达式](object-declarations.html#对象表达式)）
可以访问其 _闭包_ ，即在外部作用域中声明的变量。 与 Java 不同的是可以修改闭包中捕获的变量：

``` kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)
```

### 带接收者的函数字面值

[Function types](#function-types) with receiver, such as `A.(B) -> C`, can be instantiated with a special form of function literals – 
function literals with receiver.

As said above, Kotlin provides the ability [to call an instance](#invoking-a-function-type-instance) of a function type with receiver providing the _receiver object_.

Inside the body of the function literal, the receiver object passed to a call becomes an *implicit* *this*{: .keyword}, so that you 
can access the members of that receiver object without any additional qualifiers, or access the receiver object 
using a [`this` expression](this-expressions.html).
 
This behavior is similar to [extension functions](extensions.html), which also allow you to access the members of the receiver object 
inside the body of the function.

Here is an example of a function literal with receiver along with its type, where `plus` is called on the 
receiver object:

``` kotlin
val sum: Int.(Int) -> Int = { other -> plus(other) } 
```

匿名函数语法允许你直接指定函数字面值的接收者类型。
如果你需要使用带接收者的函数类型声明一个变量，并在之后使用它，这将非常有用。

``` kotlin
val sum = fun Int.(other: Int): Int = this + other
```

当接收者类型可以从上下文推断时，lambda 表达式可以用作带接收者的函数字面值。
One of the most important examples of their usage is [type-safe builders](type-safe-builders.html):

``` kotlin
class HTML {
    fun body() { …… }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()  // 创建接收者对象
    html.init()        // 将该接收者对象传给该 lambda
    return html
}

html {       // 带接收者的 lambda 由此开始
    body()   // 调用该接收者对象的一个方法
}
```


