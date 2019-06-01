---
type: tutorial
layout: tutorial
title:  "Kotlin 转 JavaScript"
description: "本文介绍 Kotlin 是如何编译到 JavaScript 及一些使用示例。"
authors: Hadi Hariri
showAuthorInfo: false
---

将 Kotlin 编译到 JavaScript 有许多种方法，<!--
-->推荐的方法是使用 Gradle。如果需要，你还可以直接使用 Intellij IDEA 构建 JavaScript 项目，<!--
-->使用 Maven 编译，或者手动使用命令行编译代码。<!--
-->学习如何将 Kotlin 编译为 JavaScript，请参阅下面的相应教程：
 
* [使用 Gradle](../getting-started-gradle/getting-started-with-gradle.html)
* [使用 IntelliJ IDEA](../getting-started-idea/getting-started-with-intellij-idea.html)
* [使用 Maven](../getting-started-maven/getting-started-with-maven.html)
* [使用命令行](../getting-started-command-line/command-line-library-js.html)


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
   
注意：仅有基于 Intellij IDEA 的项目才会创建包含 `kotlin.js` 和其他库文件的 `lib` 目录，此功能由 Kotlin [构面设置](https://www.jetbrains.com/help/idea/facets.html)中的 ”Copy library runtime files“ 标记控制。在 Maven 或 Gradle 构建（包含多平台项目）中，默认不会把库文件复制到编译输出目录。有关如何使用这些构建系统实现相同功能的说明，请参阅相应的教程。

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

This is the JS code generated for the Kotlin code above (the `main` function). Let's have a closer look at it.
* `if (typeof kotlin === 'undefined') { ... }` checks the existence of the `kotlin` object defined in `kotlin.js`. This object provides access to declarations from the Kotlin runtime and standard library.
* `var ConsoleOutput = function (_, Kotlin) { ... }`: this is the variable named after your Kotlin module. Its value is the result of an anonymous function call. The rest of the code is the function body.
* `var println = Kotlin.kotlin.io.println_s8jyv4$;`: a variable that refers to the `kotlin.io.println` function from the passed in parameter `Kotlin`. This is a way to import the standard `println` function defined in `kotlin.js`.
* `function main(args) { ... }`: your `main` function.
* `_.main_kand9s$ = main;` exports the declared `main` function. The name on the left-hand side will be used to access to the function from outside the module. The name contains a mangled word (`kand9s$`).
This happens because you can have overloaded functions in Kotlin and need a way to translate them to their corresponding JavaScript ones.
To change the generated function name with a custom name, use the [`@JsName` annotation](/docs/reference/js-to-kotlin-interop.html#jsname-annotation).
* `main([]);`: a call of the `main` function.
* `(typeof ConsoleOutput === 'undefined' ? {} : ConsoleOutput, kotlin);` checks the existence of `ConsoleOutput`. If such a variable already exists in the scope, the new declarations will be added to it.

Since the entire anonymous function is self-executing, it will execute as soon as the code is loaded. Its argument will be the object `kotlin` from `kotlin.js`.

If you declare your Kotlin code in a package, `main` would be followed by a package definition part. For example, this goes after the `main` declaration if you put your `main` function in the `org.example.hellojs` package:

<div class="sample" markdown="1" theme="idea" mode="js">

```javascript
  var package$org = _.org || (_.org = {});
  var package$example = package$org.example || (package$org.example = {});
  var package$hellojs = package$example.hellojs || (package$example.hellojs = {});
  package$hellojs.main_kand9s$ = main;
```
</div>

#### 运行代码

The purpose of this code is to write out some text in the console. In order to use this from the browser, load it, preferably from inside an HTML page:

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

(Use the relative paths to the `*.js` files from the directory that contains the HTML page.)

Make sure that you load the `kotlin.js` runtime first, and then your application.

这个空白页面会在控制台中输出`Hello JavaScript!`

   ![Application Output]({{ url_for('tutorial_img', filename='javascript/kotlin-to-javascript/app-output.png')}})

## 概要

As you see, Kotlin aims to create very concise and readable JavaScript allowing us to interact with it as needed. One question of course is why go to
all this trouble to as opposed to just use `console.log()`. Obviously this is a very simple example that shows the basics of how it works and we've focused on analysing the output. As application complexity grows, the benefits 
of using Kotlin and static typing start to become more apparent.

In subsequent tutorials we'll show how you can influence the files generated, for example, change location, prefix and suffixes, and how you can work with modules.
