---
type: doc
layout: reference
category: "JavaScript"
title: "JavaScript 模块"
---

# JavaScript 模块

Kotlin 允许你将 Kotlin 项目编译为热门模块系统的 JavaScript 模块。以下是<!--
-->可用选项的列表：

1. 无模块（Plain）。不为任何模块系统编译。像往常一样，你可以在全局作用域中以其名称访问模块。
   默认使用此选项。
2. [异步模块定义（AMD，Asynchronous Module Definition）](https://github.com/amdjs/amdjs-api/wiki/AMD)，它尤其为
   require.js 库所使用。
3. [CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1) 约定，广泛用于 node.js/npm
   （`require` 函数和 `module.exports` 对象）
4. 统一模块定义（UMD，Unified Module Definitions），它与 *AMD* 和 *CommonJS* 兼容，
   并且当在运行时 *AMD* 和 *CommonJS* 都不可用时，作为“plain”使用。

## 面向浏览器
 
If you're targeting the browser, you can specify the desired module type in the `webpackTask` configuration block:
 
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
import org.jetbrains.kotlin.gradle.targets.js.webpack.KotlinWebpackOutput.Target.COMMONJS

kotlin {
    target {
        browser {
            webpackTask {
                output.libraryTarget = COMMONJS 
                //output.libraryTarget = "commonjs" // alternative
             }
        }
    }
}

```

</div>
  
This way, you'll get a single JS file with all dependencies included.

## 创建库与 node.js 文件

If you're creating a JS library or a node.js file, define the module kind as described below.

### 选择目标模块系统

To select module kind, set the `moduleKind` compiler option in the Gradle build script.

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
compileKotlinJs.kotlinOptions.moduleKind = "commonjs"

```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
tasks.named("compileKotlinJs") {
    this as KotlinJsCompile
    kotlinOptions.moduleKind = "commonjs"
}
```

</div>
</div>

Available values are: `plain`, `amd`, `commonjs`, `umd`.

In Kotlin Gradle DSL, there is also a shortcut for setting the CommonJS module kind:

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```
kotlin {
    target {
         useCommonJs()
    }
}
```
</div>

## `@JsModule` 注解

要告诉 Kotlin 一个 `external` 类、 包、 函数或者属性是一个 JavaScript 模块，你可以使用 `@JsModule`
注解。考虑你有以下 CommonJS 模块叫“hello”：

<div class="sample" markdown="1" theme="idea" mode="java">

``` javascript
module.exports.sayHello = function(name) { alert("Hello, " + name); }
```

</div>

你应该在 Kotlin 中这样声明：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

``` kotlin
@JsModule("hello")
external fun sayHello(name: String)
```

</div>


### 将 `@JsModule` 应用到包

一些 JavaScript 库导出包（命名空间）而不是函数和类。
从 JavaScript 角度讲，它是一个具有一些成员的对象，这些成员*是*类、函数和属性。
将这些包作为 Kotlin 对象导入通常看起来不自然。
编译器允许使用以下助记符将导入的 JavaScript 包映射到 Kotlin 包：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
@file:JsModule("extModule")
package ext.jspackage.name

external fun foo()

external class C
```

</div>

其中相应的 JavaScript 模块的声明如下：

<div class="sample" markdown="1" theme="idea" mode="js">

``` javascript
module.exports = {
    foo:  { /* 此处一些代码 */ },
    C:  { /* 此处一些代码 */ }
}
```

</div>

重要提示：标有 `@file:JsModule` 注解的文件无法声明非外部成员。
下面的示例会产生编译期错误：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
@file:JsModule("extModule")
package ext.jspackage.name

external fun foo()

fun bar() = "!" + foo() + "!" // 此处报错
```

</div>

### 导入更深的包层次结构

在前文示例中，JavaScript 模块导出单个包。
但是，一些 JavaScript 库会从模块中导出多个包。
Kotlin 也支持这种场景，尽管你必须为每个导入的包声明一个新的 `.kt` 文件。

例如，让我们的示例更复杂一些：

<div class="sample" markdown="1" theme="idea" mode="js">

``` javascript
module.exports = {
    mylib: {
        pkg1: {
            foo: function() { /* 此处一些代码 */ },
            bar: function() { /* 此处一些代码 */ }
        },
        pkg2: {
            baz: function() { /* 此处一些代码 */ }
        }
    }
}
```

</div>

要在 Kotlin 中导入该模块，你必须编写两个 Kotlin 源文件：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
@file:JsModule("extModule")
@file:JsQualifier("mylib.pkg1")
package extlib.pkg1

external fun foo()

external fun bar()
```
</div>

以及

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
@file:JsModule("extModule")
@file:JsQualifier("mylib.pkg2")
package extlib.pkg2

external fun baz()
```

</div>

### `@JsNonModule` 注解

当一个声明具有 `@JsModule`、当你并不把它编译到一个 JavaScript 模块时，你不能在 Kotlin 代码中使用它。
通常，开发人员将他们的库既作为 JavaScript 模块也作为可下载的` .js` 文件分发，你可以将这些文件复制到<!--
-->项目的静态资源，并通过 `<script>` 元素包含。 要告诉 Kotlin，可以<!--
-->在非模块环境中使用一个 `@JsModule` 声明，你应该放置 `@JsNonModule` 声明。例如，
给定 JavaScript 代码：

<div class="sample" markdown="1" theme="idea" mode="js">

``` javascript
function topLevelSayHello(name) { alert("Hello, " + name); }
if (module && module.exports) {
    module.exports = topLevelSayHello;
}
```

</div>

可以这样描述：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
@JsModule("hello")
@JsNonModule
@JsName("topLevelSayHello")
external fun sayHello(name: String)
```

</div>


### 备注

Kotlin 以 `kotlin.js` 标准库作为单个文件分发，该文件本身被编译为 UMD 模块，因此<!--
-->你可以使用上述任何模块系统。在 NPM 上作为 [`kotlin` 包](https://www.npmjs.com/package/kotlin) 提供


