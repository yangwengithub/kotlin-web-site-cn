---
type: doc
layout: reference
category: "JavaScript"
title: "搭建 Kotlin/JS 项目"
---

# 搭建 Kotlin/JS 项目

Kotlin/JS 项目使用 Gradle 作为构建系统。为了开发者轻松管理其 Kotlin/JS 项目，我们提供了 Kotlin/JS Gradle 插件。该插件提供项目配置工具以及用以自动执行 JavaScript 开发中常用的例程的帮助程序。例如，该插件会下载 [Yarn](https://yarnpkg.com/) 软件包管理器，在后台管理NPM依赖，并使用[Webpack](https://webpack.js.org/)从Kotlin项目构建JavaScript包。

要在IntelliJ IDEA中创建 Kotlin/JS 项目，请转至 **File | New | Project**，并选择 **Gradle | Kotlin/JS for browser** 或 **Kotlin/JS for Node.js**。请确保未勾选清除**Java**复选框。
 
![New project wizard]({{ url_for('asset', path='images/reference/js-project-setup/wizard.png') }})

另外，您可以在`build.gradle`文件中手动将`org.jetbrains.kotlin.js`插件应用于Gradle项目。如果您使用Gradle Kotlin DSL，则可以使用插件`kotlin(“js”)`。

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>
 
```groovy
plugins {
    id 'org.jetbrains.kotlin.js' version '{{ site.data.releases.latest.version }}'
}
```
 
</div>
</div>
 
<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>
 
```kotlin
plugins {
     kotlin("js") version "{{ site.data.releases.latest.version }}"
}
 ```
 
</div>
</div>

The Kotlin/JS plugin lets you manage aspects of your project in the `kotlin` section of the build script.

<div class="sample" markdown="1" mode="groovy" theme="idea">

```groovy
kotlin {
    //...
}
```

</div>
 
在`kotlin`部分中，您可以管理以下方面：

* [选择执行环境](#选择执行环境): 浏览器 或 Node.js
* [管理依赖](#管理依赖): Maven 和 NPM
* [配置 run 任务](#配置-run-任务)
* [配置 test 任务](#配置-test-任务)
* [配置 webpack 绑定](#配置-webpack-绑定) 针对于浏览器项目
* [分发目标目录](#分发目标目录)

## 选择执行环境

Kotlin/JS项目可以针对两个不同的执行环境：

* Browser，用于浏览器中客户端脚本
* [Node.js](https://nodejs.org/)，用于在浏览器外部运行JavaScript代码，例如，用于服务器端脚本。

要定义 Kotlin/JS 项目的目标执行环境，请在 `target` 部分内部添加`browser {}` 或 `nodejs {}`。

<div class="sample" markdown="1" mode="groovy" theme="idea">

```groovy
kotlin {
    target {
        browser {
        }       
    }
}    
```

</div>

或者

<div class="sample" markdown="1" mode="groovy" theme="idea">

```groovy
kotlin.target.browser {     
}    
```

</div>

Kotlin/JS 插件会自动配置其任务，来与在所选环境工作。这项操作包括下载与安装运行和测试应用程序所需的依赖。因此，开发者无需额外配置即可构建，运行和测试简单项目。

## 管理依赖

就像其他任何的Gradle项目一样，Kotlin/JS 项目支持位于构建脚本的`dependencies`部分的传统的Gradle[依赖声明](https://docs.gradle.org/current/userguide/declaring_dependencies.html)。

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
dependencies {
    implementation 'org.example.myproject:1.1.0'
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
dependencies {
    implementation("org.example.myproject", "1.1.0")
}
```

</div>
</div>

Kotlin/JS Gradle插件还支持构建脚本的`kotlin`部分中特定`sourceSets`的依赖声明。

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

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

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
kotlin {
  sourceSets["main"].dependencies {
    implementation("org.example.myproject", "1.1.0")
  }
}
```

</div>
</div>


### Kotlin 标准库

所有Kotlin/JS项目都必须依赖Kotlin/JS [标准库](https://kotlinlang.org/api/latest/jvm/stdlib/)。如果您的项目包含用Kotlin编写的测试，则还应在[kotlin.test](https://kotlinlang.org/api/latest/kotlin.test/)库上添加此[依赖项](https://kotlinlang.org/api/latest/kotlin.test/)。

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
dependencies {
    implementation 'org.jetbrains.kotlin:kotlin-stdlib-js'
    testImplementation 'org.jetbrains.kotlin:kotlin-test-js'
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
dependencies {
    implementation(kotlin("stdlib-js"))
    testImplementation(kotlin("test-js"))
}
```

</div>
</div>

### NPM 依赖

在JavaScript中，管理依赖项的常用方法是[NPM](https://www.npmjs.com/)。它提供了最大的JavaScript模块公共[存储库](https://www.npmjs.com/)以及用于下载它们的工具。

Kotlin/JS插件使您可以在Gradle构建脚本中声明NPM依赖关系以及其他依赖关系，并自动执行其他所有操作。它安装了[Yarn](https://yarnpkg.com/lang/en/)程序包管理器，并使用它来将依赖项从NPM存储库下载到`node_modules`项目目录 ─── JavaScript 项目的NPM依赖项的一般位置。

要声明NPM依赖项，请将其名称和版本传递`npm()`函数给依赖项声明中的函数。

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
dependencies {
    implementation npm('react', '16.12.0')
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
dependencies {
    implementation(npm("react", "16.12.0"))
}
```

</div>
</div>

安装NPM依赖项后，您可以按照[在Kotlin中调用JS](http://www.kotlincn.net/docs/reference/js-interop.html)中所述在代码中使用其API 。

## 配置 run 任务

Kotlin/JS插件提供了一个运行任务，使您无需额外配置即可运行项目。运行Kotlin/JS项目，它使用[Webpack DevServer](https://webpack.js.org/configuration/dev-server/)。比如说，如果要自定义DevServer配置，请更改其端口，请使用Webpack配置文件。

要运行项目，请执行标准生命周期的`run`任务：

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
./gradlew run
```

</div>

要在浏览器中查看源文件更改而无需重新启动DevServer，请使用Gradle [连续构建(continuous build)](https://docs.gradle.org/current/userguide/command_line_interface.html#sec:continuous_build)：

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
./gradlew run --continuous
```

</div>

或者 

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
./gradlew run -t
```

</div>

## 配置 test 任务

Kotin/JS Gradle插件会自动为项目设置测试基础结构。对于浏览器项目，它将下载并安装具有其他必需依赖的[Karma](https://karma-runner.github.io/)测试运行程序；对于NodeJS项目，使用[Mocha](https://mochajs.org/)测试框架。

该插件还提供了有用的测试功能，例如：

* 原始地图生成
* 测试报告生成
* 在控制台中测试运行结果

默认情况下，该插件使用 [Headless Chrome](https://chromium.googlesource.com/chromium/src/+/lkgr/headless/README.md) 来运行浏览器测试。您还可以通过在构建脚本中的`useKarma`部分中添加相应的条目，从而在其他浏览器中运行它们 ：

<div class="sample" markdown="1" mode="groovy" theme="idea">

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

</div>

如果要跳过测试，请将`enabled = false`这一行添加到`testTask`中。

<div class="sample" markdown="1" mode="groovy" theme="idea">

```groovy
kotlin.target.browser {
    testTask {
        enabled = false
    }
}
```

</div>

要运行测试，请执行标准生命周期`check`任务：

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
./gradlew check
```

</div>

## 配置 Webpack 绑定

对于浏览器目标，Kotlin/JS插件使用众所周知的[Webpack](https://webpack.js.org/)模块捆绑器。为了配置项目捆绑，可以使用标准的Webpack配置文件。Webpack配置功能在其[文档](https://webpack.js.org/concepts/configuration/)中有很好的描述。对于Kotlin/JS项目，Webpack配置文件位于项目根目录下的`webpack.config.d`目录中。

为了构建可执行的JavaScript工件，Kotlin/JS插件包含`browserDevelopmentWebpack`以及`browserProductionWebpack`任务。

要使用Webpack构建项目工件，请执行Gradle任务`browserProductionWebpack`或`browserDevelopmentWebpack`：

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
./gradlew browserProductionWebpack
```

</div>

## 分发目标目录

默认情况下，Kotlin/JS项目构建的结果位于项目根目录下的`/build/distribution`目录中。

要为项目分发文件设置另一个位置，请在构建脚本中的`browser`里添加`distribution`，然后为它的`directory`属性赋值。运行项目构建任务后，Gradle会将输出包和项目资源一起保存在此位置。

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
kotlin.target.browser {
    distribution {
        directory = file("$projectDir/output/")
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
kotlin.target.browser {
    distribution {
        directory = File("$projectDir/output/")
    }
}
```

</div>
</div>
