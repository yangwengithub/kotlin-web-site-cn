---
type: doc
layout: reference
title: "使用 Gradle"
---

# 使用 Gradle

为了用 Gradle 构建 Kotlin 项目，需要[设置好 *kotlin-gradle* 插件](#插件与版本)，[将其应用](#面向-jvm)到你的项目中，并且[添加 *kotlin-stdlib* 依赖](#配置依赖)。
这些操作也可以在 IntelliJ IDEA 中通过调用 **Project action** 中的 **Tools \| Kotlin \| Configure Kotlin** 自动执行。

## 插件与版本

使用 [Gradle 插件 DSL](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block) 应用 Kotlin Gradle 插件。
Kotlin Gradle 插件 {{ site.data.releases.latest.version }} 适用于 Gradle 4.9 及更高版本。

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
plugins {
    id 'org.jetbrains.kotlin.＜……＞' version '{{ site.data.releases.latest.version }}'
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
plugins {
    kotlin("＜……＞") version "{{ site.data.releases.latest.version }}"
}
```

</div>
</div>

需要将其中的占位符 `＜……＞` 替换为可在后续部分中找到的插件名之一。

或者通过将依赖项 `kotlin-gradle-plugin` 添加到构建脚本类路径来应用插件：

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
    id "org.jetbrains.kotlin.＜……＞" version "{{ site.data.releases.latest.version }}"
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
        classpath(kotlin("gradle-plugin", version = "{{ site.data.releases.latest.version }}"))
    }
}
plugins {
    kotlin("＜……＞")
}
```
</div>
</div>

当通过 [Gradle 插件 DSL](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block) 与 [Gradle Kotlin DSL](https://github.com/gradle/kotlin-dsl) 使用 Kotlin Gradle 插件 1.1.1 及以上版本时，这不是必需的。

## 构建 Kotlin 多平台项目

使用 `kotlin-multiplatform` 插件构建[多平台项目](multiplatform.html)在<!--
-->[以 Gradle 构建多平台项目](building-mpp-with-gradle.html)中详述。

## 面向 JVM

如需面向 JVM 平台，请应用 Kotlin JVM 插件。自 Kotlin 1.1.1 起，可以使用 [Gradle 插件 DSL](https://docs.gradle.org/current/userguide/plugins.html#sec:plugins_block) 来应用该插件：


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

在这个代码块中的 `version` 应该是字面值，并且不能在其他构建脚本中应用。

或者使用旧版 `apply plugin` 方式：

<div class="sample" markdown="1" mode="groovy" theme="idea">

```groovy
apply plugin: 'kotlin'
```

</div>

不建议在 Gradle Kotlin DSL 中以 `apply` 的方式应用 Kotlin 插件。详见[下文](#使用-gradle-kotlin-dsl)。

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

## 面向 JavaScript

当面向 JavaScript 时，须应用不同的插件：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only>

``` groovy
plugins {
    id 'org.jetbrains.kotlin.js' version '{{ site.data.releases.latest.version }}'
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
plugins {
    kotlin("js") version "{{ site.data.releases.latest.version }}"
}
```

</div>
</div>

这个插件只适用于 Kotlin 文件，因此建议将 Kotlin 和 Java 文件分开（如果是同一项目包含 Java 文件的情况）。与<!--
-->面向 JVM 一样，如果不使用默认约定，需要使用 *sourceSets* 来指定源代码文件夹：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
kotlin {
    sourceSets {
        main.kotlin.srcDirs += 'src/main/myKotlin'
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
kotlin {
    sourceSets["main"].apply {    
        kotlin.srcDir("src/main/myKotlin") 
    }
}
```

</div>
</div>


## 面向 Android

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
        classpath(kotlin("gradle-plugin", version = "{{ site.data.releases.latest.version }}"))
    }
}
plugins {
    id("com.android.application")
    kotlin("android")
}
```

</div>
</div>

Kotlin Gradle 插件 {{ site.data.releases.latest.version }} 适用于 Android Gradle 插件 3.0 及更高版本。

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
  ……

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
    implementation "org.jetbrains.kotlin:kotlin-stdlib"
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
    implementation(kotlin("stdlib"))
}
```

</div>
</div>

Kotlin 标准库 `kotlin-stdlib` 面向 Java 6 及以上版本。还有扩展版本的标准库，增加了对 JDK 7 与 JDK 8 中某些特性支持。如需使用这些版本，请添加<!--
-->下列依赖之一而不是 `kotlin-stdlib`：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7"
implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8"
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
implementation(kotlin("stdlib-jdk7"))
implementation(kotlin("stdlib-jdk8"))
```

</div>
</div>

在 Kotlin 1.1.x 中，请使用 `kotlin-stdlib-jre7` 与 `kotlin-stdlib-jre8`。

如果是面向 JavaScript，请使用依赖项 `stdlib-js` 。

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
implementation "org.jetbrains.kotlin:kotlin-stdlib-js"
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
implementation(kotlin("stdlib-js"))
```

</div>
</div>

如果项目中用到了 [Kotlin 反射](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect.full/index.html)或者测试设施，还需要添加相应的依赖项：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
implementation "org.jetbrains.kotlin:kotlin-reflect"
testImplementation "org.jetbrains.kotlin:kotlin-test"
testImplementation "org.jetbrains.kotlin:kotlin-test-junit"
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
implementation(kotlin("reflect"))
testImplementation(kotlin("test"))
testImplementation(kotlin("test-junit"))
```

</div>
</div>

自 Kotlin 1.1.2 起，使用 `org.jetbrains.kotlin` group 的依赖项默认使用<!--
-->从已应用的插件获得的版本来解析。你可以用完整的依赖关系助记符：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

 ```groovy
implementation "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
 ```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

 ```kotlin
 implementation(kotlin("stdlib", kotlinVersion))
 ```

</div>
</div>

## 注解处理

Kotlin 通过 *Kotlin 注解处理工具*（`kapt`）支持注解处理。kapt 同 Gradle 合用的用法已在 [kapt 页](kapt.html)描述。

## 增量编译

Kotlin Gradle 插件支持支持增量编译。增量编译会跟踪多次构建之间源文件的变更，因此只会编译这些变更所影响的文件。

Kotlin/JVM 与 Kotlin/JS 项目均支持增量编译。对于 Kotlin 1.1.1 起的 Kotlin/JVM 项目以及自 Kotlin 1.3.20 起的 Kotlin/JS 项目默认启用增量编译。

有几种方法可以覆盖默认设置：

* 在 Gradle 配置文件中：在 `gradle.properties` 或者 `local.properties` 中，对于 Kotlin/JVM 项目添加一行 `kotlin.incremental=＜值＞`，对于 Kotlin/JS 项目添加一行 `kotlin.incremental.js=＜值＞`。`＜值＞` 是一个反应增量编译用法的布尔值。

* 在 Gradle 命令行参数中：添加带有反应增量编译用法的布尔值的 `-Pkotlin.incremental` or `-Pkotlin.incremental.js` 参数。请注意，这样用法中，该参数必须添加到后续每个子构建，并且任何具有禁用增量编译的构建将使增量缓存失效。

请注意，任何情况下首次构建都不会是增量的。


## Gradle 构建缓存支持（自 1.2.20 起）

Kotlin 插件支持 [Gradle 构建缓存](https://guides.gradle.org/using-build-cache/)（需要 Gradle 4.3 及以上版本；低版本则禁用缓存）。

如需禁用所有 Kotlin 任务的缓存，请将系统属性标志 `kotlin.caching.enabled` 设置为 `false`（运行构建带上参数 `-Dkotlin.caching.enabled=false`）。

如果使用 [kapt](kapt.html)，请注意默认情况下不会缓存注解处理任务。不过，可以手动为它们启用缓存。详见 [kapt 页](kapt.html#gradle-构建缓存支持自-1220-起)。

## 编译器选项

要指定附加的编译选项，请使用 Kotlin 编译任务的 `kotlinOptions` 属性。

当面向 JVM 时，对于生产代码这些任务称为 `compileKotlin` 而对于<!--
-->测试代码称为 `compileTestKotlin`。对于自定义源文件集（source set）这些任务称呼取决于 `compile＜Name＞Kotlin` 模式。

Android 项目中的任务名称包含[构建变体](https://developer.android.com/studio/build/build-variants.html) 名称，并遵循 `compile<BuildVariant>Kotlin` 的模式，例如 `compileDebugKotlin`、 `compileReleaseUnitTestKotlin`。

当面向 JavaScript 时，这些任务分别称为 `compileKotlin2Js` 与 `compileTestKotlin2Js`，以及对于自定义源文件集称为 `compile＜Name＞Kotlin2Js`。

要配置单个任务，请使用其名称。示例：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" mode="groovy" theme="idea" data-lang="groovy">

```groovy
compileKotlin {
    kotlinOptions.suppressWarnings = true
}

//或者

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
tasks.withType(org.jetbrains.kotlin.gradle.tasks.KotlinCompile).all {
    kotlinOptions { ... }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-lang="kotlin" data-highlight-only>

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

tasks.withType<KotlinCompile>().configureEach {
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
| `languageVersion` | 提供与指定 Kotlin 版本源代码级兼容 | "1.0"、 "1.1"、 "1.2"、 "1.3"、 "1.4 (EXPERIMENTAL)" |  |

### JVM 特有的属性

| 名称 | 描述        | 可能的值        |默认值        |
|------|-------------|-----------------|--------------|
| `javaParameters` | 为方法参数生成 Java 1.8 反射的元数据 |  | false |
| `jdkHome` | 将来自指定位置的自定义 JDK 而不是默认的 JAVA_HOME 包含到类路径中 |  |  |
| `jvmTarget` | 生成的 JVM 字节码的目标版本（1.6、 1.8、 9、 10、 11、 12 或 13），默认为 1.6 | "1.6"、 "1.8"、 "9"、 "10"、 "11"、 "12"、 "13" | "1.6" |
| `noJdk` | 不要自动在类路径中包含 Java 运行时 |  | false |
| `noReflect` | 不要自动在类路径中包含 Kotlin 反射实现 |  | true |
| `noStdlib` | 不要自动在类路径中包含 Kotlin 运行时与 Kotlin 反射 |  | true |

### JS 特有的属性

| 名称 | 描述        | 可能的值        |默认值        |
|------|-------------|-----------------|--------------|
| `friendModulesDisabled` | 禁用内部声明导出 |  | false |
| `main` | 定义是否在执行时调用 `main` 函数 | "call"、 "noCall" | "call" |
| `metaInfo` | 使用元数据生成 .meta.js 与 .kjsm 文件。用于创建库 |  | true |
| `moduleKind` | 编译器生成的 JS 模块类型 | "plain"、 "amd"、 "commonjs"、 "umd" | "plain" |
| `noStdlib` | 不要自动将默认的 Kotlin/JS stdlib 包含到编译依赖项中 |  | true |
| `outputFile` | 编译结果的目标 *.js 文件 |  |  |
| `sourceMap` | 生成源代码映射（source map） |  | false |
| `sourceMapEmbedSources` | 将源代码嵌入到源代码映射中 | "never"、 "always"、 "inlining" | |
| `sourceMapPrefix` | 将指定前缀添加到源代码映射中的路径 |  |  |
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
