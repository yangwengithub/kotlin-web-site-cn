---
type: tutorial
layout: tutorial
title:  "以 Gradle 入门 Kotlin 和 JavaScript"
description: "本文介绍如何使用 Gradle 把 Kotlin 编译到 JavaScript"
authors: Hadi Hariri，刘文俊（翻译）
date: 2016-11-04
showAuthorInfo: true
---

在本教程中，我们会学习如何

* 使用 Gradle 创建编译到 JavaScript 的应用程序
* [配置编译器选项](#配置编译器选项)

为了使用 Gradle 编译到 JavaScript，我们需要使用 `kotlin2js` 插件，而不是 `kotlin` 插件。

我们的 `build.gradle` 文件应该如下所示：

<div class="sample" markdown="1" theme="idea" mode="groovy">
```groovy
group 'org.example'
version '1.0-SNAPSHOT'

buildscript {
    ext.kotlin_version = '{{ site.data.releases.latest.version }}'
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

apply plugin: 'kotlin2js'

repositories {
    mavenCentral()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib-js:$kotlin_version"
}

```
</div>

如果要使用预览版（EAP, Early Access Preview），我们需要在 `ext.kotlin_version` 中指定其版本号，并且
在 `buildscript` 块中添加相应的仓库（预览版一般发布在 [Bintray](https://bintray.com/kotlin) 上）

在编译时，Gradle 会生成我们应用程序的输出，默认位于 `build/classes/main` 目录下，这个路径可以使用[编译器选项](#配置编译器选项)配置。

为了组装一个应用程序，我们还需要包含 Kotlin 标准库，即 `kotlin.js`，它作为依赖项包含在内，以及其他库（如果有的话）。

默认情况下，Gradle 不会在构建过程中解压 JAR 包，因此我们需要在构建中添加一个额外的步骤来执行此操作：

<div class="sample" markdown="1" theme="idea" mode="groovy">
```groovy
task assembleWeb(type: Sync) {
    configurations.compile.each { File file ->
        from(zipTree(file.absolutePath), {
            includeEmptyDirs = false
            include { fileTreeElement ->
                def path = fileTreeElement.path
                path.endsWith(".js") && (path.startsWith("META-INF/resources/") || 
                    !path.startsWith("META-INF/"))
            }
        })
    }
    from compileKotlin2Js.destinationDir
    into "${projectDir}/web"

    dependsOn classes
}

assemble.dependsOn assembleWeb
```
</div>

这个 task 会将运行时的依赖文件和编译输出一起复制到 `web` 目录。

有关生成的文件的更多信息和运行应用程序的说明，请参阅[Kotlin 转 JavaScript](../kotlin-to-javascript/kotlin-to-javascript.html)一节。

## 配置编译器选项

与使用[IntelliJ IDEA 构建系统](../getting-started-idea/getting-started-with-intellij-idea.html)或命令行类似，我们可以让编译器输出的 JavaScript 符合某个特定的模块系统的标准，例如 AMD、CommonJS 或 UMD。

要指定模块的类型，我们可以在插件中添加一条配置，如下所示：

<div class="sample" markdown="1" theme="idea" mode="groovy">
```groovy
compileKotlin2Js {
    kotlinOptions.outputFile = "${projectDir}/web/output.js"
    kotlinOptions.moduleKind = "amd"
    kotlinOptions.sourceMap = true
}
```
</div>

在这里，`moduleKind` 可以是：

* plain（默认）
* amd
* commonjs
* umd

关于不同类型模块输出的更多信息，请参阅[使用模块](../working-with-modules/working-with-modules.html)一节。

我们还可以看到如何通过配置 `sourceMap` 选项，指示编译器是否为我们生成源码映射文件。
