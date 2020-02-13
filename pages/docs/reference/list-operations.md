---
type: doc
layout: reference
category: "Collections"
title: "List 相关操作"
---

# List 相关操作

[`List`](collections-overview.html#list) 是 Kotlin 标准库中最受欢迎的集合类型。对列表元素的索引访问为 List 提供了一组强大的操作。

## 按索引取元素

List 支持按索引取元素的所有常用操作： `elementAt()` 、 `first()` 、 `last()` 与[取单个元素](collection-elements.html)中列出的其他操作。
List 的特点是能通过索引访问特定元素，因此读取元素的最简单方法是按索引检索它。
这是通过 [`get()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/get.html) 函数或简写语法 `[index]` 来传递索引参数完成的。

如果 List 长度小于指定的索引，则抛出异常。
另外，还有两个函数能避免此类异常：

* [`getOrElse()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-or-else.html) 提供用于计算默认值的函数，如果集合中不存在索引，则返回默认值。
* [`getOrNull()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-or-null.html) 返回 `null` 作为默认值。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(1, 2, 3, 4)
    println(numbers.get(0))
    println(numbers[0])
    //numbers.get(5)                         // exception!
    println(numbers.getOrNull(5))             // null
    println(numbers.getOrElse(5, {it}))        // 5
//sampleEnd
}

```
</div>

## 取列表的一部分

除了[取集合的一部分](collection-parts.html)中常用的操作， List 还提供 [`subList()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-list/sub-list.html) 该函数将指定元素范围的视图作为列表返回。
因此，如果原始集合的元素发生变化，则它在先前创建的子列表中也会发生变化，反之亦然。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = (0..13).toList()
    println(numbers.subList(3, 6))
//sampleEnd
}

```
</div>

## 查找元素位置

### 线性查找

在任何列表中，都可以使用 [`indexOf()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index-of.html) 或 [`lastIndexOf()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/last-index-of.html) 函数找到元素的位置。
它们返回与列表中给定参数相等的元素的第一个或最后一个位置。
如果没有这样的元素，则两个函数均返回 `-1`。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = listOf(1, 2, 3, 4, 2, 5)
    println(numbers.indexOf(2))
    println(numbers.lastIndexOf(2))
//sampleEnd
}

```
</div>

还有一对函数接受谓词并搜索与之匹配的元素：

* [`indexOfFirst()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index-of-first.html) 返回与谓词匹配的*第一个元素的索引*，如果没有此类元素，则返回 `-1`。
* [`indexOfLast()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/index-of-last.html) 返回与谓词匹配的*最后一个元素的索引*，如果没有此类元素，则返回 `-1`。

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4)
    println(numbers.indexOfFirst { it > 2})
    println(numbers.indexOfLast { it % 2 == 1})
//sampleEnd
}

```
</div>

### 在有序列表中二分查找

There is one more way to search elements in lists – [binary search](https://en.wikipedia.org/wiki/Binary_search_algorithm).
It works significantly faster than other built-in search functions but *requires the list to be [sorted](collection-ordering.html)* in ascending order according to a certain ordering: natural or another one provided in the function parameter.
Otherwise, the result is undefined. 

To search an element in a sorted list, call the [`binarySearch()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/binary-search.html) function passing the value as an argument.
If such an element exists, the function returns its index; otherwise, it returns `(-insertionPoint - 1)` where `insertionPoint` is the index where this element should be inserted so that the list remains sorted.
If there is more than one element with the given value, the search can return any of their indices.

You can also specify an index range to search in: in this case, the function searches only between two provided indices.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "four")
    numbers.sort()
    println(numbers)
    println(numbers.binarySearch("two"))  // 3
    println(numbers.binarySearch("z")) // -5
    println(numbers.binarySearch("two", 0, 2))  // -3
//sampleEnd
}

```
</div>

#### Comparator 二分搜索

When list elements aren't `Comparable`, you should provide a [`Comparator`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-comparator.html) to use in the binary search.
The list must be sorted in ascending order according to this `Comparator`. Let's have a look at an example:

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
data class Product(val name: String, val price: Double)

fun main() {
//sampleStart
    val productList = listOf(
        Product("WebStorm", 49.0),
        Product("AppCode", 99.0),
        Product("DotTrace", 129.0),
        Product("ReSharper", 149.0))

    println(productList.binarySearch(Product("AppCode", 99.0), compareBy<Product> { it.price }.thenBy { it.name }))
//sampleEnd
}

```
</div>

Here's a list of `Product` instances that aren't `Comparable` and a `Comparator` that defines the order: product `p1` precedes product `p2` if `p1`'s  price is less than `p2`'s price.
So, having a list sorted ascending according to this order, we use `binarySearch()` to find the index of the specified `Product`.

Custom comparators are also handy when a list uses an order different from natural one, for example, a case-insensitive order for `String` elements. 

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val colors = listOf("Blue", "green", "ORANGE", "Red", "yellow")
    println(colors.binarySearch("RED", String.CASE_INSENSITIVE_ORDER)) // 3
//sampleEnd
}

```
</div>

#### 比较函数二分搜索

Binary search with _comparison_ function lets you find elements without providing explicit search values.
Instead, it takes a comparison function mapping elements to `Int` values and searches for the element where the function returns zero.
The list must be sorted in the ascending order according to the provided function; in other words, the return values of comparison must grow from one list element to the next one.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
import kotlin.math.sign
//sampleStart
data class Product(val name: String, val price: Double)

fun priceComparison(product: Product, price: Double) = sign(product.price - price).toInt()

fun main() {
    val productList = listOf(
        Product("WebStorm", 49.0),
        Product("AppCode", 99.0),
        Product("DotTrace", 129.0),
        Product("ReSharper", 149.0))

    println(productList.binarySearch { priceComparison(it, 99.0) })
}
//sampleEnd
```
</div>

Both comparator and comparison binary search can be performed for list ranges as well.

## List 写操作

In addition to the collection modification operations described in [Collection Write Operations](collection-write.html), [mutable](collections-overview.html#集合类型) lists support specific write operations.
Such operations use the index to access elements to broaden the list modification capabilities.

### 添加

To add elements to a specific position in a list, use [`add()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/add.html) and [`addAll()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/add-all.html) providing the position for element insertion as an additional argument.
All elements that come after the position shift to the right.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "five", "six")
    numbers.add(1, "two")
    numbers.addAll(2, listOf("three", "four"))
    println(numbers)
//sampleEnd
}

```
</div>

### 更新

Lists also offer a function to replace an element at a given position - [`set()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/set.html) and its operator form `[]`. `set()` doesn't change the indexes of other elements.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "five", "three")
    numbers[1] =  "two"
    println(numbers)
//sampleEnd
}

```
</div>

[`fill()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/fill.html) simply replaces all the collection elements with the specified value.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4)
    numbers.fill(3)
    println(numbers)
//sampleEnd
}

```
</div>


### 删除

To remove an element at a specific position from a list, use the [`removeAt()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-list/remove-at.html) function providing the position as an argument.
All indices of elements that come after the element being removed will decrease by one.

<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf(1, 2, 3, 4, 3)    
    numbers.removeAt(1)
    println(numbers)
//sampleEnd
}

```
</div>

### 排序

In [Collection Ordering](collection-ordering.html), we describe operations that retrieve collection elements in specific orders.
For mutable lists, the standard library offers similar extension functions that perform the same ordering operations in place.
When you apply such an operation to a list instance, it changes the order of elements in that exact instance.

The in-place sorting functions have similar names to the functions that apply to read-only lists, but without the `ed/d` suffix:

*  `sort*` instead of `sorted*` in the names of all sorting functions: [`sort()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sort.html), [`sortDescending()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sort-descending.html), [`sortBy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/sort-by.html), and so on.
* [`shuffle()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/shuffle.html) instead of `shuffled()`.
* [`reverse()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/reverse.html) instead of `reversed()`.

[`asReversed()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/as-reversed.html) called on a mutable list returns another mutable list which is a reversed view of the original list. Changes in that view are reflected in the original list.
The following example shows sorting functions for mutable lists:


<div class="sample" markdown="1" theme="idea" data-min-compiler-version="1.3">

```kotlin
fun main() {
//sampleStart
    val numbers = mutableListOf("one", "two", "three", "four")

    numbers.sort()
    println("Sort into ascending: $numbers")
    numbers.sortDescending()
    println("Sort into descending: $numbers")

    numbers.sortBy { it.length }
    println("Sort into ascending by length: $numbers")
    numbers.sortByDescending { it.last() }
    println("Sort into descending by the last letter: $numbers")
    
    numbers.sortWith(compareBy<String> { it.length }.thenBy { it })
    println("Sort by Comparator: $numbers")

    numbers.shuffle()
    println("Shuffle: $numbers")

    numbers.reverse()
    println("Reverse: $numbers")
//sampleEnd
}

```
</div>




