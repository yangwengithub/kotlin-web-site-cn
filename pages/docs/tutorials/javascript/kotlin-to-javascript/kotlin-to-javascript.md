---
type: tutorial
layout: tutorial
title:  "Kotlin 转 JavaScript"
description: "本文介绍 Kotlin 是如何编译到 JavaScript 及一些使用示例。"
authors: Hadi Hariri，刘文俊（翻译）
showAuthorInfo: true
---

This tutorial explains how Kotlin code compiles to Javascript.
To learn more about how to create a Kotlin/JS project, see [Setting up a Kotlin/JS project](../setting-up.html).

## 检查编译输出

当编译到（我们会交替使用编译（compile）和转译（[transpile](https://en.wiktionary.org/wiki/transpile)）两个术语） JavaScript 时，Kotlin 会输出两个主要的文件：

* `kotlin.js`：Kotlin 的运行时和标准库。这个文件在所有程序中都是一样的，它与我们使用的 Kotlin 版本有关。
* `{module}.js`：来自应用程序的实际代码。 所有文件都编译为单个 JavaScript 文件，该文件与模块同名。

另外，上面的每个文件还有一个相应的 `{file}.meta.js` 元数据文件，该文件用来实现反射及其他的功能。

根据上面的描述, 我们给出下面的代码（模块名是 `ConsoleOutput`）

<div class="sample" markdown="1" data-target-platform="js" theme="idea">

```kotlin
fun main(args: Array<String>) {
    println("Hello JavaScript!")
}
```
</div>

Kotlin 编译器将会生成下面的文件：

   ![Compiler Output]({{ url_for('tutorial_img', filename='javascript/kotlin-to-javascript/compiler-output.png')}})

我们最感兴趣的文件是 `ConsoleOutput.js`：

<div class="sample" markdown="1" theme="idea" mode="js">

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

这是由上面的 Kotlin 代码（`main` 函数）编译生成的 js 代码，让我们再仔细地看看吧。
* `if (typeof kotlin === 'undefined') { ... }`：检查 `kotlin.js` 中定义的 `kotlin` 对象是否存在。通过该对象，我们才能访问到 Kotlin 运行时和标准库中声明的各种类和函数。
* `var ConsoleOutput = function (_, Kotlin) { ... }`：这是以你的 Kotlin 模块命名的变量，它的值是匿名函数的调用结果，函数体就是我们的模块中的代码。
* `var println = Kotlin.kotlin.io.println_s8jyv4$;`：一个变量，它从传入的参数 `Kotlin` 中引用了 `kotlin.io.println` 函数，这是导入 `kotlin.js` 中定义的标准 `println` 函数的方法。
* `function main(args) { ... }`：你的 `main` 函数。
* `_.main_kand9s$ = main;`：导出声明的 `main` 函数，左侧的名称将用于从模块外部访问该函数。该名称被 `kand9s$` 修饰，<!--
-->这是因为你可以在 Kotlin 中使用重载函数，而 JavaScript 并不支持重载，所以在翻译为 JavaScript 代码的时候需要以此区分。<!--
-->要自定义生成的函数的名称，请使用 [`@JsName` 注解](/docs/reference/js-to-kotlin-interop.html#jsname-注解)。
* `main([]);`：调用 `main` 函数。
* `(typeof ConsoleOutput === 'undefined' ? {} : ConsoleOutput, kotlin);`：检查 `ConsoleOutput` 是否存在。如果在作用域中早已存在该变量，则会将新的声明添加到其中。

由于整个匿名函数是自执行的，因此只要代码被加载它就会运行，它的参数正是来自 `kotlin.js` 文件中的 `kotlin` 对象。

如果你把你的 Kotlin 代码写在一个包中，在编译后的 `main` 函数后面就会生成一段包定义的代码。例如，如果你把 `main` 方法写在 `org.example.hellojs` 包中，那么编译后的 `main` 函数后面会多出这样一段：

<div class="sample" markdown="1" theme="idea" mode="js">

```javascript
  var package$org = _.org || (_.org = {});
  var package$example = package$org.example || (package$org.example = {});
  var package$hellojs = package$example.hellojs || (package$example.hellojs = {});
  package$hellojs.main_kand9s$ = main;
```
</div>

#### 运行代码

这段代码的目的是在控制台输出一些文本。要在浏览器中运行它，请加载它，最好是在 HTML 页面中加载：

<div class="sample" markdown="1" theme="idea" mode="xml" auto-indent="false">

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
</div>

（使用该 HTML 页面所在的目录到 `*.js` 文件的相对路径）

请确保首先加载 `kotlin.js` 运行时库，然后再加载你的应用程序。

该程序的输出是一个空白页，在控制台中打印出 `Hello JavaScript!`。

   ![Application Output]({{ url_for('tutorial_img', filename='javascript/kotlin-to-javascript/app-output.png')}})

## 小结

如你所见，Kotlin 旨在生成十分简洁易读的 JavaScript 代码，让我们能够根据需要与其进行交互。当然，有个问题是，为什么我们非要自找麻烦，<!--
-->而不是直接使用 `console.log()`？显然，这是一个十分简单的实例，它展示了 Kotlin 编译到 JavaScript 的基本原理，并且让我们能够专注于分析它的输出。<!--
-->随着应用程序复杂性的增加，使用 Kotlin 和静态类型的好处就开始变得更加明显。

在后续的教程中，我们将展示如何影响生成的文件，例如，更改位置、前缀和后缀，以及如何使用模块。
