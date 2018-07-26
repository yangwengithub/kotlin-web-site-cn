---
type: tutorial
layout: tutorial
title: "在工程中混用 Java 与 Kotlin"
description: "这个教程将介绍如何在一个 IntelliJ IDEA 工程中同时使用 Java 与 Kotlin。"
authors: Hadi Hariri
showAuthorInfo: false
date: 2014-09-23
related:
    - getting-started.md
---

教程将使用 IntelliJ IDEA（Ultimate 版或 Community 版）。如果需要使用构建工具，请参见相应<!--
-->[构建工具](build-tools.html)。 如需了解如何使用  IntelliJ IDEA 开始一个 Kotlin 项目，
请参见[以 IntelliJ IDEA 入门](getting-started.html)。

### 在 Kotlin 项目中加入 Java 代码
向一个 Kotlin 项目添加 Java 代码非常简单，直接在包（或文件夹）中新建（Ctrl+N/Cmd+N）Java 文件即可。

![New Java Class]({{ url_for('tutorial_img', filename='mixing-java-kotlin-intellij/new-java-class.png') }})


现在不需要进行其它的额外操作，就可以从 Kotlin 代码中使用新建的 Java 类，反之亦然。例如添加以下 Java 类：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` java
public class Customer {

    private String name;

    public Customer(String s){
        name = s;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
</div>

可以像其它 Kotlin 内置类型一样，直接使用它。

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
val customer = Customer("Phase")

println(customer.getName())
```
</div>


### 在已有的 Java 项目中使用 Kotlin
将 Kotlin 代码加入一个已有的 Java 工程也非常简单。有以下几种方式：

#### 创建新的 Kotlin 文件
只要决定在何处创建文件，并创建它。

![New Kotlin File]({{ url_for('tutorial_img', filename='mixing-java-kotlin-intellij/new-kotlin-file.png') }})

如果这是第一次创建 Kotlin 文件，IntelliJ IDEA 会提示添加所需的 Kotlin 运行库。

![Add Kotlin Runtime]({{ url_for('tutorial_img', filename='mixing-java-kotlin-intellij/add-kotlin-runtime.png') }})

在 Java 项目中，选择配置为 Java 模块（as a Kotlin Java Module）。
接下来选择需要配置的模块（如果项目中有多个模块），并选择添加<!--
-->运行库或者直接使用 Kotlin 插件中提供的运行库。

![Bundling Kotlin Runtime]({{ url_for('tutorial_img', filename='mixing-java-kotlin-intellij/bundling-kotlin-option.png') }})

#### 添加已有的 Kotlin 代码
如果不创建新的 Kotlin 文件，而直接向项目添加一个 Kotlin 文件，IntelliJ IDEA不会提示配置 Kotlin 运行库。因此需要从
*Tools\|Kotlin* 菜单，手动进行这个操作。


![Kotlin Menu]({{ url_for('tutorial_img', filename='mixing-java-kotlin-intellij/kotlin-menu.png') }})


会弹出与 [创建新的 Kotlin 文件](#创建新的-kotlin-文件) 相同的对话框。

#### 使用 J2K 将已有的 Java 代码 转换为 Kotlin 代码

Kotlin 插件自带了一个 Java 到 Kotlin 的转换工具，可以从 IntelliJ IDEA 的 *Code* 菜单中找到它。

![Convert Java to Kotlin Menu]({{ url_for('tutorial_img', filename='mixing-java-kotlin-intellij/convert-java-to-kotlin.png') }})

选择一个 Java 文件，就可以使用这个功能将它自动转换成 Kotlin 代码。
这个工具能很好的转换大部分的 Java 代码，但有时转换结果也需要进行手动调整。
