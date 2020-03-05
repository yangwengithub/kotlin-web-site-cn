---
type: tutorial
layout: tutorial
title:  "以 Maven 入门 Kotlin 和 JavaScript"
description: "本文介绍如何使用 Maven 把 Kotlin 编译到 JavaScript"
authors: Hadi Hariri，刘文俊（翻译）
date: 2016-11-04
showAuthorInfo: true
---

>__Warning__: this tutorial is outdated for Kotlin {{ site.data.releases.latest.version }}.
>We strongly recommend using Gradle for Kotlin/JS projects. For instructions on creating 
>Kotlin/JS projects with Gradle, see [Setting up a Kotlin/JS project](../setting-up.html)
{:.note}
>

* [使用 Maven 创建编译到 JavaScript 的应用程序](#创建编译到-javascript-的应用程序)
* [配置编译器选项](#配置编译器选项)


## 创建编译到 JavaScript 的应用程序


### 自动配置

创建使用 Maven 的编译到 JavaScript 的新程序，最简单的方法是让 IntelliJ IDEA <!--
-->为我们配置 Maven 项目。只需在 IntelliJ IDEA 中创建一个新的 Maven 项目，创建完成后，添加一个新的<!--
-->文件夹来保存 Kotlin 源代码，删除默认的 Java 文件夹。最终该项目应具有如下结构：
 
![Project Structure]({{ url_for('tutorial_img', filename='javascript/getting-started-maven/project-structure.png')}})

我们现在可以添加第一个 Kotlin 源代码文件，IntelliJ IDEA 会提示我们配置项目的 Kotlin 环境，这时我们应该选择编译到 <!--
-->JavaScript。


![Configure Kotlin]({{ url_for('tutorial_img', filename='javascript/getting-started-maven/configure-kotlin.png')}})


IntelliJ IDEA 将会为我们添加 [Maven 配置](#maven-配置)中的相关条目。


### 手动配置

如果我们不使用 IntelliJ IDEA，我们可以通过添加以下条目手动配置 `pom.xml` 文件以编译到 JavaScript：


#### Maven 配置

<div class="sample" markdown="1" theme="idea" mode="xml" auto-indent="false">

```xml
<properties>
    <kotlin.version>{{ site.data.releases.latest.version }}</kotlin.version> 
</properties>

<dependencies>
    <dependency>
        <groupId>org.jetbrains.kotlin</groupId>
        <artifactId>kotlin-stdlib-js</artifactId>
        <version>${kotlin.version}</version>
    </dependency>
</dependencies>

<build>
    <sourceDirectory>src/main/kotlin</sourceDirectory>
    <plugins>
        <plugin>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-maven-plugin</artifactId>
            <version>${kotlin.version}</version>
            <executions>
                <execution>
                    <id>compile</id>
                    <phase>compile</phase>
                    <goals>
                        <goal>js</goal>
                    </goals>
                </execution>
                <execution>
                    <id>test-compile</id>
                    <phase>test-compile</phase>
                    <goals>
                        <goal>test-js</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>

```
</div>

在编译时，Maven 将产生以下输出：

![Maven Output]({{ url_for('tutorial_img', filename='javascript/getting-started-maven/maven-output.png')}})

在这里我们可以看到我们的应用程序的输出，即 `kotlinjs-maven.js` 文件。

为了使用它，我们还需要在我们的应用程序中包含 Kotlin 的标准库，即 `kotlin.js`，它作为依赖项包含在内。默认情况下，<!--
-->Maven 不会在构建过程中解压 JAR 包，因此我们需要在构建中添加一个额外的步骤来执行此操作。

<div class="sample" markdown="1" theme="idea" mode="xml" auto-indent="false">

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <executions>
        <execution>
            <id>unpack</id>
            <phase>compile</phase>
            <goals>
                <goal>unpack</goal>
            </goals>
            <configuration>
                <artifactItems>
                    <artifactItem>
                        <groupId>org.jetbrains.kotlin</groupId>
                        <artifactId>kotlin-stdlib-js</artifactId>
                        <version>${kotlin.version}</version>
                        <outputDirectory>${project.build.directory}/js/lib</outputDirectory>
                        <includes>*.js</includes>
                    </artifactItem>
                </artifactItems>
            </configuration>
        </execution>
    </executions>
</plugin>
```
</div>

有关生成的文件的更多信息，请参阅 [Kotlin 转 JavaScript](../kotlin-to-javascript/kotlin-to-javascript.html)一节。

## 配置编译器选项

与使用 [IntelliJ IDEA 构建系统](../getting-started-idea/getting-started-with-intellij-idea.html)或命令行类似，我们可以让编译器输出的 JavaScript 符合某个特定的模块系统的标准，例如 AMD、CommonJS 或 UMD。

为了指定模块的类型，我们可以在插件中添加一条配置，如下所示：

<div class="sample" markdown="1" theme="idea" mode="xml" auto-indent="false">

```xml
 </executions>
 ...
 <configuration>
        <moduleKind>commonjs</moduleKind>
        <sourceMap>true</sourceMap>
 </configuration>

```
</div>

在这里，`moduleKind` 可以是：

* plain（默认）
* amd
* commonjs
* umd

关于不同类型模块输出的更多信息，请参阅[使用模块](../working-with-modules/working-with-modules.html)一节。

我们还可以看到如何通过配置 `sourceMap` 选项，指示编译器是否为我们生成源码映射文件。


