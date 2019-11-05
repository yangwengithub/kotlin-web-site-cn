---
type: doc
layout: reference
category: "Syntax"
title: "作用域函数"
---

# 作用域函数

Kotlin 标准库包含几个函数，它们的唯一目的是在对象的上下文中执行代码块。当对一个对象调用这样的函数并提供一个 [lambda 表达式](lambdas.html)时，它会形成一个临时作用域。在此作用域中，可以访问该对象而无需其名称。这些函数称为*作用域函数*。共有以下五种：`let`、`run`、`with`、`apply` 以及 `also`。

这些函数基本上做了同样的事情：在一个对象上执行一个代码块。不同的是这个对象在块中如何使用，以及整个表达式的结果是什么。

下面是作用域函数的典型用法:

<div class="sample" markdown="1" theme="idea">

```kotlin
data class Person(var name: String, var age: Int, var city: String) {
    fun moveTo(newCity: String) { city = newCity }
    fun incrementAge() { age++ }
}

fun main() {
//sampleStart
    Person("Alice", 20, "Amsterdam").let {
        println(it)
        it.moveTo("London")
        it.incrementAge()
        println(it)
    }
//sampleEnd
}
```

</div>

如果不使用 `let` 来写这段代码, 就必须引入一个新变量，并在每次使用它时重复其名称。

<div class="sample" markdown="1" theme="idea">

```kotlin
data class Person(var name: String, var age: Int, var city: String) {
    fun moveTo(newCity: String) { city = newCity }
    fun incrementAge() { age++ }
}

fun main() {
//sampleStart
    val alice = Person("Alice", 20, "Amsterdam")
    println(alice)
    alice.moveTo("London")
    alice.incrementAge()
    println(alice)
//sampleEnd
}
```

</div>

作用域函数没有引入任何新的技术，但是它们可以使你的代码更加简洁易读。

由于作用域函数的相似性质，为你的案例选择正确的函数可能有点棘手。选择主要取决于你的意图和项目中使用的一致性。下面我们将详细描述各种作用域函数及其约定用法之间的区别。

## 区别

由于作用域函数本质上都非常相似，因此了解它们之间的区别很重要。每个作用域函数之间有两个主要区别：
* 引用上下文对象的方式
* 返回值

### 上下文对象：`this` 还是 `it`

在作用域函数的 lambda 表达式里，上下文对象可以不使用其实际名称而是使用一个更简短的引用来访问。每个作用域函数都使用以下两种方式之一来访问上下文对象：作为 lambda 的[接收者](lambdas.html#带有接收者的函数字面值)（`this`）或者作为 lambda 的参数（`it`）。两者都提供了同样的功能，因此我们将针对不同的场景描述两者的优缺点，并提供使用建议。

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
    val str = "Hello"
    // this
    str.run {
        println("The receiver string length: $length")
        //println("The receiver string length: ${this.length}") // 和上句效果相同
    }

    // it
    str.let {
        println("The receiver string's length is ${it.length}")
    }
}
```

</div>

#### this

`run`、`with` 以及 `apply` 通过关键字 `this` 引用上下文对象。因此，在它们的 lambda 表达式中可以像在普通的类函数中一样访问上下文对象。在大多数场景，当你访问接收者对象时你可以省略 `this`，来让你的代码更简短。相对地，如果省略了 `this`，就很难区分接收者对象的成员和外部对象或函数。因此，对于主要对对象成员进行操作的 lambda，建议将上下文对象作为接收者（`this`）：调用其函数或赋值其属性。

<div class="sample" markdown="1" theme="idea">

```kotlin
data class Person(var name: String, var age: Int = 0, var city: String = "")

