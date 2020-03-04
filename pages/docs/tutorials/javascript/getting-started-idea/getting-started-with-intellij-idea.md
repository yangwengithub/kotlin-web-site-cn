---
type: tutorial
layout: tutorial
title:  "以 IntelliJ IDEA 入门 Kotlin 和 JavaScript"
description: "本文介绍如何使用 IntelliJ IDEA 把 Kotlin 编译到 JavaScript"
authors: Hadi Hariri，刘文俊（翻译）
date: 2016-09-30
showAuthorInfo: true
---

>__Warning__: this tutorial is outdated for Kotlin {{ site.data.releases.latest.version }}.
>We strongly recommend using Gradle for Kotlin/JS projects. For instructions on creating 
>Kotlin/JS projects with Gradle, see [Setting up a Kotlin/JS project](../setting-up.html)
{:.note}
>
在本教程中我们会学习如何

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
-->请查看 [Kotlin 转 JavaScript](../kotlin-to-javascript/kotlin-to-javascript.html) 教程。

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
* 路径指向的是 IntelliJ IDEA 在编译程序时使用的默认输出位置，下面我们会看到如何修改它。

接下来要做的就是编译我们的程序（通过 Build\|Build Project 菜单），当 JavaScript 文件生成完之后，我们就可以在浏览器中打开 `index.html` 文件，并在<!--
-->控制台调试窗口中查看结果。

## 调试程序

`此功能仅在旗舰版中支持。`

为了使用 IntelliJ IDEA 调试应用程序，我们需要执行两个步骤：

1. 安装 [JetBrains Chrome 插件](https://chrome.google.com/webstore/detail/jetbrains-ide-support/hmhgeddbohgjknpmjagkdomcpobmllji?hl=en)以支持通过 Chrome 在 IntelliJ IDEA 中进行调试。<!--
-->这个插件对于使用 IntelliJ IDEA 开发的任意类型的 Web 应用程序都有用，而不仅仅是 Kotlin。

2. 配置 Kotlin 编译器，生成源码映射文件（source map），配置项可通过 `Preferences|Kotlin Compiler` 找到。

![SourceMaps]({{ url_for('tutorial_img', filename='javascript/getting-started-idea/compiler-options-sourcemaps.png')}})

完成后，我们只需右键单击 `index.html` 文件并选择 Debug 选项。这将启动 Chrome，然后代码会停在 IntelliJ IDEA 我们的代码中定义的断点处，<!--
-->在这里我们可以对表达式进行求值，单步执行代码等。

![Debugger]({{ url_for('tutorial_img', filename='javascript/getting-started-idea/debugger.png')}})

你也可以使用标准的 Chrome 调试器调试 Kotlin 应用程序，只需要确保你已经生成了源码映射文件。

## 配置编译器选项

Kotlin 提供了一系列的编译器选项，它们也可以在 IntelliJ IDEA 中进行配置。除了刚刚介绍过的生成源码映射文件的选项，<!--
-->我们还可以设置如下属性：

* **输出文件前缀**：我们可以在编译器生成的输出文件前加上额外的 JavaScript 代码，为此，我们需要使用此选项指明我们想要包含的 JavaScript 文件的名称。
* **输出文件后缀**：与上面相同，但是这个选项会把指定文件的内容附加到编译器输出的代码后面。
* **复制运行时库文件**：指定我们希望将 `kotlin.js` 库输出到哪个子文件夹，默认情况下它是 `lib`，这就是为什么我们会在 HTML 中引用这个路径的原因。
* **模块类型**：指定我们要遵循的模块标准，这将在[使用模块](../working-with-modules/working-with-modules.html)教程中进行更深入的介绍。

## 小节

在本教程中，我们已经了解了如何创建编译到 JavaScript 的 Kotlin 应用程序，调试它以及设置编译器选项。 在接下来的其他教程中，我们将介绍更深入的主题，例如与DOM交互等。






