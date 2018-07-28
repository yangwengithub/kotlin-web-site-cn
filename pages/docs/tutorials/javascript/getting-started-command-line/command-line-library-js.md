---
type: tutorial
layout: tutorial
title: "使用命令行编译 Kotlin JavaScript 库"
description: "本教程将引导我们使用命令行编译器创建Kotlin JavaScript库。"
authors: hefang
showAuthorInfo: false
related:
    - getting-started.md
---
### 创建一个 Kotlin/JavaScript 库

我们将创建一个简单的 Kotlin/JavaScript 库.

1. 使用我们最喜欢的编辑器创建一个名为 *library.kt* 的文件:

   ``` kotlin
   package org.sample
   
   fun factorial(n: Int): Long = if (n == 0) 1 else n * factorial(n - 1)
   
   inline fun IntRange.forOdd(f: (Int) -> Unit) {
       this.forEach { if (it % 2 == 1) f(it) }
   }
   ```

2. 使用JS编译器编译该库

   ```
   $ kotlinc-js -output sample-library.js -meta-info library.kt
   ```

   `-meta-info` 选项表示创建带有二进制元信息的js附加文件
   
   如果要看到所有的命令参数可以运行:

   ```
   $ kotlinc-js -help
   ```
   
   编译完成以后会出现两个新文件:

   ```
   sample-library.js
   sample-library.meta.js
   ```
   
3. You can simply distribute two JS files, `sample-library.js` and `sample-library.meta.js`.
   前者包含编译后的Javascript代码, 后者包含关于Kotlin代码的一些编译器需要的元信息.

   其实, 你也可以把 `sample-library.meta.js` 文件的内容追加到 `sample-library.js`, 得到一个结果文件.
    
   你也可以创建一个压缩包, 这个压缩包可以做为库进行分发:  
   
   ```
   $ jar cf sample-library.jar *.js
   ```
   
### 使用 Kotlin/JavaScript 库.

   创建 binom.kt:
   
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

   和库一起编译:

```
   $ kotlinc-js -output binom.js -libraries sample-library.meta.js binom.kt
```
   
   `sample-library.js` 和 `sample-library.meta.js` 两个文件都要提供, 因为编译后的 Javascript 文件包含内联的元信息, 编译器需要这些信息
   
   如果库是以包含了 `sample-library.js` 和 `sample-library.meta.js` 文件的压缩包 `sample-library.jar` 提供的, 可以使用下面的命令
   
```
   $ kotlinc-js -output binom.js -libraries sample-library.jar binom.kt
```
  
   
