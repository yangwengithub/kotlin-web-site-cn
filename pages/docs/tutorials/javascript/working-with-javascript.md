---
type: tutorial
layout: tutorial
title:  "与 JavaScript 合用"
description: "看看如何与 DOM 交互以及使用 JavaScript 库"
authors: Hadi Hariri，Yue_plus（翻译）
date: 2017-02-27
showAuthorInfo: false
---


在本教程中，我们将了解如何

* [访问 DOM](#访问-DOM)
* [使用 kotlinx.html 生成 HTML](#使用-kotlinxhtml)
* [使用 dukat 与库交互](#使用-dukat-生成-Kotlin-的头文件)
* [使用 dynamic 与库交互](#使用-dynamic)



## 访问 DOM

Kotlin 标准库围绕 JavaScript API 提供了一系列包装器，用于与文档进行交互。通常访问的主要组件是变量 `document`。有了可以访问的权限，便可以简单地读取与写入相应的属性。例如，要设置页面背景，可以：


<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
document.bgColor = "FFAA12" 
```
</div>

DOM 还提供了一种通过 ID、name、类名、标签等检索特定元素的方法。所有返回的元素均为 `NodeList` 类型，要访问成员，需要将其强制转换为特定类型的元素。
以下代码展示了如何访问页面上的输入元素：

<div class="sample" markdown="1" theme="idea" mode="xml">
```html
<body>
    <input type="text" name="email" id="email"/>
    <script type="text/javascript" src="scripts/kotlin.js"></script>
    <script type="text/javascript" src="scripts/domInteraction.js"></script>
</body>
```
</div>

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
val email = document.getElementById("email") as HTMLInputElement
email.value = "hadi@jetbrains.com"
```
</div>

非常注意：确保在 ``body`` 标签关闭之前引入脚本。另外，将脚本放在顶部将导致在 DOM 完全加载完毕之前运行脚本。

像引用 input 元素一样，也可以访问页面上的其他元素，并将其转换为适当的类型。

## 使用 kotlinx.html

[kotlinx.html 库](http://www.github.com/kotlin/kotlinx.html)提供使用静态类型的 HTML 构建器生成 DOM 的能力。
该库可用于面向 JVM 或 JavaScript 编程时。要使用该库，需要包括相应的依赖项。
在使用 Gradle 的情况下：

<div class="sample" markdown="1" theme="idea" mode="groovy">
```groovy
repositories {
    mavenCentral()
    maven {
        url  "http://dl.bintray.com/kotlin/kotlinx.html/"
    }
}

dependencies {
    compile 'org.jetbrains.kotlinx:kotlinx-html-js:0.6.1'
    compile "org.jetbrains.kotlin:kotlin-stdlib-js:$kotlin_version"
}
```
</div>

相关配置 Gradle 以实现 JavaScript 对象的更多信息，请参见[以 Gradle 入门](getting-started-gradle/getting-started-with-gradle.html)。

包含依赖项后，便可访问提供的不同接口来生成 DOM。
以下代码将在 `window.load` 事件中为 ```div``` 内添加一个带有文本 ```Hello``` 的新 ```span``` 标签。

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
import kotlin.browser.*
import kotlinx.html.*
import kotlinx.html.dom.*

fun printHello() {
    window.onload = {
        document.body!!.append.div {
            span {
                +"Hello"
            }
        }
    }
}
```
</div>

## 使用 dukat 生成 Kotlin 的头文件

标准库中提供了一系列有关 DOM 的包装器以及使用静态类型与 JavaScript 配合使用的函数。
但是，当使用 jQuery 之类的库时会发生什么？对于 JavaScript 生态系统上可用的所有不同库，Kotlin 没有自己的“头”文件，但是 TypeScript 有。
[Definitely Typed](https://github.com/DefinitelyTyped/DefinitelyTyped/) 仓库中提供了大量的头文件可供选择。

使用 `dukat` 工具可以将任何 TypeScript 声明文件转换为 Kotlin 的。要安装该工具，可以使用 `npm`：

```bash
npm -g install dukat
```

要转换文件，只需提供输入文件，还可以提供输出目录。
以下命令将在当前文件夹中转换文件 `jquery.d.ts`，该文件是先前从 [Definitely Typed](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/types/jquery/jquery.d.ts) 仓库中下载到输出文件夹 `headers` 的：

```bash
dukat -d headers jquery.d.ts 
```

生成文件后，便可简单地将其包含在项目中并使用它：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
jQuery("#area").hover { window.alert("Hello!") }
```
</div>

注意，```jQuery``` 需要包含在相应的 HTML 中：

<div class="sample" markdown="1" theme="idea" mode="xml">
```html
<script type="text/javascript" src="js/jquery.js"></script>

<!-- 其他脚本文件…… -->
```
</div>

### 遮蔽头文件

头文件仅包含在运行时定义的函数的函数声明。例如，可以像这样定义一个 ```jQuery``` 函数：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
@JsName("$")
public external fun jQuery(selector: String): JQuery
```
</div>

上面的代码表明该函数是在外部定义的。```@JsName("$")``` 批注允许在运行时将名称映射到 ```$```。
有关外部声明的更多详细信息，请参阅 [JavaScript 互操作文档](/docs/reference/js-interop.html#external-修饰符)。

请注意，TypeScript 与 Kotlin 的类型系统不完全匹配，
因此，如果在使用 Kotlin 的 API 时遇到困难，则可能需要编辑生成的标头。


## 使用 Dynamic

尽管上述解决方案有相应头文件（是自己定义的，或从 TypeScript 头转换而来的文件）的情况下很好用，但通常需要使用一些没有标头的库。
例如，要使用jQuery插件，该插件可让我们将 HTML 表转换为漂亮的可导航网格。

通过 JavaScript 使用的方式是在相应的 ```<table>``` 元素上调用 ```dataTable()```

<div class="sample" markdown="1" theme="idea" mode="xml">
```html
<table id="data" class="display" cellspacing="0" width="100%">
    <thead>
    <tr>
        <th>Name</th>
        <th>Position</th>
        <th>Office</th>
        <th>Age</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>Tiger Nixon</td>
        <td>System Architect</td>
        <td>Edinburgh</td>
        <td>61</td>
    </tr>
    <tr>
        <td>Garrett Winters</td>
        <td>Accountant</td>
        <td>Tokyo</td>
        <td>63</td>
    </tr>
    <tr>
        <td>Ashton Cox</td>
        <td>Junior Technical Author</td>
        <td>San Francisco</td>
        <td>66</td>
    </tr>
    
      . . . 
    
    </tbody>
</table>
```
</div>

在 JavaScript 中调用以下内容

<div class="sample" markdown="1" theme="idea" mode="js">
```javascript
$("#data").dataTable()
```
</div>

假设函数 ```dataTable()``` 不存在，该如何从 Kotlin 中做到这一点？调用它会导致编译器错误？

在这种情况下，可以使用 ```dynamic```，该动态类型允许在面向 JavaScript 编程时使用动态类型。
以下变量被声明为 ```dynamic```，这意味着对其进行的调用均不会导致编译时错误：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
val myObject: dynamic = null

{ ... }

myObject.callAnything()
```
</div>

上面的代码进行编译。
但是，如果在使用前未正确初始化对象或在运行时未定义 ```callAnything()```，将会产生运行时错误。
 
标准库定义了一个名为 [`asDynamic()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.js/as-dynamic.html) 的函数，该函数将值转换为动态类型。
在前面的示例中，使用了 jQuery 处理 DOM 元素，现在可以将其与 `asDynamic()` 结合起来，然后在结果上调用 `dataTable()`：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
$("#data").asDynamic().dataTable()
```
</div>

重要的是要理解，就像在 `callAnything()` 的情况下一样，`dataTable()` 函数必须在运行时存在。
这种情况下，需要确保包含插件的相应脚本文件：

<div class="sample" markdown="1" theme="idea" mode="xml">
```html
<script type="text/javascript" src="js/jquery.js"></script>
<script type="text/javascript" src="js/jquery.dataTables.js"></script>

<!-- 其他脚本文件…… -->
```
</div>

有关 ```dynamic``` 的更多信息，请参见[动态类型](../../reference/dynamic-type.html)。