fun main() {
//sampleStart
    val adam = Person("Adam").apply { 
        age = 20                       // 和 this.age = 20 或者 adam.age = 20 一样
        city = "London"
    }
//sampleEnd
}
```

</div>

#### it

反过来，`let` 和 `also` 将上下文对象作为 lambda 表达式参数。如果没有指定参数名，对象可以用隐式默认名称 `it` 访问。`it` 比 `this` 简短，带有 `it` 的表达通常更容易阅读。然而，当调用对象函数或属性时，不能像 `this` 这样隐式地访问对象。因此，当上下文对象在作用域中主要用作函数调用中的参数时，使用 `it` 作为上下文对象会更好。如果在代码块中使用多个变量，则 `it` 也更好。

<div class="sample" markdown="1" theme="idea">

```kotlin
import kotlin.random.Random

fun writeToLog(message: String) {
    println("INFO: $message")
}

fun main() {
//sampleStart
    fun getRandomInt(): Int {
        return Random.nextInt(100).also {
            writeToLog("getRandomInt() generated value $it")
        }
    }
    
    val i = getRandomInt()
//sampleEnd
}
```

</div>

此外，当将上下文对象作为参数传递时，可以为上下文对象指定在作用域内的自定义名称。

<div class="sample" markdown="1" theme="idea">

```kotlin
import kotlin.random.Random

fun writeToLog(message: String) {
    println("INFO: $message")
}

fun main() {
//sampleStart
    fun getRandomInt(): Int {
        return Random.nextInt(100).also { value ->
            writeToLog("getRandomInt() generated value $value")
        }
    }
    
    val i = getRandomInt()
//sampleEnd
}
```

</div>

### 返回值

The scope functions differ by the result they return:
* `apply` and `also` return the context object.
* `let`, `run`, and `with` return the lambda result.

These two options let you choose the proper function depending on what you do next in your code.

#### 上下文对象

The return value of `apply` and `also` is the context object itself. Hence, they can be included into call chains as _side steps_: you can continue chaining function calls on the same object after them.  

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val numberList = mutableListOf<Double>()
    numberList.also { println("Populating the list") }
        .apply {
            add(2.71)
            add(3.14)
            add(1.0)
        }
        .also { println("Sorting the list") }
        .sort()
//sampleEnd
    println(numberList)
}
```

</div>

They also can be used in return statements of functions returning the context object.

<div class="sample" markdown="1" theme="idea">

```kotlin
import kotlin.random.Random

fun writeToLog(message: String) {
    println("INFO: $message")
}

fun main() {
//sampleStart
    fun getRandomInt(): Int {
        return Random.nextInt(100).also {
            writeToLog("getRandomInt() generated value $it")
        }
    }
    
    val i = getRandomInt()
//sampleEnd
}
```

</div>

#### Lambda 表达式结果

`let`, `run`, and `with` return the lambda result. So, you can use them when assigning the result to a variable, chaining operations on the result, and so on.

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three")
    val countEndsWithE = numbers.run { 
        add("four")
        add("five")
        count { it.endsWith("e") }
    }
    println("There are $countEndsWithE elements that end with e.")
//sampleEnd
}
```

</div>

Additionally, you can ignore the return value and use a scope function to create a temporary scope for variables. 

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three")
    with(numbers) {
        val firstItem = first()
        val lastItem = last()        
        println("First item: $firstItem, last item: $lastItem")
    }
//sampleEnd
}
```

</div>

## 几个函数

To help you choose the right scope function for your case, we'll describe them in detail and provide usage recommendations. Technically, functions are interchangeable in many cases, so the examples show the conventions that define the common usage style. 

### `let`

**The context object** is available as an argument (`it`). **The return value** is the lambda result.

`let` can be used to invoke one or more functions on results of call chains. For example, the following code prints the results of two operations on a collection:

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "four", "five")
    val resultList = numbers.map { it.length }.filter { it > 3 }
    println(resultList)    
//sampleEnd
}
```

</div>

With `let`, you can rewrite it:

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "four", "five")
    numbers.map { it.length }.filter { it > 3 }.let { 
        println(it)
        // and more function calls if needed
    } 
//sampleEnd
}
```

</div>

If the code block contains a single function with `it` as an argument, you can use the method reference (`::`) instead of the lambda:

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "four", "five")
    numbers.map { it.length }.filter { it > 3 }.let(::println)
