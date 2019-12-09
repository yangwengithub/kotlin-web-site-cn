---
type: doc
layout: reference
category: "Collections"
title: "Map 相关操作"
---

# Map 相关操作

在 [map](collections-overview.html#map) 中，键和值的类型都是用户定义的。
对 map 条目的基于键的访问启用了各种特定于 map 的处理函数，从键获取值到对键和值进行单独过滤。
在此页面上，我们提供了来自标准库的 map 处理功能的描述。

## 取键与值

要从映射中检索值，必须提供其键作为 [`get()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-map/get.html) 函数的参数。
还支持简写 `[key]` 语法。 如果找不到给定的键，则返回 `null` 。
还有一个函数 [`getValue()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-value.html) ，它的行为略有不同：如果在映射中找不到键，则抛出异常。
此外，还有两个选项可以解决键缺失的问题：

* [`getOrElse()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-or-else.html) 与列表的工作方式相同：对于不存在的键，其值由给定的 lambda 表达式返回。
* [`getOrDefault()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/get-or-default.html) 如果找不到键，则返回指定的默认值。



```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    println(numbersMap.get("one"))
    println(numbersMap["one"])
    println(numbersMap.getOrDefault("four", 10))
    println(numbersMap["five"])               // null
    //numbersMap.getValue("six")      // exception!
//sampleEnd
}

```


要对 map 的所有键或所有值执行操作，可以从属性 `keys` 和 `values` 中相应地检索它们。 `keys` 是所有映射键的集合， `values` 是所有映射值的集合。



```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    println(numbersMap.keys)
    println(numbersMap.values)
//sampleEnd
}

```


## 过滤

可以使用 [`filter()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter.html) 函数来[过滤](collection-filtering.html) map 或其他集合。
对 map 使用 `filter()` 函数时， `Pair` 将作为参数的谓词传递给它。
它将使用谓词同时过滤其中的键和值。



```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    val filteredMap = numbersMap.filter { (key, value) -> key.endsWith("1") && value > 10}
    println(filteredMap)
//sampleEnd
}

```


还有两种用于过滤 map 的特定函数：按键或按值。
这两种方式，都有对应的函数： [`filterKeys()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter-keys.html) 和 [`filterValues()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/filter-values.html) 。
两者都将返回与给定谓词匹配的新条目映射。
 `filterKeys()` 的谓词仅检查元素键， `filterValues()` 的谓词仅检查值。



```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
    val filteredKeysMap = numbersMap.filterKeys { it.endsWith("1") }
    val filteredValuesMap = numbersMap.filterValues { it < 10 }

    println(filteredKeysMap)
    println(filteredValuesMap)
//sampleEnd
}

```


## `plus` 与 `minus` 操作

Due to the key access to elements, [`plus`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/plus.html) (`+`) and [`minus`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/minus.html) (`-`) operators work for maps differently than for other collections.
`plus` returns a `Map` that contains elements of its both operands: a `Map` on the left and a `Pair` or another `Map` on the right.
When the right-hand side operand contains entries with keys present in the left-hand side `Map`, the result map contains the entries from the right side.



```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    println(numbersMap + Pair("four", 4))
    println(numbersMap + Pair("one", 10))
    println(numbersMap + mapOf("five" to 5, "one" to 11))
//sampleEnd
}

```


`minus` creates a `Map` from entries of a `Map` on the left except those with keys from the right-hand side operand.
So, the right-hand side operand can be either a single key or a collection of keys: list, set, and so on.



```kotlin
fun main() {
//sampleStart
    val numbersMap = mapOf("one" to 1, "two" to 2, "three" to 3)
    println(numbersMap - "one")
    println(numbersMap - listOf("two", "four"))
//sampleEnd
}

```


For details on using [`plusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/plus-assign.html) (`+=`) and [`minusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/minus-assign.html) (`-=`) operators on mutable maps, see [Map write operations](#map-写操作) below.

## Map 写操作

[Mutable](collections-overview.html#集合类型) maps offer map-specific write operations.
These operations let you change the map content using the key-based access to the values.

There are certain rules that define write operations on maps:

* Values can be updated. In turn, keys never change: once you add an entry, its key is constant.
* For each key, there is always a single value associated with it. You can add and remove whole entries.

Below are descriptions of the standard library functions for write operations available on mutable maps.

### 添加与更新条目

To  add a new key-value pair to a mutable map, use [`put()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-map/put.html).
When a new entry is put into a `LinkedHashMap` (the default map implementation), it is added so that it comes last when iterating the map.
In sorted maps, the positions of new elements are defined by the order of their keys. 



```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2)
    numbersMap.put("three", 3)
    println(numbersMap)
//sampleEnd
}

```


To add multiple entries at a time, use [`putAll()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/put-all.html). Its argument can be a `Map` or a group of `Pair`s: `Iterable`, `Sequence`, or `Array`.



```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
    numbersMap.putAll(setOf("four" to 4, "five" to 5))
    println(numbersMap)
//sampleEnd
}

```


Both `put()` and `putAll()` overwrite the values if the given keys already exist in the map. Thus, you can use them to update values of map entries.



```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2)
    val previousValue = numbersMap.put("one", 11)
    println("value associated with 'one', before: $previousValue, after: ${numbersMap["one"]}")
    println(numbersMap)
//sampleEnd
}

```


You can also add new entries to maps using the shorthand operator form. There are two ways:

* [`plusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/plus-assign.html) (`+=`) operator.
* the `[]` operator alias for `put()`.  



```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2)
    numbersMap["three"] = 3     // calls numbersMap.put("three", 3)
    numbersMap += mapOf("four" to 4, "five" to 5)
    println(numbersMap)
//sampleEnd
}

```


When called with the key present in the map, operators overwrite the values of the corresponding entries. 

### 删除条目

To remove an entry from a mutable map, use the [`remove()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/-mutable-map/remove.html) function.
When calling `remove()`, you can pass either a key or a whole key-value-pair.
If you specify both the key and value, the element with this key will be removed only if its value matches the second argument. 



```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
    numbersMap.remove("one")
    println(numbersMap)
    numbersMap.remove("three", 4)            //doesn't remove anything
    println(numbersMap)
//sampleEnd
}

```


You can also remove entries from a mutable map by their keys or values.
To do this, call `remove()` on the map's keys or values providing the key or the value of an entry.
When called on values, `remove()` removes only the first entry with the given value.



```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3, "threeAgain" to 3)
    numbersMap.keys.remove("one")
    println(numbersMap)
    numbersMap.values.remove(3)
    println(numbersMap)
//sampleEnd
}

```



The [`minusAssign`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.collections/minus-assign.html) (`-=`) operator is also available for mutable maps.



```kotlin
fun main() {
//sampleStart
    val numbersMap = mutableMapOf("one" to 1, "two" to 2, "three" to 3)
    numbersMap -= "two"
    println(numbersMap)
    numbersMap -= "five"             //doesn't remove anything
    println(numbersMap)
//sampleEnd
}

```


