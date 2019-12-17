---
type: doc
layout: reference
category: "Collections"
title: "序列"
---

# 序列

除了集合之外，Kotlin 标准库还包含另一种容器类型——_序列_（[`Sequence<T>`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/-sequence/index.html)）。
序列提供与 [`Iterable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-iterable/index.html) 相同的函数，但实现另一种方法来进行多步骤集合处理。

当 `Iterable` 的处理包含多个步骤时，它们会优先执行：每个处理步骤完成并返回其结果——中间集合。
在此集合上执行以下步骤。反过来，序列的多步处理在可能的情况下会延迟执行：仅当请求整个处理链的结果时才进行实际计算。

操作执行的顺序也不同：`Sequence` 对每个元素逐个执行所有处理步骤。
反过来，`Iterable` 完成整个集合的每个步骤，然后进行下一步。

因此，这些序列可避免生成中间步骤的结果，从而提高了整个集合处理链的性能。
但是，序列的延迟性质增加了一些开销，这些开销在处理较小的集合或进行更简单的计算时可能很重要。
因此，应该同时考虑使用 `Sequence` 与 `Iterable`，并确定在哪种情况更适合。

## 构造

### 由元素
要创建一个序列，请调用 [`sequenceOf()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/sequence-of.html) 函数，列出元素作为其参数。



```kotlin
val numbersSequence = sequenceOf("four", "three", "two", "one")
```


### 由 `Iterable`
如果已经有一个 `Iterable` 对象（例如 `List` 或 `Set`），则可以通过调用 [`asSequence()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/as-sequence.html) 从而创建一个序列。



```kotlin
val numbers = listOf("one", "two", "three", "four")
val numbersSequence = numbers.asSequence()

```


### 由函数
创建序列的另一种方法是通过使用计算其元素的函数来构建序列。
要基于函数构建序列，请以该函数作为参数调用 [`generateSequence()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/generate-sequence.html)。
（可选）可以将第一个元素指定为显式值或函数调用的结果。
当提供的函数返回 `null` 时，序列生成停止。因此，以下示例中的序列是无限的。



```kotlin
fun main() {
//sampleStart
    val oddNumbers = generateSequence(1) { it + 2 } // `it` 是上一个元素
    println(oddNumbers.take(5).toList())
    //println(oddNumbers.count())     // 错误：此序列是无限的。
//sampleEnd
}
```


要使用 `generateSequence()` 创建有限序列，请提供一个函数，该函数在需要的最后一个元素之后返回 `null`。



```kotlin
fun main() {
//sampleStart
    val oddNumbersLessThan10 = generateSequence(1) { if (it < 10) it + 2 else null }
    println(oddNumbersLessThan10.count())
//sampleEnd
}
```


### 由组块

最后，有一个函数可以逐个或按任意大小的组块生成序列元素——[`sequence()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/sequence.html) 函数。
此函数采用一个 lambda 表达式，其中包含 [`yield()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/-sequence-scope/yield.html) 与 [`yieldAll()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/-sequence-scope/yield-all.html) 函数的调用。
它们将一个元素返回给序列使用者，并暂停 `sequence()` 的执行，直到使用者请求下一个元素。
`yield()` 使用单个元素作为参数；`yieldAll()` 中可以采用 `Iterable` 对象、`Iterable` 或其他 `Sequence`。`yieldAll()` 的 `Sequence` 参数可以是无限的。 总之，这样的调用必须是最后一个：之后的所有调用将永远不会执行。



```kotlin
fun main() {
//sampleStart
    val oddNumbers = sequence {
        yield(1)
        yieldAll(listOf(3, 5))
        yieldAll(generateSequence(7) { it + 2 })
    }
    println(oddNumbers.take(5).toList())
//sampleEnd
}
```


## 序列操作

The sequence operations can be classified into the following groups regarding their state requirements:

* _Stateless_ operations require no state and process each element independently, for example, [`map()`](collection-transformations.html#映射) or [`filter()`](collection-filtering.html).
   Stateless operations can also require a small constant amount of state to process an element, for example, [`take()` or `drop()`](collection-parts.html).
* _Stateful_ operations require a significant amount of state, usually proportional to the number of elements in a sequence.

If a sequence operation returns another sequence, which is produced lazily, it's called _intermediate_.
Otherwise, the operation is _terminal_. Examples of terminal operations are [`toList()`](constructing-collections.html#复制) or [`sum()`](collection-aggregate.html). Sequence elements can be retrieved only with terminal operations.

Sequences can be iterated multiple times; however some sequence implementations might constrain themselves to be iterated only once. That is mentioned specifically in their documentation.

## 序列处理示例

Let's take a look at the difference between `Iterable` and `Sequence` with an example. 

### Iterable

Assume that you have a list of words. The code below filters the words longer than three characters and prints the lengths of first four such words.



```kotlin
fun main() {    
//sampleStart
    val words = "The quick brown fox jumps over the lazy dog".split(" ")
    val lengthsList = words.filter { println("filter: $it"); it.length > 3 }
        .map { println("length: ${it.length}"); it.length }
        .take(4)

    println("Lengths of first 4 words longer than 3 chars:")
    println(lengthsList)
//sampleEnd
}
```


When you run this code, you'll see that the `filter()` and `map()` functions are executed in the same order as they appear in the code.
First, you see `filter:` for all elements, then `length:` for the elements left after filtering, and then the output of the two last lines. 
This is how the list processing goes:

![List processing](/assets/images/reference/sequences/list-processing.png)

### Sequence

Now let's write the same with sequences:



```kotlin
fun main() {
//sampleStart
    val words = "The quick brown fox jumps over the lazy dog".split(" ")
    //convert the List to a Sequence
    val wordsSequence = words.asSequence()

    val lengthsSequence = wordsSequence.filter { println("filter: $it"); it.length > 3 }
        .map { println("length: ${it.length}"); it.length }
        .take(4)

    println("Lengths of first 4 words longer than 3 chars")
    // terminal operation: obtaining the result as a List
    println(lengthsSequence.toList())
//sampleEnd
}
```


The output of this code shows that the `filter()` and `map()` functions are called only when building the result list.
So, you first see the line of text `“Lengths of..”` and then the sequence processing starts.
Note that for elements left after filtering, the map executes before filtering the next element.
When the result size reaches 4, the processing stops because it's the largest possible size that `take(4)` can return.

The sequence processing goes like this:

![Sequences processing](/assets/images/reference/sequences/sequence-processing.png)

In this example, the sequence processing takes 18 steps instead of 23 steps for doing the same with lists.
