---
type: doc
layout: reference
title: "使用 Gradle"
---

# 使用 Gradle

为了用 Gradle 构建 Kotlin，你应该[设置好 *kotlin-gradle* 插件](#插件和版本)，[将其应用](#针对-jvm)到你的项目中，并且[添加 *kotlin-stdlib* 依赖](#配置依赖)。这些操作也可以在 IntelliJ IDEA 中通过调用 Project action 中的 Tools \| Kotlin \| Configure Kotlin 自动执行。

## 插件和版本

Apply the Kotlin plugin by using [the Gradle plugins DSL](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block),
replacing the placeholder with one of the plugin names that can be found in further sections:

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
plugins {
    id 'org.jetbrains.kotlin.<...>' version '{{ site.data.releases.latest.version }}'
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
plugins {
    kotlin("<...>") version "{{ site.data.releases.latest.version }}"
}
```

</div>
</div>

Alternatively, apply plugin by adding the `kotlin-gradle-plugin` dependency to the build script classpath:

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea">

``` groovy
buildscript {
    repositories {
        mavenCentral()
    }

    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:{{ site.data.releases.latest.version }}"
    }
}

plugins {
    id "org.jetbrains.kotlin.<...>" version "{{ site.data.releases.latest.version }}"
}
```
</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>


```kotlin
buildscript {
    repositories {
            mavenCentral()
    }

    dependencies {
        classpath(kotlin("gradle-plugin", version = "{{ site.data.releases.latest.version }}'"))
    }
}
plugins {
    kotlin("<...>")
}
```
</div>
</div>

当通过 [Gradle 插件 DSL](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block) 与 [Gradle Kotlin DSL](https://github.com/gradle/kotlin-dsl) 使用 Kotlin Gradle 插件 1.1.1 及以上版本时，这不是必需的。

## Building Kotlin Multiplatform Projects

Using the `kotlin-multiplatform` plugin for building [multiplatform projects](multiplatform.html) is described in
[Building Multiplatform Projects with Gradle](building-mpp-with-gradle.html).

## 针对 JVM

To target the JVM, apply the Kotlin JVM plugin. Starting with Kotlin 1.1.1, the plugin can be applied using the [Gradle plugins DSL](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block):


<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only>

```groovy
plugins {
    id "org.jetbrains.kotlin.jvm" version "{{ site.data.releases.latest.version }}"
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
plugins {
    kotlin("jvm") version "{{ site.data.releases.latest.version }}"
}
```

</div>
</div>

The `version` should be literal in this block, and it cannot be applied from another build script.

Alternatively, you can use the older `apply plugin` approach:

<div class="sample" markdown="1" mode="groovy" theme="idea">

```groovy
apply plugin: 'kotlin'
```

</div>

It's not recommended to apply Kotlin plugins with `apply` in Gradle Kotlin DSL. The details are provided [below](#using-gradle-kotlin-dsl).

Kotlin 源代码可以与同一个文件夹或不同文件夹中的 Java 源代码混用。默认约定是使用不同的文件夹：

<div class="sample" markdown="1" mode="groovy" theme="idea" auto-indent="false">

```groovy
project
    - src
        - main (root)
            - kotlin
            - java
```

</div>

如果不使用默认约定，那么应该更新相应的 *sourceSets* 属性：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
sourceSets {
    main.kotlin.srcDirs += 'src/main/myKotlin'
    main.java.srcDirs += 'src/main/myJava'
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
sourceSets["main"].java.srcDir("src/main/myJava")
sourceSets["main"].withConvention(KotlinSourceSet::class) {
    kotlin.srcDir("src/main/myKotlin")
}
```

</div>
</div>

对于 Gradle Kotlin DSL，请改用 `java.sourceSets { …… }` 配置源集。

## 针对 JavaScript

当针对 JavaScript 时，须应用不同的插件：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only>

``` groovy
plugins {
    id 'kotlin2js' version '{{ site.data.releases.latest.version }}'
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
plugins {
    id("kotlin2js") version "{{ site.data.releases.latest.version }}"
}
```

</div>
</div>

Note that this way of applying the Kotlin/JS plugin requires adding the following code to Gradle settings file (`settings.gradle`):
<div class="sample" markdown="1" mode="groovy" theme="idea" auto-indent="false">

```groovy
pluginManagement {
    resolutionStrategy {
        eachPlugin {
            if (requested.id.id == "kotlin2js") {
                useModule("org.jetbrains.kotlin:kotlin-gradle-plugin:${requested.version}")
            }
        }
    }
}
```
</div>

这个插件只适用于 Kotlin 文件，因此建议将 Kotlin 和 Java 文件分开（如果是同一项目包含 Java 文件的情况）。与<!--
-->针对 JVM 一样，如果不使用默认约定，需要使用 *sourceSets* 来指定源代码文件夹：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
sourceSets {
    main.kotlin.srcDirs += 'src/main/myKotlin'
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
sourceSets["main"].withConvention(KotlinSourceSet::class) {
    kotlin.srcDir("src/main/myKotlin")
}
```

</div>
</div>

除了输出的 JavaScript 文件，该插件默认会创建一个带二进制描述符的额外 JS 文件。
如果你是构建其他 Kotlin 模块可以依赖的可重用库，那么该文件是必需的，并且应该与转换结果一起分发。
其生成由 `kotlinOptions.metaInfo` 选项控制：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
compileKotlin2Js {
    kotlinOptions.metaInfo = true
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
tasks {
    "compileKotlin2Js"(Kotlin2JsCompile::class)  {
        kotlinOptions.metaInfo = true
    }
}
```

</div>
</div>


## 针对 Android

Android 的 Gradle 模型与普通 Gradle 有点不同，所以如果我们要构建一个用 Kotlin 编写的 Android 项目，我们需要<!--
-->用 *kotlin-android* 插件取代 *kotlin* 插件：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
buildscript {
    ext.kotlin_version = '{{ site.data.releases.latest.version }}'

    ……

    dependencies {
        classpath 'com.android.tools.build:gradle:3.2.1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

plugins {
    id 'com.android.application'
    id 'kotlin-android'
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
buildscript {
    dependencies {
        classpath("com.android.tools.build:gradle:3.2.1")
        classpath(kotlin("gradle-plugin", version = "{{ site.data.releases.latest.version }}'"))
    }
}
plugins {
    id("com.android.application")
    id("kotlin-android")
}
```

</div>
</div>

不要忘记配置[标准库依赖关系](#配置依赖)。

### Android Studio

如果使用 Android Studio，那么需要在 android 下添加以下内容：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
android {
  ……

  sourceSets {
    main.java.srcDirs += 'src/main/kotlin'
  }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
android {
  ...

    sourceSets["main"].java.srcDir("src/main/kotlin")
}
```

</div>
</div>

这让 Android Studio 知道该 kotlin 目录是源代码根目录，所以当项目模型加载到 IDE 中时，它会被正确识别。或者，你可以将 Kotlin 类放在 Java 源代码目录中，该目录通常位于 `src/main/java`。


## 配置依赖

除了上面显示的 `kotlin-gradle-plugin` 依赖之外，还需要添加 Kotlin 标准库的依赖：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
repositories {
    mavenCentral()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib"
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
repositories {
    mavenCentral()
}

dependencies {
    compile(kotlin("stdlib"))
}
```

</div>
</div>

The Kotlin standard library `kotlin-stdlib` targets Java 6 and above. There are extended versions of the standard library that add support for some of the features of JDK 7 and JDK 8. To use these versions, add one of the
following dependencies instead of `kotlin-stdlib`:

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
compile "org.jetbrains.kotlin:kotlin-stdlib-jdk7"
compile "org.jetbrains.kotlin:kotlin-stdlib-jdk8"
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
compile(kotlin("stdlib-jdk7"))
compile(kotlin("stdlib-jdk8"))
```

</div>
</div>

在 Kotlin 1.1.x 中，请使用 `kotlin-stdlib-jre7` 与 `kotlin-stdlib-jre8`。

If you target JavaScript, use the `stdlib-js` dependency.

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
compile "org.jetbrains.kotlin:kotlin-stdlib-js"
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
compile(kotlin("stdlib-js"))
```

</div>
</div>

If your project uses [Kotlin reflection](/api/latest/jvm/stdlib/kotlin.reflect.full/index.html) or testing facilities, you need to add the corresponding dependencies as well:

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
compile "org.jetbrains.kotlin:kotlin-reflect"
testCompile "org.jetbrains.kotlin:kotlin-test"
testCompile "org.jetbrains.kotlin:kotlin-test-junit"
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
compile(kotlin("reflect"))
testCompile(kotlin("test"))
testCompile(kotlin("test-junit"))
```

</div>
</div>

从 Kotlin 1.1.2 起，使用 `org.jetbrains.kotlin` group 的依赖项默认使用<!--
-->从已应用的插件获得的版本来解析。你可以用完整的依赖关系助记符：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

 ```groovy
compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
 ```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

 ```kotlin
 compile(kotlin("stdlib", kotlinVersion))
 ```

</div>
</div>

## 注解处理

Kotlin supports annonation processing via the _Kotlin annotation processing tool_(`kapt`). Usage of kapt with Gradle is described on the [kapt page](kapt.html).

## 增量编译

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` groovy
archivesBaseName = 'myExampleProject_lib'
```
</div>

With Gradle Kotlin DSL, it is:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
setProperty("archivesBaseName", "myExampleProject_lib")
```
</div>

  1. Add `kotlin.incremental=true` or `kotlin.incremental=false` line either to a `gradle.properties` or to a `local.properties` file;

  2. Add `-Pkotlin.incremental=true` or `-Pkotlin.incremental=false` to Gradle command line parameters. Note that in this case the parameter should be added to each subsequent build, and any build with disabled incremental compilation invalidates incremental caches.

Note, that the first build won't be incremental.


## Gradle 构建缓存支持（自 1.2.20 起）

Kotlin 插件支持 [Gradle 构建缓存](https://guides.gradle.org/using-build-cache/)（需要 Gradle 4.3 及以上版本；低版本则禁用缓存）。

如需禁用所有 Kotlin 任务的缓存，请将系统属性标志 `kotlin.caching.enabled` 设置为 `false`（运行构建带上参数 `-Dkotlin.caching.enabled=false`）。

If you use [kapt](kapt.html), note that the kapt annotation processing tasks are not cached by default. However, you can enable caching for them manually. See the [kapt page](kapt.html#gradle-build-cache-support-since-1220) for details.

## 编译器选项

要指定附加的编译选项，请使用 Kotlin 编译任务的 `kotlinOptions` 属性。

当针对 JVM 时，对于生产代码这些任务称为 `compileKotlin` 而对于<!--
-->测试代码称为 `compileTestKotlin`。对于自定义源文件集（source set）这些任务称呼取决于 `compile＜Name＞Kotlin` 模式。

Android 项目中的任务名称包含[构建变体](https://developer.android.com/studio/build/build-variants.html) 名称，并遵循 `compile<BuildVariant>Kotlin` 的模式，例如 `compileDebugKotlin`、 `compileReleaseUnitTestKotlin`。

当针对 JavaScript 时，这些任务分别称为 `compileKotlin2Js` 与 `compileTestKotlin2Js`，以及对于自定义源文件集称为 `compile＜Name＞Kotlin2Js`。

要配置单个任务，请使用其名称。示例：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
compileKotlin {
    kotlinOptions.suppressWarnings = true
}

//or

compileKotlin {
    kotlinOptions {
        suppressWarnings = true
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile
// ……

val compileKotlin: KotlinCompile by tasks

compileKotlin.kotlinOptions.suppressWarnings = true
```

</div>
</div>

请注意，对于 Gradle Kotlin DSL，首先从项目的 `tasks` 中获取任务。

相应地，为 JS 与 Common 目标使用类型 `Kotlin2JsCompile` 与 `KotlinCompileCommon`。

也可以在项目中配置所有 Kotlin 编译任务：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
tasks.withType(org.jetbrains.kotlin.gradle.tasks.KotlinCompile::class.java).all {
    kotlinOptions { …… }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
tasks.withType<KotlinCompile> {
    kotlinOptions.suppressWarnings = true
}
```

</div>
</div>

对于 Gradle 任务的完整选项列表如下：

### JVM、JS 与 JS DCE 的公共属性

| 名称 | 描述        | 可能的值        |默认值        |
|------|-------------|-----------------|--------------|
| `allWarningsAsErrors` | 任何警告都报告为错误 |  | false |
| `suppressWarnings` | 不生成警告 |  | false |
| `verbose` | 启用详细日志输出 |  | false |
| `freeCompilerArgs` | 附加编译器参数的列表 |  | [] |

### JVM 与 JS 的公共属性

| Name | Description | Possible values |Default value |
|------|-------------|-----------------|--------------|
| `apiVersion` | 只允许使用来自捆绑库的指定版本中的声明 | "1.0"、 "1.1"、 "1.2"、 "1.3"、 "1.4 (EXPERIMENTAL)" |  |
| `languageVersion` | 提供与指定语言版本源代码兼容性 | "1.0"、 "1.1"、 "1.2"、 "1.3"、 "1.4 (EXPERIMENTAL)" |  |

### JVM 特有的属性

| 名称 | 描述        | 可能的值        |默认值        |
|------|-------------|-----------------|--------------|
| `javaParameters` | 为方法参数生成 Java 1.8 反射的元数据 |  | false |
| `jdkHome` | 要包含到 classpath 中的 JDK 主目录路径，如果与默认 JAVA_HOME 不同的话 |  |  |
| `jvmTarget` | 生成的 JVM 字节码的目标版本（1.6 或 1.8），默认为 1.6 | "1.6"、 "1.8" | "1.6" |
| `noJdk` | 不要在 classpath 中包含 Java 运行时 |  | false |
| `noReflect` | 不要在 classpath 中包含 Kotlin 反射实现 |  | true |
| `noStdlib` | 不要在 classpath 中包含 Kotlin 运行时 |  | true |

### JS 特有的属性

| 名称 | 描述        | 可能的值        |默认值        |
|------|-------------|-----------------|--------------|
| `friendModulesDisabled` | 禁用内部声明导出 |  | false |
| `main` | 是否要调用 main 函数 | "call"、 "noCall" | "call" |
| `metaInfo` | 使用元数据生成 .meta.js 与 .kjsm 文件。用于创建库 |  | true |
| `moduleKind` | 编译器生成的模块类型 | "plain"、 "amd"、 "commonjs"、 "umd" | "plain" |
| `noStdlib` | 不使用捆绑的 Kotlin stdlib |  | true |
| `outputFile` | 输出文件路径 |  |  |
| `sourceMap` | 生成源代码映射（source map） |  | false |
| `sourceMapEmbedSources` | 将源代码嵌入到源代码映射中 | "never"、 "always"、 "inlining" | |
| `sourceMapPrefix` | 源代码映射中路径的前缀 |  |  |
| `target` | 生成指定 ECMA 版本的 JS 文件 | "v5" | "v5" |
| `typedArrays` | 将原生数组转换为 JS 带类型数组 |  | true |


## 生成文档

要生成 Kotlin 项目的文档，请使用 [Dokka](https://github.com/Kotlin/dokka)；
相关配置说明请参见 [Dokka README](https://github.com/Kotlin/dokka/blob/master/README.md#using-the-maven-plugin)
。Dokka 支持混合语言项目，并且可以生成多种格式的输出
，包括标准 JavaDoc。

## OSGi

关于 OSGi 支持请参见 [Kotlin OSGi 页](kotlin-osgi.html)。

## 使用 Gradle Kotlin DSL

使用 [Gradle Kotlin DSL](https://github.com/gradle/kotlin-dsl) 时，请使用 `plugins { …… }` 块应用 Kotlin 插件。如果使用 `apply { plugin(……) }` 来应用的话，可能会遇到未解析的到由 Gradle Kotlin DSL 所生成扩展的引用问题。为了解决这个问题，可以注释掉出错的用法，运行 Gradle 任务 `kotlinDslAccessorsSnapshot`，然后解除该用法注释并重新运行构建或者重新将项目导入到 IDE 中。

## 示例

以下示例显示了配置 Gradle 插件的不同可能性：

* [Kotlin](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/hello-world)
* [混用 Java 与 Kotlin](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/mixed-java-kotlin-hello-world)
* [Android](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/android-mixed-java-kotlin-project)
* [JavaScript](https://github.com/JetBrains/kotlin/tree/master/libraries/tools/kotlin-gradle-plugin-integration-tests/src/test/resources/testProject/kotlin2JsProject)
