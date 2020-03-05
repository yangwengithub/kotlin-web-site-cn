---
type: tutorial
layout: tutorial
title: "使用命令行编译 Kotlin JavaScript 库"
description: "本教程将引导我们使用命令行编译器创建 Kotlin JavaScript 库。"
authors: hefang
showAuthorInfo: false
related:
    - getting-started.md
---

>__Warning__: this tutorial is outdated for Kotlin {{ site.data.releases.latest.version }}.
>We strongly recommend using Gradle for Kotlin/JS projects. For instructions on creating 
>Kotlin/JS projects with Gradle, see [Setting up a Kotlin/JS project](../setting-up.html)
{:.note}
>
### 创建一个 Kotlin/JavaScript 库

我们将创建一个简单的 Kotlin/JavaScript 库。

1. 使用我们最喜欢的编辑器创建一个名为 *library.kt* 的文件：

   <div class="sample" markdown="1" theme="idea" data-highlight-only>

   ``` kotlin
   package org.sample
   
   fun factorial(n: Int): Long = if (n == 0) 1 else n * factorial(n - 1)
   
   inline fun IntRange.forOdd(f: (Int) -> Unit) {
       this.forEach { if (it % 2 == 1) f(it) }
   }
   ```

   </div>

2. 使用 JS 编译器编译该库：

   ```
   $ kotlinc-js -output sample-library.js -meta-info library.kt
   ```

   `-meta-info` 选项表示创建带有二进制元信息的 js 附加文件。
   
   如果要看到所有的命令参数可以运行：

   ```
   $ kotlinc-js -help
   ```
   
   编译完成以后会出现两个新文件：

   ```
   sample-library.js
   sample-library.meta.js
   ```
   
3. 你可以简单地分发这两个 JS 文件，`sample-library.js` 和 `sample-library.meta.js`。<!--
   -->前者包含编译后的 Javascript 代码，后者包含关于 Kotlin 代码的一些编译器需要的元信息。

   其实，你也可以把 `sample-library.meta.js` 文件的内容追加到 `sample-library.js`，分发这个合并的结果文件。

   你也可以创建一个压缩包，这个压缩包可以做为库进行分发：

   <div class="sample" markdown="1" theme="idea" mode="shell">
   ```
   $ jar cf sample-library.jar *.js
   ```

   </div>

### 使用 Kotlin/JavaScript 库

   创建 binom.kt:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
import org.sample.factorial
import org.sample.forOdd
    
fun binom(m: Int, n: Int): Long =
    if (m < n) factorial(n) / factorial(m) / factorial(n-m) else 1
        
fun oddFactorial(n: Int): Long {
    var result: Long = 1L
    (1..n).forOdd { result = result * it }
    return result
}        
```

</div>

   编译代码，使用上面创建的库：

<div class="sample" markdown="1" theme="idea" mode="shell">

```bash
$ kotlinc-js -output binom.js -libraries sample-library.meta.js binom.kt
```

</div>

   `sample-library.js` 和 `sample-library.meta.js` 两个文件都要提供, 因为编译后的 Javascript 文件包含内联的元信息, 编译器需要这些信息

   如果库是以包含了 `sample-library.js` 和 `sample-library.meta.js` 文件的压缩包 `sample-library.jar` 提供的, 可以使用下面的命令

<div class="sample" markdown="1" theme="idea" mode="shell">

```bash
$ kotlinc-js -output binom.js -libraries sample-library.jar binom.kt
```

</div>

   
