---
type: tutorial
layout: tutorial
title:  "以 IntelliJ IDEA 入门 Kotlin 和 JavaScript"
description: "本文介绍如何使用 IntelliJ IDEA 把 Kotlin 编译到 JavaScript"
authors: Hadi Hariri，刘文俊（翻译）
date: 2016-09-30
showAuthorInfo: true
---

在本教程中，我们会学习如何

* [创建编译到 JavaScript 的应用程序](#创建编译到-javascript-的应用程序)
* [调试程序](#调试程序)
* [配置编译器选项](#配置编译器选项)


## 创建编译到 JavaScript 的应用程序

在创建面向 JavaScript 的新应用程序或模块时，我们需要选择 `Kotlin - JavaScript` 作为编译目标：

 
 ![First Step of Wizard]({{ url_for('tutorial_img', filename='javascript/getting-started-idea/first-step-wizard.png')}})
 
下一步会提示我们选择 Kotlin 运行时库。默认情况下，插件会选择与当前安装版本关联的 Kotlin 库。<!--
-->如果我们不需要修改，在输入项目名称和保存位置后<!--
-->点击 Finish 即可。
 
![Selecting Runtime]({{ url_for('tutorial_img', filename='javascript/getting-started-idea/second-step-wizard.png')}})
 
当 IDE 完成项目创建之后，我们会看到这样的目录结构：
 
![Project Structure]({{ url_for('tutorial_img', filename='javascript/getting-started-idea/project-structure.png')}})

此时我们可以开始编写 Kotlin 代码了。在这个示例中，我们会编写一些代码，将字符串<!--
-->输出到控制台窗口中。

<div class="sample" markdown="1" theme="idea" data-target-platform="js">
```kotlin
fun main(args: Array<String>) {
    val message = "Hello JavaScript!"
    println(message)
}
```
</div>

我们现在需要一个 HTML 页面来加载代码，因此我们将创建一个名为 `index.html` 的文件。如果你想了解 Kotlin 是如何编译到 JavaScript 以及关于输出文件的更多信息，<!--
-->请查看[Kotlin 转 JavaScript](../kotlin-to-javascript/kotlin-to-javascript.html)教程。

<div class="sample" markdown="1" theme="idea" mode="xml" auto-indent="false">
```html 
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Console Output</title>
    </head>
    <body>
        <script type="text/javascript" src="out/production/sampleapp/lib/kotlin.js"></script>
        <script type="text/javascript" src="out/production/sampleapp/sampleapp.js"></script>
    </body>
</html>
```
</div>

几个要点：

* 应该先加载 `kotlin.js` 文件，因为我们的程序会使用到它。
* 路径指向的是 Intellij IDEA 在编译程序时使用的默认输出位置，下面我们会看到如何修改它。

接下来要做的就是编译我们的程序（通过 Build\|Build Project 菜单），当 JavaScript 文件生成完之后，我们就可以在浏览器中打开 `index.html` 文件，并在<!--
-->控制台调试窗口中查看结果。

## Debugging the application

`This feature is only supported in the Ultimate edition.`

In order to debug the application using IntelliJ IDEA, we need to perform two steps:

1. Install the [JetBrains Chrome Extension](https://chrome.google.com/webstore/detail/jetbrains-ide-support/hmhgeddbohgjknpmjagkdomcpobmllji?hl=en) which allows debugging inside IntelliJ IDEA via Chrome. This is useful for any type
of web application developed with IntelliJ IDEA, not just Kotlin.

2. Configure the Kotlin Compiler to generate source maps, accessible via `Preferences|Kotlin Compiler`

![SourceMaps]({{ url_for('tutorial_img', filename='javascript/getting-started-idea/compiler-options-sourcemaps.png')}})

Once that's done, we simply right-click on our `index.html` file and select the Debug option. This launches Chrome and then stops at the breakpoint defined in our code inside IntelliJ IDEA, from where
we can evaluate expressions, step through code, etc.

![Debugger]({{ url_for('tutorial_img', filename='javascript/getting-started-idea/debugger.png')}})

It is also possible to debug Kotlin applications using the standard Chrome debugger. Just make sure that you do generate source maps.

## Configuring Compiler Options

Kotlin provides a series of compiler options that are accessible in IntelliJ IDEA also. In addition to the one we've just seen for
generating source maps, we also have the ability to set

* **Output file prefix**. We can prefix the output the compiler generates with additional JavaScript. In order to do so, we indicate the name of the file that contains the JavaScript we want in this box.
* **Output file postfix**. Same as above, but in this case the compiler will append the contents of the selected file to the output.
* **Copy runtime library files**. Indicates in what subfolder we want the `kotlin.js` library to be output to. By default it is `lib` which is why in the HTML we are referencing this path. 
* **Module Kind**. Indicates what module standard to follow. This is covered in the [Working with Modules](../working-with-modules/working-with-modules.html) tutorial in more depth.

## Summary

In this tutorial we've seen how to create a Kotlin application that targets JavaScript, debug it as well as set compiler options. In other tutorials we'll cover more in-depth topics such as interacting with the DOM, etc.






