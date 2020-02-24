---
type: tutorial
layout: tutorial
title:  "在浏览器中调试 Kotlin"
authors: Yue_plus（翻译）
date: 2017-07-31
showAuthorInfo: false
---

本教程介绍了如何调试 Gradle 构建的 Kotlin/JS 项目。
如果使用的是 Maven 或 IDEA，操作方法相似。

在阅读本教程之前，
请仔细阅读[以 Gradle 入门 Kotlin 和 JavaScript](http://www.kotlincn.net/docs/tutorials/javascript/getting-started-gradle/getting-started-with-gradle.html).


## 创建源代码映射

要在浏览器中调试 Kotlin 源，应告诉编译器生成源映射文件。
将以下行添加到 Gradle 配置中：

<div class="sample" markdown="1" theme="idea" mode="groovy">

``` groovy
compileKotlin2Js {
    kotlinOptions.sourceMap = true
    kotlinOptions.sourceMapEmbedSources = "always"

    // 其他配置选项
} 
```

</div>

现在，如果重新构建项目，则应该同时看到生成的 `.js` 和 `.js.map` 文件。


## 在 Chrome DevTools 中进行调试

要在 Google Chrome 浏览器中调试 Kotlin，应使用 DevTools。
请参阅其[官方文档](https://developer.chrome.com/devtools)
以了解如何打开与使用 DevTools。

现在，如果打开了 DevTools，则应该在 Sources 选项卡中同时看到 JavaScript 与 Kotlin 文件，
如下图所示：

![Debugging in Chrome DevTools]({{ url_for('tutorial_img', filename='javascript/debugging-javascript/chrome-devtools.png')}})

请注意，可以在 Source 选项卡中打开文件夹，并查看项目中正在使用的库的源，
包括 Kotlin 标准库（`kotlin.js`）。
但是，这需要在启用源映射以及嵌入源映射的源的情况下编译库。
因此，推荐做法是：
如果共享 Kotlin/JS 的库，请在发行版中包含源映射。
