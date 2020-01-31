---
type: doc
layout: reference
category: "Collections"
title: "集合转换操作"
---

# 集合转换

Kotlin 标准库为集合 _转换_ 提供了一组扩展函数。
这些函数根据提供的转换规则从现有集合中构建新集合。
在此页面中，我们将概述可用的集合转换函数。

## 映射

_映射_ 转换从另一个集合的元素上的函数结果创建一个集合。
基本的映射函数是 [`map()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map.html)。
它将给定的 lambda 函数应用于每个后续元素，并返回 lambda 结果列表。
结果的顺序与元素的原始顺序相同。
如需应用还要用到元素索引作为参数的转换，请使用 [`mapIndexed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-indexed.html)。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = setOf(1, 2, 3)
    println(numbers.map { it * 3 })
    println(numbers.mapIndexed { idx, value -> value * idx })
//sampleEnd
}
```
</div>

如果转换在某些元素上产生 `null` 值，则可以通过调用 [`mapNotNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-not-null.html) 函数取代 `map()` 或 [`mapIndexedNotNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-indexed-not-null.html) 取代 `mapIndexed()` 来从结果集中过滤掉 `null` 值。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = setOf(1, 2, 3)
    println(numbers.mapNotNull { if ( it == 2) null else it * 3 })
    println(numbers.mapIndexedNotNull { idx, value -> if (idx == 0) null else value * idx })
//sampleEnd
}
```
</div>

映射转换时，有两个选择：转换键，使值保持不变，反之亦然。
要将指定转换应用于键，请使用 [`mapKeys()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-keys.html)；反过来，[`mapValues()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-values.html) 转换值。
这两个函数都使用将映射条目作为参数的转换，因此可以操作其键与值。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    println(numbersMap.mapKeys { it.key.toUpperCase() })
    println(numbersMap.mapValues { it.value + it.key.length })
//sampleEnd
}
```
</div>

## 双路合并

_双路合并_ 转换是根据两个集合中具有相同位置的元素构建配对。
在 Kotlin 标准库中，这是通过 [`zip()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/zip.html) 扩展函数完成的。
在一个集合（或数组）上调用以另一个集合（或数组）作为参数，`zip()` 返回 `Pair` 对象的列表（`List`）。
接收者集合的元素是这些配对中的第一个元素。
如果集合的大小不同，则 `zip()` 的结果为较小集合的大小；结果中不包含较大集合的后续元素。
`zip()` 也可以中缀形式调用 `a zip b` 。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val colors = listOf("red", "brown", "grey")
    val animals = listOf("fox", "bear", "wolf")
    println(colors zip animals)

    val twoAnimals = listOf("fox", "bear")
    println(colors.zip(twoAnimals))
//sampleEnd
}
```
</div>

也可以使用带有两个参数的转换函数来调用 `zip()`：接收者元素和参数元素。
在这种情况下，结果 `List` 包含在具有相同位置的接收者对和参数元素对上调用的转换函数的返回值。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val colors = listOf("red", "brown", "grey")
    val animals = listOf("fox", "bear", "wolf")
    
    println(colors.zip(animals) { color, animal -> "The ${animal.capitalize()} is $color"})
//sampleEnd
}
```
</div>

当拥有 `Pair` 的 `List` 时，可以进行反向转换 _unzipping_——从这些键值对中构建两个列表：

* 第一个列表包含原始列表中每个 `Pair` 的键。
* 第二个列表包含原始列表中每个 `Pair` 的值。

