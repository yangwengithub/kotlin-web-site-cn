---
type: doc
layout: reference
category: "Collections"
title: "取集合的一部分"
---

# 取集合的一部分

Kotlin 标准库包含用于取集合的一部分的扩展函数。
这些函数提供了多种方法来选择结果集合的元素：显式列出其位置，指定结果大小等。

## Slice

[`slice()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/slice.html) 返回具有给定索引的 collection 元素列表。
索引可以作为[区间](ranges.html)或作为整数值的集合传递。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart    
    val numbers = listOf("one", "two", "three", "four", "five", "six")    
    println(numbers.slice(1..3))
    println(numbers.slice(0..4 step 2))
    println(numbers.slice(setOf(3, 5, 0)))    
//sampleEnd
}
```
</div>

## Take 与 drop

要从头开始获取指定数量的元素，请使用 [`take()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/take.html) 函数。
要从尾开始获取指定数量的元素，请使用 [`takeLast()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/take-last.html)。
当调用的数字大于集合的大小时，两个函数都将返回整个集合。

要从头或从尾去除给定数量的元素，请调用 [`drop()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/drop.html) 或 [`dropLast()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/drop-last.html) 函数。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.take(3))
    println(numbers.takeLast(3))
    println(numbers.drop(1))
    println(numbers.dropLast(5))
//sampleEnd
}
```
</div>

还可以使用谓词来定义要获取或去除的元素的数量。
有四个与上述功能相似的函数：

* [`takeWhile()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/take-while.html) 是带有谓词的 `take()`：它将不停获取元素直到排除与谓词匹配的首个元素。如果首个集合元素与谓词不匹配，则结果为空。
* [`takeLastWhile()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/take-last-while.html) 与 `takeLast()` 类似：它从集合末尾获取与谓词匹配的元素区间。区间的首个元素是与谓词不匹配的最后一个元素右边的元素。如果最后一个集合元素与谓词不匹配，则结果为空。
* [`dropWhile()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/drop-while.html) 与具有相同谓词的 `takeWhile()` 相反：它将首个与谓词不匹配的元素返回到末尾。
* [`dropLastWhile()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/drop-last-while.html) 与具有相同谓词的 `takeLastWhile()` 相反：它返回从开头到最后一个与谓词不匹配的元素。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.takeWhile { !it.startsWith('f') })
    println(numbers.takeLastWhile { it != "three" })
    println(numbers.dropWhile { it.length == 3 })
    println(numbers.dropLastWhile { it.contains('i') })
//sampleEnd
}
```
</div>

## Chunked

To break a collection onto parts of a given size, use the [`chunked()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/chunked.html) function.
`chunked()` takes a single argument – the size of the chunk – and returns a `List` of `List`s of the given size.
The first chunk starts from the first element and contains the `size` elements, the second chunk holds the next `size` elements, and so on.
The last chunk may have a smaller size. 

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = (0..13).toList()
    println(numbers.chunked(3))
//sampleEnd
}
```
</div>

You can also apply a transformation for the returned chunks right away.
To do this, provide the transformation as a lambda function when calling `chunked()`.
The lambda argument is a chunk of the collection. When `chunked()` is called with a transformation,
the chunks are short-living `List`s that should be consumed right in that lambda.  

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = (0..13).toList() 
    println(numbers.chunked(3) { it.sum() })  // `it` is a chunk of the original collection
//sampleEnd
}
```
</div>

## Windowed

You can retrieve all possible ranges of the collection elements of a given size.
The function for getting them is called [`windowed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/windowed.html): it returns a list of element ranges that you would see if you were looking at the collection through a sliding window of the given size.
Unlike `chunked()`,  `windowed()` returns element ranges (_windows_) starting from *each* collection element.
All the windows are returned as elements of a single `List`.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five")    
    println(numbers.windowed(3))
//sampleEnd
}
```
</div>

`windowed()` provides more flexibility with optional parameters:

* `step` defines a distance between first elements of two adjacent windows. By default the value is 1, so the result contains windows starting from all elements. If you increase the step to 2, you will receive only windows starting from odd elements: first, third, an so on.
* `partialWindows` includes windows of smaller sizes that start from the elements at the end of the collection. For example, if you request windows of three elements, you can't build them for the last two elements. Enabling `partialWindows` in this case includes two more lists of sizes 2 and 1.

Finally, you can apply a transformation to the returned ranges right away.
To do this, provide the transformation as a lambda function when calling `windowed()`.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = (1..10).toList()
    println(numbers.windowed(3, step = 2, partialWindows = true))
    println(numbers.windowed(3) { it.sum() })
//sampleEnd
}
```
</div>

To build two-element windows, there is a separate function - [`zipWithNext()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/zip-with-next.html).
It creates pairs of adjacent elements of the receiver collection.
Note that `zipWithNext()` doesn't break the collection into pairs; it creates a `Pair` for _each_ element except the last one, so its result on `[1, 2, 3, 4]` is `[[1, 2], [2, 3], [3, 4]]`, not `[[1, 2`], `[3, 4]]`.
`zipWithNext()` can be called with a transformation function as well; it should take two elements of the receiver collection as arguments.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four", "five")    
    println(numbers.zipWithNext())
    println(numbers.zipWithNext() { s1, s2 -> s1.length > s2.length})
//sampleEnd
}
```
</div>