//sampleEnd
}
```

</div>

`let` is often used for executing a code block only with non-null values. To perform actions on a non-null object, use the safe call operator `?.` on it and call `let` with the actions in its lambda.

<div class="sample" markdown="1" theme="idea">

```kotlin
fun processNonNullString(str: String) {}

fun main() {
//sampleStart
    val str: String? = "Hello"   
    //processNonNullString(str)       // compilation error: str can be null
    val length = str?.let { 
        println("let() called on $it")        
        processNonNullString(it)      // OK: 'it' is not null inside '?.let { }'
        it.length
    }
//sampleEnd
}
```

</div>

Another case for using `let` is introducing local variables with a limited scope for improving code readability. To define a new variable for the context object, provide its name as the lambda argument so that it can be used instead of the default `it`.

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    val modifiedFirstItem = numbers.first().let { firstItem ->
        println("The first item of the list is '$firstItem'")
        if (firstItem.length >= 5) firstItem else "!" + firstItem + "!"
    }.toUpperCase()
    println("First item after modifications: '$modifiedFirstItem'")
//sampleEnd
}
```

</div>

### `with`

A non-extension function: **the context object** is passed as an argument, but inside the lambda, it's available as a receiver (`this`). **The return value** is the lambda result. 

We recommend `with` for calling functions on the context object without providing the lambda result. In the code, `with` can be read as “_with this object, do the following._”

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three")
    with(numbers) {
        println("'with' is called with argument $this")
        println("It contains $size elements")
    }
//sampleEnd
}
```

</div>

Another use case for `with` is introducing a helper object whose properties or functions will be used for calculating a value.

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three")
    val firstAndLast = with(numbers) {
        "The first element is ${first()}," +
        " the last element is ${last()}"
    }
    println(firstAndLast)
//sampleEnd
}
```

</div>

### `run`

**The context object** is available as a receiver (`this`). **The return value** is the lambda result.

`run` does the same as `with` but invokes as `let` - as an extension function of the context object.

`run` is useful when your lambda contains both the object initialization and the computation of the return value.

<div class="sample" markdown="1" theme="idea">

```kotlin
class MultiportService(var url: String, var port: Int) {
    fun prepareRequest(): String = "Default request"
    fun query(request: String): String = "Result for query '$request'"
}

fun main() {
//sampleStart
    val service = MultiportService("https://example.kotlinlang.org", 80)

    val result = service.run {
        port = 8080
        query(prepareRequest() + " to port $port")
    }
    
    // the same code written with let() function:
    val letResult = service.let {
        it.port = 8080
        it.query(it.prepareRequest() + " to port ${it.port}")
    }
//sampleEnd
    println(result)
    println(letResult)
}
```

</div>

Besides calling `run` on a receiver object, you can use it as a non-extension function. Non-extension `run` lets you execute a block of several statements where an expression is required.

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val hexNumberRegex = run {
        val digits = "0-9"
        val hexDigits = "A-Fa-f"
        val sign = "+-"
        
        Regex("[$sign]?[$digits$hexDigits]+")
    }
    
    for (match in hexNumberRegex.findAll("+1234 -FFFF not-a-number")) {
        println(match.value)
    }
//sampleEnd
}
```

</div>

### `apply`

**The context object** is available as a receiver (`this`). **The return value** is the object itself.

Use `apply` for code blocks that don't return a value and mainly operate on the members of the receiver object. The common case for `apply` is the object configuration. Such calls can be read as “_apply the following assignments to the object._”

<div class="sample" markdown="1" theme="idea">

```kotlin
data class Person(var name: String, var age: Int = 0, var city: String = "")

