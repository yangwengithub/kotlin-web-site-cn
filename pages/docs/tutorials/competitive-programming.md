---
type: tutorial
layout: tutorial
title:  "竞技程序设计"
authors: Roman Elizarov，灰蓝天际（翻译）
showAuthorInfo: true
description: "本教程介绍了 Kotlin 在竞技性程序设计中的基本用法。"
---

## 先决条件

本教程适用于之前未使用 Kotlin
的竞技程序员，也适用于之前从未参与过任何竞技性程序设计活动的 Kotlin 开发人员。
本教程假定读者具有相应的编程技能。

## 竞技程序设计与 Kotlin

[竞技性程序设计](https://en.wikipedia.org/wiki/Competitive_programming)
是一项智力运动，参赛选手在严格的限制条件下编写程序精确地解决指定的<!--
-->算法问题。问题可以简单到<!--
-->任何软件开发人员都能解题、只需很少代码就能得到正确答案，也可以复杂到需要<!--
-->特殊的算法、数据结构知识以及大量实践。虽然 Kotlin 不是专为竞技性<!--
-->编程而设计的，但是它恰好适合这一领域，显著减少了<!--
-->程序员所需编写与阅读的样板代码量，这样几乎可以像动态<!--
-->脚本语言一样编写代码，同时又有静态类型语言的工具与性能支持。

关于如何搭建 Kotlin 开发环境，请参见[以 IntelliJ IDEA 入门](/docs/tutorials/getting-started.html)<!--
-->。在竞技程序设计中，通常会创建单个项目，而每个问题的答案<!--
-->写在单个源文件中。

## 简单示例：可达数问题

我们来看一个具体的示例。

[Codeforces](http://codeforces.com/)
第 555 轮第 3 次分赛已于 4 月 26 日举行，意味着它有适合任何开发者尝试的问题。
可以打开[这个链接](http://codeforces.com/contest/1157)来阅读问题。
这组问题中最简单的是<!--
-->[问题 A：可达数](http://codeforces.com/contest/1157/problem/A)。
它要求实现问题陈述中所描述的简单算法。

我们会通过创建一个新的 Kotlin 源文件来解这个问题。文件名无关紧要。`A.kt` 就挺好。
首先，我们需要实现问题陈述中指定的函数，如下：

> 我们以这样的方式来表示函数 f(x)：将 x 加 1，然后，如果得到的数至少以一个零结尾，
就去掉这个零。

Kotlin 是实用且不拘一格的语言，既支持命令式也支持函数式编程风格，
不强迫开发人员选择任何一种特定风格。可以按函数式风格实现函数 `f`，使用像
[尾递归](/docs/reference/functions.html#尾递归函数)这样的 Kotlin 特性：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
tailrec fun removeZeroes(x: Int): Int =
    if (x % 10 == 0) removeZeroes(x / 10) else x
    
fun f(x: Int) = removeZeroes(x + 1)
```

</div>

也可以编写函数 `f` 的命令式实现，使用传统的
[while 循环](/docs/reference/control-flow.html) 与可变变量（在 Kotlin 中用
[var](/docs/reference/basic-syntax.html#定义变量) 表示）：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun f(x: Int): Int {
    var cur = x + 1
    while (cur % 10 == 0) cur /= 10
    return cur
}
```

</div>

由于普遍使用类型推断，在 Kotlin 中很多地方的类型都是可选的，不过每个声明仍然具有<!--
-->编译期已知的明确定义的静态类型。

现在就只剩编写读取输入的主函数并实现问题<!--
-->陈述所要求算法的其余部分——计算在对标准输入所给出的初始数
`n` 重复应用函数 `f` 时所产生的不同整数的个数。

默认情况下，Kotlin 在 JVM 上运行，可以直接访问丰富且高效的集合库，其中包含<!--
-->通用的集合与数据结构，如动态大小的数组（`ArrayList`）、
基于哈希的 map 与 set（`HashMap`/`HashSet`）、基于树的map 与 set（`TreeMap`/`TreeSet`）等。
使用整数哈希 set 来跟踪应用函数 `f` 时已达到的值，
该问题解法的一个简单命令式版本可以这样编写：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun main() {
    var n = readLine()!!.toInt() // 读取输入的整数
    val reached = HashSet<Int>() // 可变的哈希 set
    while (reached.add(n)) n = f(n) // 迭代函数 f
    println(reached.size) // 输出答案
}
```

</div>

请注意在
[readLine()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.io/read-line.html)
函数调用之后使用了 Kotlin 的[空断言操作符](/docs/reference/null-safety.html#-操作符) `!!`。
Kotlin 的 `readLine()` 函数定义成了返回<!--
-->[可空类型](/docs/reference/null-safety.html#可空与非空类型)
`String?`，并且会在输入结束时返回 `null`，这样明确迫使开发人员处理<!--
-->输入缺失的情况。
 
在竞技程序设计中无需处理输入格式错误的情况。
竞技程序设计中的输入格式向来都是精确指定的，并且实际输入不能偏离<!--
-->问题陈述中的输入规范。这基本上就是空断言操作符 `!!` 的行为——
它断言输入的字符串存在，如不存在则抛出异常。同样，如果输入不是整数，那么
[String.toInt()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/to-int.html)
函数会抛出异常。

所有在线竞技程序设计活动都允许使用预编写代码，因此可以定义自己的<!--
-->面向竞技性编程的工具函数库，以使实际解题代码更易<!--
-->于读写。然后，可以使用该代码作为解题模板。例如，可以定义<!--
-->以下辅助函数来读取竞技程序设计中的输入：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
private fun readLn() = readLine()!!
private fun readInt() = readLn().toInt()
// 用于在解题中会用到的其他类型的类似声明等
```

</div>

请注意这里使用了 `private`（私有）[可见修饰符](/docs/reference/visibility-modifiers.html)。
虽然可见性修饰符的概念与竞技程序设计并无瓜葛，
但是它让你能够将<!--
-->基于相同模板的多个解题文件放在同一包中，而不会出现公有声明冲突的报错。

## 函数式操作符示例：长数问题

For more complicated problems, Kotlin's extensive library of functional operations on collections comes in handy to 
minimize the boilerplate and turn the code into a linear top-to-bottom and left-to-right fluent data transformation 
pipeline. For example, the 
[Problem B: Long Number](http://codeforces.com/contest/1157/problem/B) problem 
takes a simple greedy algorithm to implement and it can be written using this style without a single mutable variable:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
fun main() {
    // read input
    val n = readLine()!!.toInt()
    val s = readLine()!!
    val fl = readLine()!!.split(" ").map { it.toInt() }
    // define local function f
    fun f(c: Char) = '0' + fl[c - '1']
    // greedily find first and last indices
    val i = s.indexOfFirst { c -> f(c) > c }
        .takeIf { it >= 0 } ?: s.length
    val j = s.withIndex().indexOfFirst { (j, c) -> j > i && f(c) < c }
        .takeIf { it >= 0 } ?: s.length
    // compose and write the answer
    val ans =
        s.substring(0, i) +
        s.substring(i, j).map { c -> f(c) }.joinToString("") +
        s.substring(j)
    println(ans)
}
```

</div>

In this dense code, in addition to collection transformations, you can see such handy Kotlin features as local functions,
and [elvis operator](/docs/reference/null-safety.html#elvis-operator) `?:`
that allow to express the 
[idioms](/docs/reference/idioms.html) like "take value if it is positive or else use length" with concise and readable 
expression like `.takeIf { it >= 0 } ?: s.length`, yet it is perfectly fine with Kotlin to create additional mutable
variables and express the same code in imperative style, too.

To make reading input in the competitive programming tasks like this more concise, 
you can have the following list of helper input-reading functions:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
private fun readLn() = readLine()!! // string line
private fun readInt() = readLn().toInt() // single int
private fun readStrings() = readLn().split(" ") // list of strings
private fun readInts() = readStrings().map { it.toInt() } // list of ints
```

</div>

With these helpers, the part of code for reading input becomes simpler, closely following the input 
specification in the problem statement line by line:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
    // read input
    val n = readInt()
    val s = readLn()
    val fl = readInts()
```

</div>

Note, that in competitive programming it is customary to give variables shorter names than it is 
typical in industrial programming practice, since the code is to be written just once and need not be supported thereafter. 
However, these names are usually still mnemonic &mdash; `a` for arrays,
`i`, `j`, etc for indices, `r`, and `c` for row and column numbers in tables, `x` and `y` for coordinates, etc.
It is easier to keep the same names for input data as it is given in the problem statement. 
However, more complex problems require more code to solve and subsequently variable and function names tend to 
become longer and more self-explanatory. 

## 更多提示和技巧

Competitive programming problems often have input like this:

> The first line of the input contains two integers `n` and `k`

In Kotlin this line can be concisely parsed with the following statement using
[destructuring declaration](/docs/reference/multi-declarations.html#destructuring-declarations) 
from a list of integers:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val (n, k) = readInts() 
```

</div>

It might be temping to use JVM's `java.util.Scanner` class to parse less structured 
input formats. Kotlin is designed to interoperate well with JVM libraries, so that their use feels quite
natural in Kotlin. However, beware that `java.util.Scanner` is extremely slow. So slow, in fact, that parsing
10<sup>5</sup> or more integers with it might not fit into a typical 2 second time-limit, which a simple Kotlin's 
`split(" ").map { it.toInt() }` would handle. 

Writing output in Kotlin is usually straightforward with 
[println(...)](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.io/println.html)
calls and using Kotlin's 
[string templates](/docs/reference/basic-types.html#string-templates). However, care must be taken when output 
contains on order of 10<sup>5</sup> lines or more. Issuing so many `println` calls is too slow, since the output 
in Kotlin is automatically flushed after each line. 
A faster way to write many lines from an array or a list is using
[joinToString()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/join-to-string.html) function
with `"\n"` as separator, like this:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
println(a.joinToString("\n")) // each element of array/list of a separate line
```

</div>

## 学习 Kotlin

Kotlin is designed to be easy to learn for people who already know Java.
A quick overview of differences is given on [the official comparison page](/docs/reference/comparison-to-java.html). 
A short introduction to the basic syntax of Kotlin language for software developers can be found directly in the
reference section of the web site starting from [basic syntax](/docs/reference/basic-syntax.html). 

IDEA has built-in 
[Java-to-Kotlin converter](https://www.jetbrains.com/help/idea/converting-a-java-file-to-kotlin-file.html). 
It can be used by people familiar with Java to learn the corresponding Kotlin syntactic constructions, but it
is not perfect and it is still worth familiarizing yourself with Kotlin and learning the 
[Kotlin idioms](/docs/reference/idioms.html).

A great resource to study Kotlin syntax and API of the Kotlin standard library are
[Kotlin Koans](/docs/tutorials/koans.html).