要分割键值对列表，请调用 [`unzip()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/unzip.html)。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numberPairs = listOf("one" to 1, "two" to 2, "three" to 3, "four" to 4)
    println(numberPairs.unzip())
//sampleEnd
}
```
</div>

## 关联

_关联_ 转换允许从集合元素和与其关联的某些值构建 Map。
在不同的关联类型中，元素可以是关联 Map 中的键或值。

基本的关联函数 [`associateWith()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/associate-with.html) 创建一个 `Map`，其中原始集合的元素是键，并通过给定的转换函数从中产生值。
如果两个元素相等，则仅最后一个保留在 Map 中。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    println(numbers.associateWith { it.length })
//sampleEnd
}
```
</div>

为了使用集合元素作为值来构建 Map，有一个函数 [`associateBy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/associate-by.html)。
它需要一个函数，该函数根据元素的值返回键。如果两个元素相等，则仅最后一个保留在 Map 中。
还可以使用值转换函数来调用 `associateBy()`。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")

    println(numbers.associateBy { it.first().toUpperCase() })
    println(numbers.associateBy(keySelector = { it.first().toUpperCase() }, valueTransform = { it.length }))
//sampleEnd
}
```
</div>

另一种构建 Map 的方法是使用函数 [`associate()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/associate.html)，其中 Map 键和值都是通过集合元素生成的。
它需要一个 lambda 函数，该函数返回 `Pair`：键和相应 Map 条目的值。

请注意，`associate()` 会生成临时的 `Pair` 对象，这可能会影响性能。
因此，当性能不是很关键或比其他选项更可取时，应使用 `associate()`。

后者的一个示例：从一个元素一起生成键和相应的值。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
data class FullName (val firstName: String, val lastName: String)

fun parseFullName(fullName: String): FullName {
    val nameParts = fullName.split(" ")
    if (nameParts.size == 2) {
        return FullName(nameParts[0], nameParts[1])
    } else throw Exception("Wrong name format")
}

//sampleStart
    val names = listOf("Alice Adams", "Brian Brown", "Clara Campbell")
    println(names.associate { name -> parseFullName(name).let { it.lastName to it.firstName } })  
//sampleEnd
}
```
</div>

此时，首先在一个元素上调用一个转换函数，然后根据该函数结果的属性建立 Pair。


## 打平

If you operate nested collections, you may find the standard library functions that provide flat access to nested collection elements useful.

The first function is [`flatten()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/flatten.html). You can call it on a collection of collections, for example, a `List` of `Set`s.
The function returns a single `List` of all the elements of the nested collections.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numberSets = listOf(setOf(1, 2, 3), setOf(4, 5, 6), setOf(1, 2))
    println(numberSets.flatten())
//sampleEnd
}
```
</div>

Another function – [`flatMap()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/flat-map.html) provides a flexible way to process nested collections.
It takes a function that maps a collection element to another collection.
As a result, `flatMap()` returns a single list of its return values on all the elements.
So, `flatMap()` behaves as a subsequent call of `map()` (with a collection as a mapping result) and `flatten()`.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
data class StringContainer(val values: List<String>)

fun main() {
//sampleStart
    val containers = listOf(
        StringContainer(listOf("one", "two", "three")),
        StringContainer(listOf("four", "five", "six")),
        StringContainer(listOf("seven", "eight"))
    )
    println(containers.flatMap { it.values })
//sampleEnd
}
```
</div>

## 字符串表示

If you need to retrieve the collection content in a readable format, use functions that transform the collections to strings: [`joinToString()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/join-to-string.html) and [`joinTo()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/join-to.html).

`joinToString()` builds a single `String` from the collection elements based on the provided arguments.
`joinTo()` does the same but appends the result to the given [`Appendable`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.text/-appendable/index.html) object.

When called with the default arguments, the functions return the result similar to calling `toString()` on the collection: a `String` of elements' string representations separated by commas with spaces. 

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    
    println(numbers)         
    println(numbers.joinToString())
    
    val listString = StringBuffer("The list of numbers: ")
    numbers.joinTo(listString)
    println(listString)
//sampleEnd
}
```
</div>

To build a custom string representation, you can specify its parameters in function arguments `separator`, `prefix`, and `postfix`.
The resulting string will start with the `prefix` and end with the `postfix`. The `separator` will come after each element except the last.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")    
    println(numbers.joinToString(separator = " | ", prefix = "start: ", postfix = ": end"))
//sampleEnd
}
```
</div>

For bigger collections, you may want to specify the `limit` – a number of elements that will be included into result.
If the collection size exceeds the `limit`, all the other elements will be replaced with a single value of the `truncated` argument.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = (1..100).toList()
    println(numbers.joinToString(limit = 10, truncated = "<...>"))
//sampleEnd
}
```
</div>

Finally, to customize the representation of elements themselves, provide the `transform` function. 

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = listOf("one", "two", "three", "four")
    println(numbers.joinToString { "Element: ${it.toUpperCase()}"})
//sampleEnd
}
```
</div>
