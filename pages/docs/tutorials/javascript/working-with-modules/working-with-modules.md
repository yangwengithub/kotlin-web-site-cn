---
type: tutorial
layout: tutorial
title:  "Working with Kotlin and JavaScript Modules"
description: "A look at how to use Kotlin to interact with JavaScript modules."
authors: Hadi Hariri 
date: 2016-09-30
showAuthorInfo: false
---


在本教程中你将会看到：

* [使用IntelliJ IDEA配置模块](#configuring-modules-with-intellij-idea)
* [使用Maven或者Gradle配置模块](#configuring-modules-when-using-maven-or-gradle)
* [在浏览器中通过AMD使用Koltin](#using-amd)
* [在node.js中通过CommonJS使用Kotlin](#using-commonjs)



## 使用IntelliJ IDEA配置模块
 
Koltin所生成的JavaScript代码能够兼容异步模块定义（AMD, Asynchronous Module Definition）, CommonJS和统一模块定义（UMD，Unified Module Definitions）。

* **AMD** 通常是使用在客户端的浏览器上，其概念是可以异步加载模块，从而提高性能和易用性。
* **CommonJS** 通常是使用在服务端的模块系统，特别是适合用于node.js，Node模块全都是遵循这样的定义的。 CommonJS 模块也可以通过[Browserify](http://browserify.org/)在浏览器中使用。
* **UMD** 则是试图让模块在客户端和服务端都能使用。

我们可以利用Kotlin编译器配置来选择生成哪种模块。 注意如果使用UMS选项的话，一旦其中一类无法编译的话则会生成另外一类。
一般来说IDEA中Kotlin的编译器选项会影响整个项目而不仅仅是一个单独的模块。
 
![Kotlin Compiler Options]({{ url_for('tutorial_img', filename='javascript/working-with-modules/kotlin-compiler.png')}})

## 使用Maven或者Gradle配置模块
 
如果是使用Maven或者Gradle的话还可以配置模块的输出格式，详细信息请移步[JavaScript Modules](http://kotlinlang.org/docs/reference/js-modules.html)。

## 使用异步模块定义（AMD）
 
当我们要开始使用AMD，首先需要把编译器的选项设置成AMD。通过这些操作，我们就可以编译出的模块就可以和其他任意常规的AMD模块一起使用。
 
举个例子：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
class Customer(val id: Int, val name: String, val email: String) {
    var isPreferred = false
    fun makePreferred() {
        isPreferred = true
    }
}
```
</div>
 
会生成如下的JavaScript代码：

<div class="sample" markdown="1" theme="idea" mode="js">
```javascript
define('customerBL', ['kotlin'], function (Kotlin) {
  'use strict';
  var _ = Kotlin.defineRootPackage(null, /** @lends _ */ {
    Customer: Kotlin.createClass(null, function Customer(id, name, email) {
      this.id = id;
      this.name = name;
      this.email = email;
      this.isPreferred = false;
    }, /** @lends _.Customer.prototype */ {
      makePreferred: function () {
        this.isPreferred = true;
      }
    })
  });
  Kotlin.defineModule('customerBL', _);
  return _;
});
```
</div>
 
假设我们有个项目的结构如下：

![Project Structure AMD]({{ url_for('tutorial_img', filename='javascript/working-with-modules/project-structure-amd.png')}})
 
那么可以在 `index.html` 中利用标签中 `data-main` 属性的值来定义 `main.js` 从而引入 `require.js` ，如下所示：

<div class="sample" markdown="1" theme="idea" mode="xml">
```html
<head>
    <meta charset="UTF-8">
    <title>Sample AMD</title>
    <script data-main="scripts/main"  src="scripts/require.js"></script>
</head>
```
</div>
 
其中 `main.js` 的内容如下：

<div class="sample" markdown="1" theme="idea" mode="js">
```javascript
requirejs.config({
    paths: {
        kotlin: 'out/lib/kotlin.js',
        customerBL: 'out/customerBL'
    }
});

requirejs(["customerBL"], function (customerBL) {
    console.log(customerBL)
});
```
</div>
 
通过这样的配置，我们就可以访问 `customerBL` 模块里的任意函数了。


## 使用 CommonJS 
 
为了能够配合node.js一起使用Kotlin，我们需要把编译器的选项设置成使用CommonJs，这样我们编译出来的模块就可以被node的模块系统所使用。
 
举个例子：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
class Customer(val id: Int, val name: String, val email: String) {
    var isPreferred = false
    fun makePreferred() {
        isPreferred = true
    }
}
```
</div>
 
会生成如下的JavaScript代码：

<div class="sample" markdown="1" theme="idea" mode="js">
```javascript
module.exports = function (Kotlin) {
  'use strict';
  var _ = Kotlin.defineRootPackage(null, /** @lends _ */ {
    Customer: Kotlin.createClass(null, function Customer(id, name, email) {
      this.id = id;
      this.name = name;
      this.email = email;
      this.isPreferred = false;
    }, /** @lends _.Customer.prototype */ {
      makePreferred: function () {
        this.isPreferred = true;
      }
    })
  });
  Kotlin.defineModule('customerBL', _);
  return _;
}(require('kotlin'));

```
</div>
 
函数的最后一行是回调了自身并传递了一个 `kotlin` 参数，该参数代表了基本库。这可以通过以下两种方式获得：

*本地引用* 
 
编译器总会在编译的时候自动生成kotlin.js文件。关联该文件最简单的方式就是在编译器选项中设置 `node_modules` 的输出目录。这样，Node将会在该目录下彻底查询的时候自动地使用该文件。

![Node Modules]({{ url_for('tutorial_img', filename='javascript/working-with-modules/node-modules.png')}})

*NPM目录*
 
Kotlin基本库在[npm](https://www.npmjs.com/)是可用的因此我们可以简单的在项目的`package.json`中以一个依赖的形式进行引用，如下所示：

<div class="sample" markdown="1" theme="idea" mode="js">
```json
{
  "name": "node-demo",
  "description": "A sample of using Kotlin with node.js",
  "version": "0.0.1",
  "dependencies": {
    "kotlin": ">=1.0.4-eap-111"
  }
}
```
</div>
 
我们可以简单地在node.js代码中通过使用 `require` 来导入模块从而可以使用模块里的任意类和函数，如下所示：

<div class="sample" markdown="1" theme="idea" mode="js">
```javascript
var customerBL = require('./scripts/customerBL')

var customer = new customerBL.Customer(1, "Jane Smith", "jane.smith@company.com")

console.dir(customer)
customer.makePreferred()
console.dir(customer)
```
</div>
 
这个例子里，我们是把编译输出目录设置成了 `scripts` 文件夹，一旦程序开始运行那么应该会有如下的输出：

![Output CommonJS]({{ url_for('tutorial_img', filename='javascript/working-with-modules/output-commonjs.png')}})
