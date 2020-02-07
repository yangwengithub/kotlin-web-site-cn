<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">
```kotlin
/*
 使用 JVM 上的任何现有库，因为有 100％ 的兼容性，包括 SAM 支持。
*/

import io.reactivex.Flowable
import io.reactivex.schedulers.Schedulers

Flowable
    .fromCallable {
        Thread.sleep(1000) //  模仿高开销的计算
        "Done"
    }
    .subscribeOn(Schedulers.io())
    .observeOn(Schedulers.single())
    .subscribe(::println, Throwable::printStackTrace)
```
</div>

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">
```kotlin
// 无论是面向 JVM 还是 JavaScript 平台，都可用 Kotlin 写代码然后部署到你想要的地方

import kotlin.browser.window

fun onLoad() {
    window.document.body!!.innerHTML += "<br/>Hello, Kotlin!"
}
```
</div>
