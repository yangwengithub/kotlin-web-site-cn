---
type: tutorial
layout: tutorial
title:  "Kotlin to JavaScript"
description: "A look at how Kotlin compiles to JavaScript and the use cases for that."
authors: Hadi Hariri 
date: 2016-09-29
showAuthorInfo: false
---

有多种方式可以把Kotlin编译成JavaScript.
推荐的方法是使用Gradle; 如果需要的话，你也可以直接使用IntelliJ IDEA建立JavaScript项目，使用Maven，或者从命令行手动编译代码。
要了解有关如何编译为JavaScript的更多信息，请参阅相应的教程
 
* [使用Gradle](../getting-started-gradle/getting-started-with-gradle.html)
* [使用IntelliJ IDEA](../getting-started-idea/getting-started-with-intellij-idea.html)
* [使用Maven](../getting-started-maven/getting-started-with-maven.html)
* [使用命令行](../getting-started-command-line/command-line-library-js.html)


### Examining the Compilation Output

编译后Kotlin会生成两个文件:

* `kotlin.js`. Kotlin正在使用版本的运行时标准库, 该文件在程序运行期间不会改变
* `{module}.js`. 程序实际的代码。 所有文件都被编译成一个与模块名称相同的JavaScript文件。

另外, 每个文件都会生成一个 `{file}.meta.js` 文件用于对应Kotlin代码和Javascript代码的关系

根据上面的描述, 我们给出下面的代码(模块名是 `ConsoleOutput`)

<div class="sample" markdown="1" data-target-platform="js" theme="idea">
```kotlin
fun main(args: Array<String>) {
    println("Hello JavaScript!")
}
```
</div>

Kotlin编译器将会生成下面的文件


   ![Compiler Output]({{ url_for('tutorial_img', filename='javascript/kotlin-to-javascript/compiler-output.png')}})
   
注意：包含kotlin.js和其他库文件的lib目录仅在基于IntelliJ IDEA的项目中创建，并由Kotlin构面设置中的复制库运行时文件标志控制。 在Maven或Gradle构建（包括多平台项目）中，默认情况下没有库文件被复制到编译输出目录。 请参阅相应的教程，了解如何在这些构建系统上实现相同的效果。

我们最感兴趣的文件是`ConsoleOutput.js`

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```javascript
if (typeof kotlin === 'undefined') {
  throw new Error("Error loading module 'ConsoleOutput'. Its dependency 'kotlin' was not found. /* ... */");
}
var ConsoleOutput = function (_, Kotlin) {
  'use strict';
  var println = Kotlin.kotlin.io.println_s8jyv4$;
  function main(args) {
    println('Hello JavaScript!');
  }
  _.main_kand9s$ = main;
  main([]);
  Kotlin.defineModule('ConsoleOutput', _);
  return _;
}(typeof ConsoleOutput === 'undefined' ? {} : ConsoleOutput, kotlin);
```
</div>

这是我们的主函数生成的输出, 我们可以看到它声明了一个函数并将其赋给一个名为`ConsoleOutput`的变量，该变量名与模块名称一致。
接下来，它使用传入参数`Kotlin`来调用函数`defineRootPackage`。 这个函数又接受一个对应于包中声明的代码的对象。 由于我们没有声明包名，它使用默认的根包。 如果我们在一个包中声明我们的代码，那么调用将是`definePackage`。这些函数都是Kotlin标准库文件`kotlin.js`中的一部分
 
我们只声明了一个 `main` 函数. 我们可以看到编译器加了一些字母做为后缀, 原因是在Kotlin中可能会有重载的函数, 编译器会用这种方法把重载的函数编译成唯一的Javascript函数. 虽然目前我们不能影响这些函数的名字，但是会引入一个注解（`@JsName`）来允许这个选项。

最后把代码定义成了一个模块

鉴于这是一个自执行函数，只要代码被加载，它就会执行，接收`kotlin.js`中定义的对象`kotlin`作为参数，并提供对所有使用的函数的访问。

#### 运行代码

这段代码的目的是在控制台中写出一些文本。 为了在浏览器中使用它，我们需要从HTML页面中加载它：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Console Output</title>
</head>
<body>

<script type="text/javascript" src="out/production/ConsoleOutput/lib/kotlin.js"></script>
<script type="text/javascript" src="out/production/ConsoleOutput/ConsoleOutput.js"></script>
</body>
</html>
```
(我们使用HTML页面所在在相对路径来加载`*.js`文件)

请注意, 我们要先加载Kotlin运行时标准库`kotlin.js`, 再加载我们的业务代码文件

这个空白页面会在控制台中输出`Hello JavaScript`

   ![Application Output]({{ url_for('tutorial_img', filename='javascript/kotlin-to-javascript/app-output.png')}})



## 概要

可以看出，Kotlin旨在创建非常简洁易读的JavaScript，使我们能够根据需要与之交互。 有一个问题，当然是为什么要把所有的这些例子都用到console.log()上。 显然，这是一个非常简单的例子，它显示了Kotlin是如何工作的基础知识，我们专注于分析输出。 随着应用程序复杂性的增长，使用Kotlin和强类型编程好处会变得更加明显。

在随后的教程中，我们将看到如何影响生成的文件，如位置，前缀和后缀，以及如何使用模块。
