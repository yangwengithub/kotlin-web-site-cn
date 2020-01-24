### 安全

彻底告别那些烦人的 NullPointerException——著名的[十亿美金的错误](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)。

``` kotlin
var output: String
output = null   // 编译错误
```

Kotlin 可以保护你避免对可空类型的误操作

``` kotlin
val name: String? = null    // 可控类型
println(name.length())      // 编译错误
```

并且如果你检测类型是正确的，编译器会为你做自动类型转换

``` kotlin
fun calculateTotal(obj: Any) {
    if (obj is Invoice)
        obj.calculateTotal()
}
```
