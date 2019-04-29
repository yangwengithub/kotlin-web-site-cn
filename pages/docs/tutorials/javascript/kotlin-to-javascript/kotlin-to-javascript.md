---
type: tutorial
layout: tutorial
title:  "Kotlin to JavaScript"
description: "A look at how Kotlin compiles to JavaScript and the use cases for that."
authors: Hadi Hariri
showAuthorInfo: false
---

There are multiple ways to compile Kotlin to JavaScript.
The recommended approach is to use Gradle; if desired, you can also build JavaScript projects directly from
IntelliJ IDEA, use Maven, or compile the code manually from the command line.
To learn more about how to compile to JavaScript please see the corresponding tutorials:
 
* [使用Gradle](../getting-started-gradle/getting-started-with-gradle.html)
* [使用IntelliJ IDEA](../getting-started-idea/getting-started-with-intellij-idea.html)
* [使用Maven](../getting-started-maven/getting-started-with-maven.html)
* [使用命令行](../getting-started-command-line/command-line-library-js.html)


## Examining the compilation output

When compiling (we'll use this term interchangeably with [transpiling](https://en.wiktionary.org/wiki/transpile)) to JavaScript, Kotlin outputs two main files:

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
