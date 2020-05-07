---
type: doc
layout: reference
category: "JavaScript"
title: "搭建 Kotlin/JS 项目"
---

# 搭建 Kotlin/JS 项目

Kotlin/JS 项目使用 Gradle 作为构建系统。为了开发者轻松管理其 Kotlin/JS 项目，我们提供了 Kotlin/JS Gradle 插件。
该插件提供项目配置工具以及用以自动执行 JavaScript 开发中常用的例程的帮助程序。
举个例子，该插件会下载 [Yarn](https://yarnpkg.com/) 软件包管理器，在后台管理 [npm](https://www.npmjs.com/) 依赖，并使用
[webpack](https://webpack.js.org/) 从 Kotlin 项目构建 JavaScript 包。

要在 IntelliJ IDEA 中创建 Kotlin/JS 项目，请转至 **File | New | Project**，
并选择 **Gradle | Kotlin/JS for browser** 或 **Kotlin/JS for Node.js**。请不要勾选 **Java** 复选框。

![New project wizard](/assets/images/reference/js-project-setup/wizard.png)


另外，你可以在 Gradle build file (`build.gradle` or `build.gradle.kts`) 中手动将 `org.jetbrains.kotlin.js` 插件应用于 Gradle 项目。
如果你使用 Gradle Kotlin DSL，则可以使用插件 `kotlin(“js”)`。



> Groovy DSL
```groovy
plugins {
    id 'org.jetbrains.kotlin.js' version '{{ site.data.releases.latest.version }}'
}
```






> Kotlin DSL
```kotlin
plugins {
     kotlin("js") version "{{ site.data.releases.latest.version }}"
}
```




Kotlin/JS 插件可让你在构建脚本的 `kotlin` 部分中管理项目的各个方面。



```groovy
kotlin {
    //...
}
```



在 `kotlin` 部分中，你可以管理以下方面：

* [选择执行环境](#选择执行环境): 浏览器或 Node.js
* [管理依赖](#管理依赖): Maven 和 npm
* [配置 run 任务](#配置-run-任务)
* [配置 test 任务](#配置-test-任务)
* [配置 webpack 绑定](#配置-webpack-绑定) 针对于浏览器项目
* [分发目标目录](#分发目标目录)

## 选择执行环境

Kotlin/JS 项目可以针对两个不同的执行环境：

* Browser，用于浏览器中客户端脚本
* [Node.js](https://nodejs.org/)，用于在浏览器外部运行 JavaScript 代码，例如，用于服务器端脚本。

要定义 Kotlin/JS 项目的目标执行环境，请在 `target` 部分添加 `browser {}` 或 `nodejs {}`。



```groovy
kotlin {
    target {
        browser {
        }       
    }
}    
```



或者

> Groovy DSL

```groovy
kotlin.target.browser {     
}    
```



Kotlin/JS 插件会自动配置其任务与所选环境配合工作。
这项操作包括下载与安装运行和测试应用程序所需的依赖。
因此，开发者无需额外配置就可以构建、运行和测试简单项目。

## 管理依赖

就像其他任何的 Gradle 项目一样，Kotlin/JS 项目支持位于构建脚本的 `dependencies` 
部分的传统 Gradle [依赖声明](https://docs.gradle.org/current/userguide/declaring_dependencies.html)。


> Groovy DSL

```groovy
dependencies {
    implementation 'org.example.myproject:1.1.0'
}
```





> Kotlin DSL

```kotlin
dependencies {
    implementation("org.example.myproject", "1.1.0")
}
```




Kotlin/JS Gradle 插件还支持构建脚本的 `kotlin` 部分中特定 
`sourceSets` 的依赖声明。


> Groovy DSL

```groovy
kotlin {
    sourceSets {
        main {
            dependencies {
                implementation 'org.example.myproject:1.1.0'
            }
        }
    }
}
```





> Kotlin DSL

```kotlin
kotlin {
  sourceSets["main"].dependencies {
    implementation("org.example.myproject", "1.1.0")
  }
}
```





### Kotlin 标准库

所有 Kotlin/JS 项目都必须依赖 Kotlin/JS [标准库](https://kotlinlang.org/api/latest/jvm/stdlib/)。
如果你的项目包含用 Kotlin 编写的测试，则还应在添加 [kotlin.test](https://kotlinlang.org/api/latest/kotlin.test/) 
[依赖项](https://kotlinlang.org/api/latest/kotlin.test/)。


> Groovy DSL

```groovy
dependencies {
    implementation 'org.jetbrains.kotlin:kotlin-stdlib-js'
    testImplementation 'org.jetbrains.kotlin:kotlin-test-js'
}
```





> Kotlin DSL

```kotlin
dependencies {
    implementation(kotlin("stdlib-js"))
    testImplementation(kotlin("test-js"))
}
```




### npm 依赖

在 JavaScript 中，管理依赖项的常用方法是 [npm](https://www.npmjs.com/)。
它提供了最大的 JavaScript 模块公共[存储库](https://www.npmjs.com/)以及用于下载它们的工具。

Kotlin/JS 插件使你可以在 Gradle 构建脚本中声明 npm 依赖项以及其他依赖项，并自动执行其他所有操作。
它安装了 [Yarn](https://yarnpkg.com/lang/en/) 程序包管理器，
并使用它来将依赖项从 npm 存储库下载项目的 `node_modules` 目录
─── JavaScript 项目的 npm 依赖项的一般位置。

要声明 npm 依赖项，将其名称与版本传给依赖项声明内的 `npm()` 函数。


> Groovy DSL

```groovy
dependencies {
    implementation npm('react', '16.12.0')
}
```





> Kotlin DSL

```kotlin
dependencies {
    implementation(npm("react", "16.12.0"))
}
```




安装 npm 依赖项后，你可以按照[在 Kotlin 中调用 JS](http://www.kotlincn.net/docs/reference/js-interop.html) 
中所述，在代码中使用其 API。

## 配置 run 任务

Kotlin/JS 插件提供了一个 `run` 任务，使你无需额外配置即可运行项目。
它使用 [webpack DevServer](https://webpack.js.org/configuration/dev-server/) 来运行 Kotlin/JS 项目。
如果要自定义 DevServer 配置，请更改其端口，请使用 webpack 配置文件。

要运行项目，请执行标准生命周期的 `run` 任务：



```bash
./gradlew run
```



要在浏览器中查看源文件更改而不想重新启动 DevServer，请使用 Gradle 
[连续构建（continuous build）](https://docs.gradle.org/current/userguide/command_line_interface.html#sec:continuous_build)：



```bash
./gradlew run --continuous
```



或者 



```bash
./gradlew run -t
```



## 配置 test 任务

Kotin/JS Gradle 插件会自动为项目设置测试基础结构。对于浏览器项目，它将下载并安装具有其他必需依赖的 
[Karma](https://karma-runner.github.io/)  测试运行程序；
对于 NodeJS 项目，使用 [Mocha](https://mochajs.org/) 测试框架。

该插件还提供了有用的测试功能，例如：

* 源代码映射文件生成
* 测试报告生成
* 在控制台中测试运行结果

默认情况下，该插件使用 
[Headless Chrome](https://chromium.googlesource.com/chromium/src/+/lkgr/headless/README.md) 
来运行浏览器测试。你还可以通过在构建脚本中的 `useKarma` 部分中添加相应的条目，从而在其他浏览器中运行它们 ：



```groovy
kotlin.target.browser {
    testTask {
        useKarma {
            useIe()
            useSafari()
            useFirefox()
            useChrome()
            useChromeCanary()
            useChromeHeadless()
            usePhantomJS()
            useOpera()
        }
    }       
}
```



如果要跳过测试，请将 `enabled = false` 这一行添加到 `testTask` 中。



```groovy
kotlin.target.browser {
    testTask {
        enabled = false
    }
}
```



要运行测试，请执行标准生命周期 `check` 任务：



```bash
./gradlew check
```



## 配置 Webpack 绑定

对于浏览器目标，Kotlin/JS 插件使用众所周知的 [Webpack](https://webpack.js.org/) 模块捆绑器。

The Kotlin/JS Gradle plugin automatically generates a standard webpack configuration file 
at build time which you can find the at `build/js/packages/projectName/webpack.config.js`.

The most common webpack adjustments can be made directly via the
`kotlin.target.browser.webpackTask` configuration block in the Gradle build file.

If you want to make further adjustments to the webpack configuration, place your additional configuration files inside a directory
called `webpack.config.d` in the root of your project. When building your project, all JS configuration files will automatically
be merged into the `build/js/packages/projectName/webpack.config.js` file.
To add a new [webpack loader](https://webpack.js.org/loaders/), for example, add the following to
a `.js` file inside the `webpack.config.d`:



```javascript
config.module.rules.push({
    test: /\.extension$/,
    loader: 'loader-name'
});
```



All webpack configuration
capabilities are well described in its [documentation](https://webpack.js.org/concepts/configuration/).

为了通过 webpack 构建可执行的 JavaScript 构件，Kotlin/JS 插件包含 `browserDevelopmentWebpack` 以及
`browserProductionWebpack` Gradle 任务。 Execute them to obtain artifacts for development or production respectively:



```bash
./gradlew browserProductionWebpack
```



## Configuring Yarn

To configure additional Yarn features, place a `.yarnrc` file in the root of your project.
At build time, it gets picked up automatically.

For example, to use a custom registry for npm packages, add the following line to a file called
`.yarnrc` in the project root:



```
registry "http://my.registry/api/npm/"
```



To learn more about `.yarnrc`, please visit the [official Yarn documentation](https://classic.yarnpkg.com/en/docs/yarnrc/).

## 分发目标目录

默认情况下，Kotlin/JS 项目构建的结果位于项目根目录下的 `/build/distribution` 目录中。

要为项目分发文件设置另一个位置，请在构建脚本中的 `browser` 里添加 `distribution`，然后<!--
-->为它的 `directory` 属性赋值。
运行项目构建任务后，Gradle 会将输出的内容<!--
-->和项目资源一起保存在此位置。


> Groovy DSL

```groovy
kotlin.target.browser {
    distribution {
        directory = file("$projectDir/output/")
    }
}
```





> Kotlin DSL

```kotlin
kotlin.target.browser {
    distribution {
        directory = File("$projectDir/output/")
    }
}
```