fun main() {
//sampleStart
    val adam = Person("Adam").apply {
        age = 32
        city = "London"        
    }
//sampleEnd
}
```

</div>

Having the receiver as the return value, you can easily include `apply` into call chains for more complex processing.

### `also`

**The context object** is available as an argument (`it`). **The return value** is the object itself.

`also` is good for performing some actions that take the context object as an argument. Use `also` for additional actions that don't alter the object, such as logging or printing debug information. Usually, you can remove the calls of `also` from the call chain without breaking the program logic. 

When you see `also` in the code, you can read it as “_and also do the following_”.

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three")
    numbers
        .also { println("The list elements before adding new one: $it") }
        .add("four")
//sampleEnd
}
```

</div>

## 函数选择

To help you choose the right scope function for your purpose, we provide the table of key differences between them. 

|Function|Object reference|Return value|Is extension function|
|---|---|---|---|
|`let`|`it`|Lambda result|Yes|
|`run`|`this`|Lambda result|Yes|
|`run`|-|Lambda result|No: called without the context object|
|`with`|`this`|Lambda result|No: takes the context object as an argument.|
|`apply`|`this`|Context object|Yes|
|`also`|`it`|Context object|Yes|

Here is a short guide for choosing scope functions depending on the intended purpose:

* Executing a lambda on non-null objects: `let`
* Introducing an expression as a variable in local scope: `let`
* Object configuration: `apply`
* Object configuration and computing the result: `run`
* Running statements where an expression is required: non-extension `run`
* Additional effects: `also`
* Grouping function calls on an object: `with`

The use cases of different functions overlap, so that you can choose the functions based on the specific conventions used in your project or team.

Although the scope functions are a way of making the code more concise, avoid overusing them: it can decrease your code readability and lead to errors. Avoid nesting scope functions and be careful when chaining them: it's easy to get confused about the current context object and the value of `this` or `it`.

## `takeIf` 与 `takeUnless`

In addition to scope functions, the standard library contains the functions `takeIf` and `takeUnless`. These functions let you embed checks of the object state in call chains. 

When called on an object with a predicate provided, `takeIf` returns this object if it matches the predicate. Otherwise, it returns `null`. So, `takeIf` is a filtering function for a single object. In turn, `takeUnless` returns the object if it doesn't match the predicate and `null` if it does. The object is available as a lambda argument (`it`).

<div class="sample" markdown="1" theme="idea">

```kotlin
import kotlin.random.*

fun main() {
//sampleStart
    val number = Random.nextInt(100)

    val evenOrNull = number.takeIf { it % 2 == 0 }
    val oddOrNull = number.takeUnless { it % 2 == 0 }
    println("even: $evenOrNull, odd: $oddOrNull")
//sampleEnd
}
```

</div>

When chaining other functions after `takeIf` and `takeUnless`, don't forget to perform the null check or the safe call (`?.`) because their return value is nullable.

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    val str = "Hello"
    val caps = str.takeIf { it.isNotEmpty() }?.toUpperCase()
   //val caps = str.takeIf { it.isNotEmpty() }.toUpperCase() //compilation error
    println(caps)
//sampleEnd
}
```

</div>

`takeIf` and `takeUnless` are especially useful together with scope functions. A good case is chaining them with `let` for running a code block on objects that match the given predicate. To do this, call `takeIf` on the object and then call `let` with a safe call (`?`). For objects that don't match the predicate, `takeIf` returns `null` and `let` isn't invoked.

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    fun displaySubstringPosition(input: String, sub: String) {
        input.indexOf(sub).takeIf { it >= 0 }?.let {
            println("The substring $sub is found in $input.")
            println("Its start position is $it.")
        }
    }

    displaySubstringPosition("010000011", "11")
    displaySubstringPosition("010000011", "12")
//sampleEnd
}
```

</div>

This is how the same function looks without the standard library functions:

<div class="sample" markdown="1" theme="idea">

```kotlin
fun main() {
//sampleStart
    fun displaySubstringPosition(input: String, sub: String) {
        val index = input.indexOf(sub)
        if (index >= 0) {
            println("The substring $sub is found in $input.")
            println("Its start position is $index.")
        }
    }

    displaySubstringPosition("010000011", "11")
    displaySubstringPosition("010000011", "12")
//sampleEnd
}
```

</div>
