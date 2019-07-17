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

同样的，Map 也有这样的函数 [`mapOf()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/map-of.html) 与 [`mutableMapOf()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/mutable-map-of.html)。映射的键和值作为 `Pair` 对象传递（通常使用中缀函数 `to` 创建）。

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

对于 List，有一个接受 List 的大小与初始化函数的构造函数，该初始化函数根据索引定义元素的值。

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

要创建具体类型的集合，例如 `ArrayList` 或 `LinkedList`，可以使用这些类型的构造函数。
类似的构造函数对于 `Set` 与 `Map` 的各实现中均有提供。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
val linkedList = LinkedList<String>(listOf("one", "two", "three"))
val presizedSet = HashSet<Int>(32)
```
</div>

## 复制

要创建与现有集合具有相同元素的集合，可以使用复制操作。标准库中的集合复制操作创建了具有相同元素引用的 __浅__ 复制集合。
因此，对集合元素所做的更改会反映在其所有副本中。

在特定时刻通过集合复制函数，例如[`toList()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/to-list.html)、[`toMutableList()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/to-mutable-list.html)、[`toSet()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/to-set.html) 等等。创建了集合的快照。
结果是创建了一个具有相同元素的新集合
如果在源集合中添加或删除元素，则不会影响副本。副本也可以独立于源集合进行更改。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val sourceList = mutableListOf(1, 2, 3)
    val copyList = sourceList.toMutableList()
    val readOnlyCopyList = sourceList.toList()
    sourceList.add(4)
    println("Copy size: ${copyList.size}")   
    
    //readOnlyCopyList.add(4)             // 编译异常
    println("Read-only copy size: ${readOnlyCopyList.size}")
//sampleEnd
}
```
</div>

这些函数还可用于将集合转换为其他类型，例如根据 List 构建 Set，反之亦然。

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

或者，可以创建对同一集合实例的新引用。
因此，当通过引用更改集合实例时，更改将反映在其所有引用中。

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

集合的初始化可用于限制其可变性。例如，如果构建了一个 `MutableList` 的 `List` 引用，当你试图通过此引用修改集合的时候，编译器会抛出错误。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart 
    val sourceList = mutableListOf(1, 2, 3)
    val referenceList: List<Int> = sourceList
    //referenceList.add(4)            // 编译异常
    sourceList.add(4)
    println(referenceList) // 显示 sourceList 当前状态
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
