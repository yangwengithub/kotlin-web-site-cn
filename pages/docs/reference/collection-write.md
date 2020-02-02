---
type: doc
layout: reference
category: "Collections"
title: "集合写操作"
---

# 集合写操作

[可变集合](collections-overview.html#集合类型)支持更改集合内容的操作，例如添加或删除元素。
在次页面上，我们将描述实现 `MutableCollection` 的所有写操作。
有关 `List` 与 `Map` 可用的更多特定操作，请分别参见 [List 相关操作](list-operations.html)与 [Map 相关操作](map-operations.html)。

## 添加元素

要将单个元素添加到列表或集合，请使用 [`add()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/add.html) 函数。指定的对象将添加到集合的末尾。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4)
    numbers.add(5)
    println(numbers)
//sampleEnd
}
```
</div>

[`addAll()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/add-all.html) 将参数对象的每个元素添加到列表或集合中。参数可以是 `Iterable`、`Sequence` 或 `Array`。
接收者的类型和参数可能不同，例如，你可以将所有内容从 `Set` 添加到 `List`。

当在列表上调用时，`addAll()` 会按照在参数中出现的顺序添加各个新元素。
你也可以调用 `addAll()` 时指定一个元素位置作为第一参数。
参数集合的第一个元素会被插入到这个位置。
其他元素将跟随在它后面，将接收者元素移到末尾。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 5, 6)
    numbers.addAll(arrayOf(7, 8))
    println(numbers)
    numbers.addAll(2, setOf(3, 4))
    println(numbers)
//sampleEnd
}
```
</div>

你还可以使用 [`plus` 运算符](collection-plus-minus.html) - [`plusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/plus-assign.html) (`+=`) 添加元素。
 当应用于可变集合时，`+=` 将第二个操作数(一个元素或另一个集合)追加到集合的末尾。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two")
    numbers += "three"
    println(numbers)
    numbers += listOf("four", "five")    
    println(numbers)
//sampleEnd
}
```
</div>

## 删除元素

若要从可变集合中移除元素，请使用 [`remove()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/remove.html) 函数。
`remove()` 接受元素值，并删除该值的一个匹配项。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4, 3)
    numbers.remove(3)                    // removes the first `3`
    println(numbers)
    numbers.remove(5)                    // removes nothing
    println(numbers)
//sampleEnd
}
```
</div>

For removing multiple elements at once, there are the following functions :

* [`removeAll()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/remove-all.html) removes all elements that are present in the argument collection.
   Alternatively, you can call it with a predicate as an argument; in this case the function removes all elements for which the predicate yields `true`.
* [`retainAll()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/retain-all.html) is the opposite of `removeAll()`: it removes all elements except the ones from the argument collection.
   When used with a predicate, it leaves only elements that match it.
* [`clear()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/clear.html) removes all elements from a list and leaves it empty.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4)
    println(numbers)
    numbers.retainAll { it >= 3 }
    println(numbers)
    numbers.clear()
    println(numbers)

    val numbersSet = mutableSetOf("one", "two", "three", "four")
    numbersSet.removeAll(setOf("one", "two"))
    println(numbersSet)
//sampleEnd
}
```
</div>

Another way to remove elements from a collection is with the [`minusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/minus-assign.html) (`-=`) operator – the in-place version of [`minus`](collection-plus-minus.html).
The second argument can be a single instance of the element type or another collection.
With a single element on the right-hand side, `-=` removes the _first_ occurrence of it.
In turn, if it's a collection, _all_ occurrences of its elements are removed.
For example, if a list contains duplicate elements, they are removed at once.
The second operand can contain elements that are not present in the collection. Such elements don't affect the operation execution.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "three", "four")
    numbers -= "three"
    println(numbers)
    numbers -= listOf("four", "five")    
    //numbers -= listOf("four")    // does the same as above
    println(numbers)    
//sampleEnd
}
```
</div>

## 更新元素

Lists and sets also provide operations for updating elements.
They are described in [List Specific Operations](list-operations.html) and [Map Specific Operations](map-operations.html).
For sets, updating doesn't make sense since it's actually removing an element and adding another one.

