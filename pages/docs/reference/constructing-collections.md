---
type: doc
layout: reference
category: "Collections"
title: "构造集合"
---

# 构造集合

## 由元素构造

创建集合的最常用方法是使用标准库函数 [`listOf<T>()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/list-of.html)、[`setOf<T>()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/set-of.html)、[`mutableListOf<T>()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/mutable-list-of.html)、[`mutableSetOf<T>()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/mutable-set-of.html)。
如果以逗号分隔的集合元素列表作为参数，编译器会自动检测元素类型。创建空集合时，须明确指定类型。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val numbersSet = setOf("one", "two", "three", "four")
val emptySet = mutableSetOf<String>()
```
</div>

同样的，Map 也有这样的函数 [`mapOf()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-of.html) 与 [`mutableMapOf()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/mutable-map-of.html)。映射的键和值作为 `Pair`对象传递（通常在函数中插入 `to` 创建）。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 1)
```
</div>

注意，`to` 符号创建了一个短时存活的 `Pair` 对象，因此建议仅在性能不重要时才使用它。
为避免过多的内存使用，请使用其他方法。例如，可以创建可写 Map 并使用写入操作填充它。
[`apply()`](scope-functions.html#apply) 函数可以帮助保持初始化流畅。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val numbersMap = mutableMapOf<String, String>().apply { this["one"] = "1"; this["two"] = "2" }
```
</div>

## 空集合

还有用于创建没有任何元素的集合的函数：[`emptyList()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/empty-list.html)、[`emptySet()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/empty-set.html) 与 [`emptyMap()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/empty-map.html)。
创建空集合时，应指定集合将包含的元素类型。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val empty = emptyList<String>()
```
</div>

## list 的初始化函数

对于 List，有一个构造函数接受 List 的大小并初始化函数，它根据索引定义元素值。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val doubled = List(3, { it * 2 })  // 如果你想操作这个集合，应使用 MutableList
    println(doubled)
//sampleEnd
}
```
</div>

## 具体类型构造函数

要创建具体的类型集合，例如 `ArrayList` 或 `LinkedList`，可以使用这些类型的构造函数。
类似的构造函数在 `Set` 与 `Map` 中均有实现。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val linkedList = LinkedList<String>(listOf("one", "two", "three"))
val presizedSet = HashSet<Int>(32)
```
</div>

## 复制

To create a collection with the same elements as an existing collection, you can use copying operations. Collection copying operations from the standard library create _shallow_ copy collections with references to the same elements.
Thus, a change made to a collection element reflects in all its copies. 

Collection copying functions, such as [`toList()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/to-list.html), [`toMutableList()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/to-mutable-list.html), [`toSet()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/to-set.html) and others, create a snapshot of a collection at a specific moment.
Their result is a new collection of the same elements.
If you add or remove elements from the original collection, this won't affect the copies. Copies may be changed independently of the source as well.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val sourceList = mutableListOf(1, 2, 3)
    val copyList = sourceList.toMutableList()
    val readOnlyCopyList = sourceList.toList()
    sourceList.add(4)
    println("Copy size: ${copyList.size}")   
    
    //readOnlyCopyList.add(4)             // compilation error
    println("Read-only copy size: ${readOnlyCopyList.size}")
//sampleEnd
}
```
</div>

These functions can also be used for converting collections to other types, for example, build a set from a list or vice versa.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val sourceList = mutableListOf(1, 2, 3)    
    val copySet = sourceList.toMutableSet()
    copySet.add(3)
    copySet.add(4)    
    println(copySet)
//sampleEnd
}
```
</div>

Alternatively, you can create new references to the same collection instance. New references are created when you initialize a collection variable with an existing collection.
So, when the collection instance is altered through a reference, the changes are reflected in all its references.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val sourceList = mutableListOf(1, 2, 3)
    val referenceList = sourceList
    referenceList.add(4)
    println("Source size: ${sourceList.size}")
//sampleEnd
}
```
</div>

Collection initialization can be used for restricting mutability. For example, if you create a `List` reference to a `MutableList`, the compiler will produce errors if you try to modify the collection through this reference.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart 
    val sourceList = mutableListOf(1, 2, 3)
    val referenceList: List<Int> = sourceList
    //referenceList.add(4)            //compilation error
    sourceList.add(4)
    println(referenceList) // shows the current state of sourceList
//sampleEnd
}
```
</div>

## 调用其他集合的函数

Collections can be created in result of various operations on other collections. For example, [filtering](collection-filtering.html) a list creates a new list of elements that match the filter:

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart 
    val numbers = listOf("one", "two", "three", "four")  
    val longerThan3 = numbers.filter { it.length > 3 }
    println(longerThan3)
//sampleEnd
}
```
</div>

[Mapping](collection-transformations.html#映射) produces a list of a transformation results:

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

[Association](collection-transformations.html#关联) produces maps:

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

For more information about operations on collections in Kotlin, see [Collection Operations Overview](collection-operations.html).
