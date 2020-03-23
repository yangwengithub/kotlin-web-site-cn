---
type: doc
layout: reference
title: "使用 Gradle 构建多平台项目"
---

# 使用 Gradle 构建多平台项目

> 多平台项目是 Kotlin 1.2 与 Kotlin 1.3 中的一个实验性特性。本文档中描述的所有语言<!--
-->与工具特性在未来的 Kotlin 版本中都可能会有所变化。
{:.note}

本文档解释了 [Kotlin 多平台项目](multiplatform.html)的结构，并描述了如何<!--
-->使用 Gradle 配置与构建这些项目。

## 目录

* [项目结构](#项目结构)
* [搭建一个多平台项目](#搭建一个多平台项目)
* [Gradle 插件](#gradle-插件)
* [设置目标](#设置目标)
    * [已支持平台](#已支持平台)
    * [配置编译项](#配置编译项)
* [配置源集](#配置源集)
    * [关联源集](#关联源集)
    * [添加依赖](#添加依赖)
    * [语言设置](#语言设置)
* [默认项目布局](#默认项目布局)
* [运行测试](#运行测试)
* [发布多平台库](#发布多平台库)
    * [实验性的元数据发布模式](#实验性的元数据发布模式)
    * [目标消歧义](#目标消歧义)
* [JVM 目标平台中的 Java 支持](#jvm-目标平台中的-java-支持)
* [Android 支持](#android-支持)
    * [发布 Android 库](#发布-android-库)
* [使用 Kotlin/Native 目标平台](#使用-kotlinnative-目标平台)
    * [目标快捷方式](#目标快捷方式)
    * [构建最终原生二进制文件](#构建最终原生二进制文件)

## 项目结构

Kotlin 多平台项目的布局由以下构建块构成：

* [目标](#设置目标)是构建的一部分，负责构建、测试<!--
-->及打包其中一个平台的完整软件。因此，多平台项目通常包含<!--
-->多个目标。

* 构建每个目标涉及一到多次编译 Kotlin 源代码。换句话说，一个目标可能有一到<!--
-->多个[编译项](#配置编译项)。例如，一个编译项用于编译生产代码，另一个用于编译测试代码。

* Kotlin 源代码会放到[源集](#配置源集)中。除了 Kotlin 源文件与<!--
-->资源外，每个源集都可能有自己的依赖项。源集之间以<!--
-->*“依赖”*关系构成了层次结构。源集本身是平台无关的，但是<!--
-->如果一个源集只面向单一平台编译，那么它可能包含平台相关代码与依赖项。

每个编译项都有一个默认源集，是放置该编译项的源代码与依赖项的地<!--
-->方。默认源集还用于通过<!--
-->“依赖”关系将其他源集引到该编译项中。

以下是一个面向 JVM 与 JS 的项目的图示：

![项目结构]({{ url_for('asset', path='images/reference/building-mpp-with-gradle/mpp-structure-default-jvm-js.png') }})

这里有两个目标，即 `jvm` 与 `js`，每个目标都分别编译生产代码、测试代码，其中一些代码是共享的。
这种布局只是通过创建两个目标来实现的，并没有对编译项与源集进行额外配置<!--
-->：都是为相应目标[默认创建](#默认项目布局)的。

在上述示例中，JVM 目标的生产源代码由其 `main` 编译项编译，其中<!--
-->包括来自 `jvmMain` 与 `commonMain`（由于*依赖*关系）的源代码与依赖项：

![源集与编译项]({{ url_for('asset', path='images/reference/building-mpp-with-gradle/mpp-one-compilation.png') }})

这里 `jvmMain` 源集为共享的 `commonMain` 源集中的预期 API 提供了[平台相关实现](platform-specific-declarations.html)<!--
-->。这就是在平台之间灵活共享代码、
按需使用平台相关实现的方式。

在后续部分，会详细描述这些概念以及将其配置到项目中的 DSL。

## 搭建一个多平台项目

可以在 IDE 的 New Project - Kotlin 对话框下选择一个多平台项目模板<!--
-->来创建一个多平台项目。

例如，如果选择了“Kotlin (Multiplatform Library)”，会创建一个包含三个<!--
-->[目标](#设置目标)的库项目，其中一个用于 JVM，一个用于 JS，还有一个用于您正在使用的原生平台。
这些是在 `build.gradle` <!--
-->脚本中以下列方式配置的：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
plugins {
    id 'org.jetbrains.kotlin.multiplatform' version '{{ site.data.releases.latest.version }}'
}

repositories {
    mavenCentral()
}

kotlin {
    jvm() // 使用默认名称 “jvm” 创建一个 JVM 目标
    js()  // 使用名称 “js” 创建一个 JS 目标
    mingwX64("mingw") // 使用名称 “mingw” 创建一个 Windows (MinGW X64) 目标
    
    sourceSets { /* …… */ }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
plugins {
    kotlin("multiplatform") version "{{ site.data.releases.latest.version }}"
}

repositories {
    mavenCentral()
}

kotlin {
    jvm() // 使用默认名称 “jvm” 创建一个 JVM 目标
    js()  // 使用名称 “js” 创建一个 JS 目标
    mingwX64("mingw") // 使用名称 “mingw” 创建一个 Windows (MinGW X64) 目标

    sourceSets { /* …… */ }
}
```

</div>
</div>

这三个目标是通过预设函数 `jvm()`、`js()` 与 `mingwX64()` 创建的，它们提供了一些<!--
-->[默认项目布局](#默认项目布局)。每个[已支持平台](#已支持平台)都有预设的函数。

然后配置[源集](#配置源集)及其[依赖](#添加依赖)，如下所示：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
plugins { /* …… */ }

kotlin {
    /* 省略目标声明 */

    sourceSets {
        commonMain {
            dependencies {
                implementation kotlin('stdlib-common')
            }
        }
        commonTest {
            dependencies {
                implementation kotlin('test-common')
                implementation kotlin('test-annotations-common')
            }
        }

        // 仅用于 JVM 的源码及其依赖的默认源集
        // 或者使用 jvmMain { …… }
        jvm().compilations.main.defaultSourceSet {
            dependencies {
                implementation kotlin('stdlib-jdk8')
            }
        }
        // 仅用于 JVM 的测试及其依赖
        jvm().compilations.test.defaultSourceSet {
            dependencies {
                implementation kotlin('test-junit')
            }
        }

        js().compilations.main.defaultSourceSet  { /* …… */ }
        js().compilations.test.defaultSourceSet { /* …… */ }

        mingwX64('mingw').compilations.main.defaultSourceSet { /* …… */ }
        mingwX64('mingw').compilations.test.defaultSourceSet { /* …… */ }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
plugins { /* …… */ }

kotlin {
    /* 省略目标声明 */

    sourceSets {
        val commonMain by getting {
            dependencies {
                implementation(kotlin("stdlib-common"))
            }
        }
        val commonTest by getting {
            dependencies {
                implementation(kotlin("test-common"))
                implementation(kotlin("test-annotations-common"))
            }
        }

        // 仅用于 JVM 的源码及其依赖的默认源集
        jvm().compilations["main"].defaultSourceSet {
            dependencies {
                implementation(kotlin("stdlib-jdk8"))
            }
        }
        // 仅用于 JVM 的测试及其依赖
        jvm().compilations["test"].defaultSourceSet {
            dependencies {
                implementation(kotlin("test-junit"))
            }
        }

        js().compilations["main"].defaultSourceSet  { /* …… */ }
        js().compilations["test"].defaultSourceSet { /* …… */ }

        mingwX64("mingw").compilations["main"].defaultSourceSet { /* …… */ }
        mingwX64("mingw").compilations["test"].defaultSourceSet { /* …… */ }
    }
}
```

</div>
</div>

这些在上面配置的目标的生产与测试的源码<!--
-->都有各自的[默认源集名称](#默认项目布局)。源集 `commonMain` 与 `commonTest` 将被分别包含在所有目标的生产与测试编译项中。
需要注意的是，公共源集 `commonMain` 与 `commonTest` 的依赖使用的都是公共构件，而<!--
-->平台库将转到特定目标的源集。

## Gradle 插件

Kotlin 多平台项目需要 Gradle 4.7 及以上版本，不支持旧版本的 Gradle。

如需在 Gradle 项目中从头开始设置多平台项目，首先要将
`kotlin-multiplatform` 插件应用到项目中，即在
`build.gradle` 文件的开头添加以下内容：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
plugins {
    id 'org.jetbrains.kotlin.multiplatform' version '{{ site.data.releases.latest.version }}'
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
plugins {
    kotlin("multiplatform") version "{{ site.data.releases.latest.version }}"
}
```

</div>
</div>

这会在顶层创建 `kotlin` 扩展。然后可以在构建脚本中访问该扩展，来：

* 为多个平台[设置目标](#设置目标)（默认不会创建目标）；
* [配置源集](#配置源集)及其[依赖](#添加依赖)；

## 设置目标

目标是构建的一部分，负责编译，测试与打包针对<!--
-->一个[已支持平台](#已支持平台)的软件。

所有的目标可能共享一些源代码，也可能拥有平台专用的源代码。

由于平台的不同，目标也以不同的方式构建，并且拥有各个平台专用的<!--
-->设置。Gradle 插件捆绑了一些已支持平台的预设。

要创建一个目标，请使用其中一个预设函数，这些预置函数根据目标平台命名，并可选择<!--
-->接收一个目标名称与一个配置代码块：

<div class="sample" markdown="1" theme="idea" mode='groovy'>

``` groovy
kotlin {
    jvm() // 用默认名称 “jvm” 创建一个 JVM 目标
    js("nodeJs") // 用自定义名称 “nodeJs” 创建一个 JS 目标
        
    linuxX64("linux") {
        /* 在此处指定 “linux” 的其他设置 */
    }
}
``` 
</div>

如果存在，这些预置函数将返回一个现有的目标。这可以用于配置一个现有的目标：

<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    /* …… */

    // 配置 “jvm6” 目标的属性
    jvm("jvm6").attributes { /* …… */ }
}
```

</div>

注意目标平台与命名都很重要：如果一个目标作为 `jvm('jvm6')` 创建，使用 `jvm()` 将会<!--
-->创建一个单独的目标（使用默认名称 `jvm`）。如果用于创建该名称下的预设函数<!--
-->不同，将会报告一个错误。

从预置函数创建的目标将被添加到域对象集合 `kotlin.targets` 中，这可以用于<!--
-->通过名称访问它们或者配置所有目标：

<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    jvm()
    js("nodeJs")

    println(targets.names) // 打印：[jvm, metadata, nodeJs]

    // 配置所有的目标，包括稍后添加的目标
    targets.all {
        compilations["main"].defaultSourceSet { /* …… */ }
    }
}
```

</div>

要从动态创建或访问多个预设中的多个目标，你可以使用 `targetFromPreset` 函数，
它接收一个接收预设（那些被包含在 `kotlin.presets` 域对象集合中的），以及可选的目标名称与配置的代码块。

例如，要为每一个 Kotlin/Native 支持的平台（见下文）创建目标，使用以下代码：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    presets.withType(org.jetbrains.kotlin.gradle.plugin.mpp.KotlinNativeTargetPreset).each {
        targetFromPreset(it) {
            /* 配置每个已创建的目标 */
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
import org.jetbrains.kotlin.gradle.plugin.mpp.KotlinNativeTargetPreset

/* …… */

kotlin {
    presets.withType<KotlinNativeTargetPreset>().forEach {
        targetFromPreset(it) {
            /* 配置每个已创建的目标 */
        }
    }
}
```

</div>
</div>


### 已支持平台

如上所示，对于以下目标平台，可以使用预设函数应用目标平台预设<!--
-->：

* `jvm` 用于 Kotlin/JVM；
* `js` 用于 Kotlin/JS；
* `android` 用于 Android 应用程序与库。请注意在创建目标之前， 
   应该应用其中之一的 Android Gradle 插件；
  
*  Kotlin/Native 目标平台预设（参见下文[备注](#使用-kotlinnative-目标平台)）：
  
    * `androidNativeArm32` 与 `androidNativeArm64` 用于 Android NDK；
    * `iosArm32`、 `iosArm64`、 `iosX64` 用于 iOS；
    * `watchosArm32`、 `watchosArm64`、 `watchosX86` 用于 watchOS；
    * `tvosArm64`、 `tvosX64` 用于 tvOS；
    * `linuxArm32Hfp`、 `linuxMips32`、 `linuxMipsel32`、 `linuxX64` 用于 Linux；
    * `macosX64` 用于 MacOS；
    * `mingwX64` 与 `mingwX86` 用于 Windows；
    * `wasm32` 用于 WebAssembly。
    
    请注意，某些 Kotlin/Native 目标平台需要[适宜的主机](#使用-kotlinnative-目标平台)来构建。
    
某些目标平台可能需要附加配置。Android 与 iOS 示例请参见<!--
-->[多平台项目：iOS 与 Android](/docs/tutorials/native/mpp-ios-android.html) 教程。

### 配置编译项

构建目标需要一次或多次编译 Kotlin。目标的每次 Kotlin 编译项都可以用于<!--
-->不同的目的（例如生产代码，测试），并包含不同的[源集](#配置源集)。
可以在 DSL 中访问目标的编译项，例如，配置<!--
-->[Kotlin 编译器选项](using-gradle.html#编译器选项)或者获取依赖项文件和编译项输出用来获取任务。

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    jvm {
        compilations.main.kotlinOptions {
            // 为“main”编译项设置 Kotlin 编译器选项：
            jvmTarget = "1.8"
        }

        compilations.main.compileKotlinTask // 获取 Kotlin 任务“compileKotlinJvm”
        compilations.main.output // 获取 main 编译项输出
        compilations.test.runtimeDependencyFiles // 获取测试运行时路径
    }

    // 配置所有目标的所有编译项：
    targets.all {
        compilations.all {
            kotlinOptions {
                allWarningsAsErrors = true
            }
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    jvm {
        val main by compilations.getting {
            kotlinOptions {
                // 为“main”编译项设置 Kotlin 编译器选项：
                jvmTarget = "1.8"
            }

            compileKotlinTask // 获取 Kotlin 任务“compileKotlinJvm”
            output // 获取 main 编译项输出
        }

        compilations["test"].runtimeDependencyFiles // 获取测试运行时路径
    }

    // 配置所有目标的所有编译项：
    targets.all {
        compilations.all {
            kotlinOptions {
                allWarningsAsErrors = true
            }
        }
    }
}
```

</div>
</div>

每个编译项都附带一个[默认源集](#配置源集)，该默认源集<!--
-->存储特定于该编译项的源和依赖项。目标 `bar` 的编译项 `foo` 的默认源集的<!--
-->名称为 `barFoo`。也可以使用 `defaultSourceSet` 
从编译项中访问它：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    jvm() // 使用默认名称“jvm”创建一个 JVM 目标

    sourceSets {
        // “jvm”目标的“main”编译项的默认源集：
        jvmMain {
            /* …… */
        }
    }

    // 或者，从目标的编译项中访问它：
    jvm().compilations.main.defaultSourceSet {
        /* …… */
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    jvm() // 使用默认名称“jvm”创建一个 JVM 目标

    sourceSets {
        // “jvm”目标的“main”编译项的默认源集：
        val jvmMain by getting {
            /* …… */
        }
    }

    // 或者，从目标的编译项中访问它：
    jvm().compilations["main"].defaultSourceSet {
        /* …… */
    }
}
```

</div>
</div>

为了收集参与编译项的所有源集，包括通过依赖关系添加的源集，可以<!--
-->使用属性 `allKotlinSourceSets`。

对于某些特定用例，可能需要创建自定义编译项。这可以在目标的 `compilations` 领域对象集合<!--
-->中完成。请注意，需要为所有自定义编译项手动设置依赖项，并且<!--
-->自定义编译项输出的使用取决于构建所有者。例如，对目标 `jvm()` 的集成测试的<!--
-->自定义编译项：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    jvm() {
        compilations.create('integrationTest') {
            defaultSourceSet {
                dependencies {
                    def main = compilations.main
                    // 根据 main 编译项的编译类路径和输出进行编译：
                    implementation(main.compileDependencyFiles + main.output.classesDirs)
                    implementation kotlin('test-junit')
                    /* …… */
                }
            }

            // 创建一个测试任务来运行此编译项产生的测试：
            tasks.create('jvmIntegrationTest', Test) {
                // 使用包含编译依赖项（包括“main”）的类路径运行测试，
                // 运行时依赖项以及此编译项的输出：
                classpath = compileDependencyFiles + runtimeDependencyFiles + output.allOutputs

                // 仅运行此编译项输出中的测试：        
                testClassesDirs = output.classesDirs
            }
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    jvm() {
        compilations {
            val main by getting

            val integrationTest by compilations.creating {
                defaultSourceSet {
                    dependencies {
                        // 根据 main 编译项的编译类路径和输出进行编译：
                        implementation(main.compileDependencyFiles + main.output.classesDirs)
                        implementation(kotlin("test-junit"))
                        /* …… */
                    }
                }

                // 创建一个测试任务来运行此编译项产生的测试：
                tasks.create<Test>("integrationTest") {
                    // 使用包含编译依赖项（包括“main”）的类路径运行测试，
                    // 运行时依赖项以及此编译项的输出：
                    classpath = compileDependencyFiles + runtimeDependencyFiles + output.allOutputs

                    // 仅运行此编译项输出中的测试：
                    testClassesDirs = output.classesDirs
                }
            }
        }
    }
}
```

</div>
</div>

还要注意，默认情况下，自定义编译项的默认源集既不依赖于 `commonMain` 也不依赖于
`commonTest`。

## 配置源集

Kotlin 源集是 Kotlin 源代码及其资源、依赖关系以及语言设置的集合，
一个源集可能会参与一个或多个[目标](#设置目标)的 Kotlin 编译项。

源集不限于平台特定的或“共享的”；允许包含的内容取决于其用法：
添加到多个编译项中的源集仅限于通用语言特性及依赖项，仅由单个目标使用的源集<!--
-->可以具有平台特定的依赖项，并且其代码可能使用目标平台<!--
-->特定的语言特性。

默认情况下会创建并配置一些源集：`commonMain`、`commonTest` 和编译项的<!--
-->默认源集。 请参见[默认项目布局](#默认项目布局)。

源集在 `kotlin { ... }` 扩展的 `sourceSets { ... }` 块内配置：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    sourceSets { 
        foo { /* …… */ } // 创建或配置名称为 “foo” 的源集
        bar { /* …… */ }
    }
}
``` 

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    sourceSets {
        val foo by creating { /* …… */ } // 创建一个名为 “foo” 的新源集
        val bar by getting { /* …… */ } // 使用名称 “bar” 配置现有的源集
    }
}
```

</div>
</div>

> 注意：创建源集不会将其链接到任何目标。一些源集是[预定义的](#默认项目布局)
因此默认情况下进行编译。但是，始终需要将自定义源集明确地定向到编译项。
请参见：[关联源集](#关联源集)。
{:.note}

源集名称区分大小写。在通过名称引用默认源集时，请确保源集的名称前缀<!--
-->与目标名称匹配，例如，目标 `iosX64` 的源集 `iosX64Main`。

源集本身是平台无关的，但是<!--
-->如果仅针对单个平台进行编译，则可以将其视为特定于平台的。因此，源集可以包含<!--
-->平台之间共享的公共代码或平台特定的代码。

每个源集都有 Kotlin 源代码的默认源目录：`src/<源集名称>/kotlin`。要将 Kotlin 源目录<!--
-->以及资源添加到源集中，请使用其 `kotlin` 与 `resources` `SourceDirectorySet`：

默认情况下，源集的文件存储在以下目录中：

* 源文件：`src/<source set name>/kotlin`
* 资源文件：`src/<source set name>/resources`

应该手动创建这些目录。

要将自定义 Kotlin 源目录和资源添加到源集中，请使用其 `kotlin` 与 `resources` `SourceDirectorySet`：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin { 
    sourceSets { 
        commonMain {
            kotlin.srcDir('src')
            resources.srcDir('res')
        } 
    }
}
``` 

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    sourceSets {
        val commonMain by getting {
            kotlin.srcDir("src")
            resources.srcDir("res")
        }
    }
}
```

</div>
</div>

### 关联源集

Kotlin 源集可能与 *“depends on”* 关系有关，因而如果一个源集 `foo` 依赖于一个<!--
-->源集 `bar`，那么：

* 每当为特定目标编译 `foo` 时，`bar` 也参与到编译中，并且还会编译成<!--
-->相同的目标二进制格式，例如 JVM 类文件或者 JS 代码；

* `foo` 源中的代码能 “看到” `bar` 的定义，包括 `internal` 的以及 `bar` 的[依赖](#添加依赖)，即使是<!--
-->被指定为 `implementation` 的依赖；

* `foo` 可能包含针对 `bar` 的预期定义的[特定平台的实现](platform-specific-declarations.html)；

* `bar` 的资源总是与 `foo` 的资源一起处理与复制；

* `foo` 与 `bar` 的语言应该是一致的；

不允许源集间循环依赖。

源集 DSL 可以用于定义两个源集之间的联系：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    sourceSets {
        commonMain { /* …… */ }
        allJvm {
            dependsOn commonMain
            /* …… */
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    sourceSets {
        val commonMain by getting { /* …… */ }
        val allJvm by creating {
            dependsOn(commonMain)
            /* …… */
        }
    }
}
```

</div>
</div>

除了[默认源集](#默认项目布局)外，还应将创建的自定义源集显式地包含在<!--
-->依赖关系层次结构中，以便于能够使用其他源集的定义，并且最重要的是能够参与到<!--
-->编译中。
大多数时候，它们需要 `dependsOn(commonMain)` 或 `dependsOn(commonTest)` 声明，并且一些默认的特定平台的<!--
-->源集应该直接或间接地<!--
-->依赖于自定义的源集

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin { 
    mingwX64()
    linuxX64()

    sourceSets {
        // 带有两个目标测试的自定义源集
        desktopTest {
            dependsOn commonTest
            /* …… */
        }
        // 将 “windows” 的默认测试源集设置为依赖于 “desktopTest”
        mingwX64().compilations.test.defaultSourceSet {
            dependsOn desktopTest
            /* …… */
        }
        // 并且为其他目标做同样的工作：
        linuxX64().compilations.test.defaultSourceSet {
            dependsOn desktopTest
            /* …… */
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    mingwX64()
    linuxX64()

    sourceSets {
        // 带有两个目标测试的自定义源集
        val desktopTest by creating {
            dependsOn(getByName("commonTest"))
            /* …… */
        }
        // 将 “windows” 的默认测试源集设置为依赖于 “desktopTest”
        mingwX64().compilations["test"].defaultSourceSet {
            dependsOn(desktopTest)
            /* …… */
        }
        // 并且为其他目标做同样的工作：
        linuxX64().compilations["test"].defaultSourceSet {
            dependsOn(desktopTest)
            /* …… */
        }
    }
}
```

</div>
</div>

### 添加依赖

为了添加依赖到源集中，需要在源集 DSL 中使用 `dependencies { …… }` 块，支持以下<!--
-->四种依赖：

* `api` 依赖在编译项与运行时均会使用，并导出到库使用者。如果<!--
-->当前模块的公共 API 中使用了依赖中的任何类型，那么它应该是一个 `api` 依赖；
  
* `implementation` 依赖在当前模块的编译项与运行时均会使用，但不暴露<!--
-->给其他具有 `implementation` 依赖的模块的编译项。对于那种内部逻辑实现所需要的依赖，应该使用 `implementation`
依赖类型。如果模块是一个未发布的 endpoint 应用，它或许该<!--
-->使用 `implementation` 依赖而不是 `api` 依赖。

* `compileOnly` 依赖仅用于当前模块的编译项，并且在运行时与<!---
->其他模块的编译项均不可用。这些依赖应该用于运行时具有第三方实现 API 中。
  
* `runtimeOnly` 依赖在运行时可用，但在任何模块的编译项都是不可见的。

每个源集都可以通过以下方式指定依赖：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    sourceSets {
        commonMain {
            dependencies {
                api 'com.example:foo-metadata:1.0'
            }
        }
        jvm6Main {
            dependencies {
                api 'com.example:foo-jvm6:1.0'
            }
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```groovy
kotlin {
    sourceSets {
        val commonMain by getting {
            dependencies {
                api("com.example:foo-metadata:1.0")
            }
        }
        val jvm6Main by getting {
            dependencies {
                api("com.example:foo-jvm6:1.0")
            }
        }
    }
}
```

</div>
</div>

请注意，为了 IDE 能够正确地识别公共源的依赖，除了特定平台源集构件的依赖外，
公共源集还需要在
Kotlin 元数据包中具有相应的依赖。通常，
在使用一个已发布的库时（除非它与 Gradle 元数据一起发布，如下所述），
需要有一个后缀为 `-common` （如 `kotlin-stdlib-common`）或 `-metadata` 的构件。

然而，在另一个多平台项目中的 `project('……')` 依赖会被自动处理成一个<!--
-->合适的目标。在源集的依赖中指定单个 `project('……')` 依赖就足够了，
并且包含在源集中的编译将会接收到其项目的合适的特定平台的构件，
鉴于它具有兼容的目标：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    sourceSets {
        commonMain {
            dependencies {
                // 包含源集 “commonMain” 的所有编译项
                // 会将依赖项解析为兼容的目标（如果有）：
                api project(':foo-lib')
            }
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    sourceSets {
        val commonMain by getting {
            dependencies {
                // 包含源集 “commonMain” 的所有编译项
                // 会将依赖项解析为兼容的目标（如果有）：
                api(project(":foo-lib"))
            }
        }
    }
}
```

</div>
</div>

同样的，如果以实验性的[Gradle 元数据发布模式](#实验性的元数据发布模式)发布了一个多平台库，并且该项目<!--
-->也设置为使用元数据，那么只需要为公共源集指定一次依赖。
除此以外，应该为每个特定平台的源集<!--
-->提供库的相应平台模块（除了公共模块），如上所示。

指定依赖的另一种方式是在顶层使用 Gradle 内置 DSL，其配置名称遵循<!--
-->模式 `<源集名称><依赖类型>`：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
dependencies {
    commonMainApi 'com.example:foo-common:1.0'
    jvm6MainApi 'com.example:foo-jvm6:1.0'
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
dependencies {
    "commonMainApi"("com.example:foo-common:1.0")
    "jvm6MainApi"("com.example:foo-jvm6:1.0")
}
```

</div>
</div>

一些 Gradle 内置依赖（例如 `gradleApi()`、`localGroovy()`、或 `gradleTestKit()`）<!--
-->在源集依赖 DSL 中是不可用的。但是，你可以将它们添加到顶级依赖块中，如上所示。

可以使用 `kotlin("stdlib")` 表示法添加对 Kotlin 模块（例如 `kotlin-stdlib` 或 `kotlin-reflect`）的依赖，
这是 `"org.jetbrains.kotlin:kotlin-stdlib"` 的简写。

### 语言设置

源集的语言设置可以通过以下方式指定：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    sourceSets {
        commonMain {
            languageSettings {
                languageVersion = '1.3' // 可填：“1.0”、“1.1”、“1.2”、“1.3”
                apiVersion = '1.3' // 可填：“1.0”、“1.1”、“1.2”、“1.3”
                enableLanguageFeature('InlineClasses') // 语言特性名称
                useExperimentalAnnotation('kotlin.ExperimentalUnsignedTypes') // 注解的全限定名
                progressiveMode = true // 默认为 false
            }
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    sourceSets {
        val commonMain by getting {
            languageSettings.apply {
                languageVersion = "1.3" // 可填：“1.0”、“1.1”、“1.2”、“1.3”
                apiVersion = "1.3" // 可填：“1.0”、“1.1”、“1.2”、“1.3”
                enableLanguageFeature("InlineClasses") // 语言特性名称
                useExperimentalAnnotation("kotlin.ExperimentalUnsignedTypes") // 注解的全限定名
                progressiveMode = true // 默认为 false
            }
        }
    }
}
```

</div>
</div>


可以一次性为所有源集配置语言设置：

<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin.sourceSets.all {
    languageSettings.progressiveMode = true
}
```

</div>


源集的语言设置会影响 IDE 识别源代码的方式。由于当前的限制，在
Gradle 构建中，只有构建的默认源集的语言设置会被使用，并且应用于<!--
-->参与编译的所有源代码。

检查语言设置是否相互依赖，以确保源集之间的一致性。即如果 `foo` 依赖于 `bar`：

* `foo` 需设置高于或等于 `bar` 的 `languageVersion`；
* `foo` 需要启用所有 `bar` 启用的非稳定语言特性（对于错误修复特性则没有这种要求）；
* `foo` 需要使用所有 `bar` 使用的实验性注解；
* `apiVersion`、错误修复的语言特性 和 `progressiveMode` 可以被任意设置；

## 默认项目布局

默认情况下，每个项目都包含了两个源集，`commonMain` 与 `commonTest`，在其中可以放置应在<!--
-->所有目标平台之间共享的所有代码。这些源集会被分别添加到每个生产和测试编译项。

之后，当目标被添加时，将为其创建默认编译项：

* 针对 JVM、JS 和原生目标的 `main` 与 `test` 编译项；
* 针对每个 [Android 构建版本](https://developer.android.com/studio/build/build-variants)的编译项；

对于每个编译项，在由 `<目标名称><编译项名称>` 组成的名称下都有一个默认源集。这个默认源集<!--
-->参与编译，因此它应用于特定平台的代码与依赖，并且以依赖的方式将其他<!--
-->源集添加到编译项中。例如，一个有着
`jvm6` （JVM）与 `nodeJs`（JS）目标的项目将拥有源集：`commonMain`、`commonTest`、`jvm6Main`、`jvm6Test`、`nodeJsMain` 以及 `nodeJsTest`。

仅仅是默认源集就涵盖了很多用例，因此不需要自定义源集。
 
每个源集都默认拥有在 `src/<源集名称>/kotlin` 目录下的 Kotlin 源代码与在 `src/<源集名称>/resources` 目录下的资源。

在 Android 项目中，将为每个 [Android 源集](https://developer.android.com/studio/build/#sourcesets)创建额外的 Kotlin 源集.
如果其 Android 目标的名称为 `foo`，那么其 Android 源集 `bar` 将获得一个对应的 Kotlin 源集 `fooBar`。
然而，Kotlin 编译项能够使用来自所有 `src/bar/java`、`src/bar/kotlin` 以及 `src/fooBar/kotlin`
目录的 Kotlin 源代码。而 Java 源代码则只能从上述第一个目录读取。

## 运行测试

目前默认支持 JVM、Android、Linux、Windows 以及 macOS 在 Gradle 构建中运行测试；
JS 与其他 Kotlin/Native 目标<!--
-->需要手动配置以在适当的环境、模拟器或测试框架下运行测试。

将为每个适合测试的目标创建名为 `<目标名称>Test` 的测试任务。运行 `check` 任务以<!--
-->为所有目标运行测试。

由于 `commonTest` [默认源集](#默认项目布局)被添加到所有测试编译项中，所以会将所有目标平台<!--
-->上所需的测试和测试工具放在此处。

[`kotlin.test` API](https://kotlinlang.org/api/latest/kotlin.test/index.html)对于多平台测试是可用的。
添加 `kotlin-test-common` 与 `kotlin-test-annotations-common` 依赖到 `commonTest` 以在<!--
-->公共测试中使用断言函数（例如 `kotlin.test.assertTrue(……)`
以及 `@Test`/`@Ignore`/`@BeforeTest`/`@AfterTest` 注解）

对于 JVM 目标，将 `kotlin-test-junit` 或 `kotlin-test-testng` 用于相应的断言器实现和<!--
-->注解映射。

对于 Kotlin/JS 目标，把 `kotlin-test-js` 添加为测试依赖。至此，将创建针对 Kotlin/JS 的测试任务，但默认情况下并不会运行测试；
应该手动配置它们以使用 JavaScript 测试框架运行测试。

Kotlin/Native 目标不需要额外测试依赖，并且内置了 `kotlin.test` API 的实现。

## 发布多平台库

> 目标平台集合由多平台库作者定义，并且他们应该为库提供所有特定平台的实现。
> 不支持为多平台库添加用户端的新目标。
{:.note} 

来自多平台项目的库构建可以通过 [`maven-publish` Gradle 插件](https://docs.gradle.org/current/userguide/publishing_maven.html)<!--
-->发布到 Maven 仓库，这个插件可通过<!--
-->以下方式应用：

<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
plugins {
    /* …… */
    id("maven-publish")
}
```

</div>

一个库也需要在项目中设置 `group` 与 `version` 字段：

<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
plugins { /* …… */ }

group = "com.example.my.library"
version = "0.0.1"
 ```

</div>

与发布一个普通的 Kotlin/JVM 或 Java 项目相比，并没有必要通过 `publishing { …… }` DSL
来手动创建一个发布项。将为可在当前主机<!--
-->构建的每个目标自动创建发布项，但 Android 目标除外，Android 目标需要额外的步骤来配置<!--
-->发布，参见[发布 Android 库](#发布-android-库)。

通过在 `publishing { …… }` DSL 中的 `repositories` 块添加将被发布的库的仓库。
如 [Maven Publish Plugin. Repositories](https://docs.gradle.org/current/userguide/publishing_maven.html#publishing_maven:repositories) 所述。

默认构件 ID 遵循模式 `<项目名称>-<小写的目标名称>`，例如对于项目 `sample-lib` 中<!--
-->名为 `nodeJs` 的目标，为 `sample-lib-nodejs`。

默认情况下，将为每个发布项中添加源代码 JAR（除了它的主构件）。源代码 JAR 包含了<!--
-->目标主编译项所使用的源代码。如果你还需要发布文档构件（例如
Javadoc JAR），则需要手动配置其构建并且将其作为构件添加到相关发布项中，如<!--
-->下所示。

此外，会默认添加名为 `metadata` 的额外发布项，它包含序列化的 Kotlin
定义并且被 IDE 用于分析多平台库。
这个发布项的默认构件 ID 的形式为 `<项目名称>-metadata`。

可以更改 Maven 坐标，并且可以为在
`targets { …… }` 块或 `publishing { …… }` DSL 中的发布项添加额外的构件文件：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    jvm('jvm6') {
        mavenPublication { // 为目标 “jvm6” 设置发布项
            // 默认的 artifactId 为 “foo-jvm6”，修改它：
            artifactId = 'foo-jvm'
            // 添加 docs JAR 构件（应是一个自定义任务）：
            artifact(jvmDocsJar)
        }
    }
}

// 使用 `publishing { …… }` DSL 来配置发布项是可选的：
publishing {
    publications {
        jvm6 { /* 为目标 “jvm6” 设置发布项 */ }
        metadata { /* 为 Kotlin 元数据设置发布项 */ }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    jvm("jvm6") {
        mavenPublication { // 为目标 “jvm6” 设置发布项
            // 默认的 artifactId 为 “foo-jvm6”，修改它：
            artifactId = "foo-jvm"
            // 添加 docs JAR 构件（应是一个自定义任务）：
            artifact(jvmDocsJar)
        }
    }
}

// 使用 `publishing { …… }` DSL 来配置发布项是可选的：
publishing {
    publications.withType<MavenPublication>().apply {
        val jvm6 by getting { /* 为目标 “jvm6” 设置发布项 */ }
        val metadata by getting { /* 为 Kotlin 元数据设置发布项 */ }
    }
}
```

</div>
</div>

由于 Kotlin/Native 的汇编构件需要多次构建才能在不同的主机平台运行，所以发布<!--
-->包含 Kotlin/Native 目标的多平台库需要使用同一套主机进行。为了避免<!--
-->重复发布能在多个平台
（例如 JVM、JS、Kotlin 元数据以及 WebAssembly）上构建的模块，可以将这些模块的发布任务配置为<!--
-->有条件地运行。

这个简化的例子确保了 JVM、JS 与 Kotlin 元数据的发布仅在命令行中传递
`-PisLinux=true` 到构建时上传：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    jvm()
    js()
    mingwX64()
    linuxX64()

    // 注意 Kotlin 元数据也在这里。
    // 由于 mingwx64() 目标在 Linux 构建中不兼容而被自动跳过。
    configure([targets["metadata"], jvm(), js()]) {
        mavenPublication { targetPublication ->
            tasks.withType(AbstractPublishToMaven)
                .matching { it.publication == targetPublication }
                .all { onlyIf { findProperty("isLinux") == "true" } }
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    jvm()
    js()
    mingwX64()
    linuxX64()

    // 注意 Kotlin 元数据也在这里。
    // 由于 mingwx64() 目标在 Linux 构建中不兼容而被自动跳过。
    configure(listOf(metadata(), jvm(), js())) {
        mavenPublication {
            val targetPublication = this@mavenPublication
            tasks.withType<AbstractPublishToMaven>()
                .matching { it.publication == targetPublication }
                .all { onlyIf { findProperty("isLinux") == "true" } }
        }
    }
}
```

</div>
</div>

### 实验性的元数据发布模式

Gradle 模块元数据提供了丰富的发布与解析依赖项的特性，这些特性用于 Kotlin
多平台项目来为构建作者简化依赖配置。特别是多平台库的发布项<!--
-->可能包含一个特殊的 “根” 模块，它基于整个库，并且<!--
-->在添加为依赖项时自动解析到适当的特定平台构件中，如下所述。

Gradle 5.3 或更高的版本，依赖项解析期间总是使用模块元数据，但在默认情况下，发布项不会<!--
-->包含任何模块元数据。为了启用发布模块元数据，需要添加
`enableFeaturePreview("GRADLE_METADATA")` 到根项目的 `settings.gradle` 文件。对于更旧的 Gradle 版本，
模块元数据的使用也需要这个。

> 注意通过 Gradle 5.3 或更高版本发布的模块元数据不能被低于
> 5.3 的 Gradle 所读取。
{:.note}

随着启用 Gradle 元数据，一个额外的名为 `kotlinMultiplatform` 的 “根” 发布项将添加到项目的<!--
-->发布项中。这个发布项的默认构件 ID 与没有任何额外后缀的项目名称相匹配。
为了配置这个发布项，可以通过 `maven-publish` 插件的 `publishing { …… }` DSL 访问：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin { /* …… */ }

publishing {
    publications {
        kotlinMultiplatform {
            artifactId = "foo"
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin { /* …… */ }

publishing {
    publications {
        val kotlinMultiplatform by getting {
            artifactId = "foo"
        }
    }
}
```

</div>
</div>

这个发布项没有包含任何构件并且仅将其他发布项引用为它的变体。然而，
如果仓库需要，则可能需要提供源代码和文档构件。在这种情况下，在发布项的 scope 中<!--
-->通过使用 [`artifact(...)`](https://docs.gradle.org/current/javadoc/org/gradle/api/publish/maven/MavenPublication.html#artifact-java.lang.Object-) 
添加那些构件， 如上所示访问。

如果库拥有一个 “根” 发布项，用户可以在公共源集中指定对整个库的单个依赖，
并且将为每个包含这个依赖项的编译项（如果有）选择一个合适的特定平台版本。
考虑一个为 JVM 与 JS 编译并且与
“根” 发布项一起发布的 `sample-lib` 库：
 
<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    jvm('jvm6')
    js('nodeJs')

    sourceSets {
        commonMain {
            dependencies {
                // 这单个依赖将解析到适当的目标模块，
                // 例如，对于 JVM 解析为 `sample-lib-jvm6`，而对于 JS 解析为 `sample-lib-js`：
                api 'com.example:sample-lib:1.0'
            }
        }
    }
}
```

</div>    
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    jvm("jvm6")
    js("nodeJs")

    sourceSets {
        val commonMain by getting {
            dependencies {
                // 这单个依赖将解析到适当的目标模块，
                // 例如，对于 JVM 解析为 `sample-lib-jvm6`，而对于 JS 解析为 `sample-lib-js`：
                api("com.example:sample-lib:1.0")
            }
        }
    }
}
```

</div>
</div>

这需要使用者的 Gradle 构建可以读取 Gradle 模块元数据，要么使用 Gradle 5.3+，要么<!--
-->在 `settings.gradle` 中通过 `enableFeaturePreview("GRADLE_METADATA")` 显式地启用它

### 目标消歧义

在一个多平台库中，对于单个平台可能拥有多个目标。例如，这些目标<!--
-->可能提供了相同的 API，并且在运行时调用的实现库中有所不同，例如测试框架或日志解决方案。

然而，对这种多平台库的依赖可能存在歧义，并且可能因为没有<!--
-->充足的信息来决定选择哪个目标，从而导致解析的失败。

解决的方法是用自定义属性标记目标, Gradle 会根据它来解析依赖项。
但是，库的作者与用户必须同时给目标加上自定义属性，
并且库作者有责任将属性与可能的值传达给使用者。
 
添加属性对库作者和用户来说是对称的。例如，考虑一个<!--
-->在两个目标中分别支持了 JUnit 和 TestNG 的测试库。库作者需要为这两个<!--
-->目标添加属性，如下： 

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
def testFrameworkAttribute = Attribute.of('com.example.testFramework', String)

kotlin {
    jvm('junit') {
        attributes.attribute(testFrameworkAttribute, 'junit')
    }
    jvm('testng') {
        attributes.attribute(testFrameworkAttribute, 'testng')
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
val testFrameworkAttribute = Attribute.of("com.example.testFramework", String::class.java)

kotlin {
    jvm("junit") {
        attributes.attribute(testFrameworkAttribute, "junit")
    }
    jvm("testng") {
        attributes.attribute(testFrameworkAttribute, "testng")
    }
}
```

</div>
</div>

用户可能只需要给产生歧义的单个目标添加属性。

如果将依赖项被添加到自定义的配置项中（而不是<!--
-->通过插件创建的配置项之一）时出现了相同的歧义，你可以通过相同的方式将属性添加到配置项中：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
def testFrameworkAttribute = Attribute.of('com.example.testFramework', String)

configurations {
    myConfiguration {
        attributes.attribute(testFrameworkAttribute, 'junit')
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
val testFrameworkAttribute = Attribute.of("com.example.testFramework", String::class.java)

configurations {
    val myConfiguration by creating {
        attributes.attribute(testFrameworkAttribute, "junit")
    }
}
```

</div>
</div>

## JVM 目标平台中的 Java 支持

这个特性自 Kotlin 1.3.40 可用。

默认情况下，JVM 目标将忽略 Java 源代码，并且只编译 Kotlin 源文件。
为了将 Java 源代码包含入 JVM 目标的编译项中，或是为了应用需要
`java` 插件才能工作的 Gradle 插件，你需要为目标显式地启用 Java 支持：

<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    jvm {
        withJava()
    }
}
```

</div>

这将会应用 Gradle `java` 插件，并配置目标以与它协作。
注意，在 JVM 目标中仅应用 Java 插件但没有指定 `withJava()`，
将不会对目标有任何影响。

Java 源代码的文件系统位置与 `java` 插件的默认值不同。
Java 源文件需要被放置在 Kotlin 源代码<!--
-->根目录的同级目录中。例如，如果 JVM 目标有一个默认名称 `jvm`，则路径为：

<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```
src
├── jvmMain
│   ├── java // production Java sources
│   ├── kotlin
│   └── resources
├── jvmTest
│   ├── java // test Java sources
│   ├── kotlin
…   └── resources
```

</div>

公共源集不能包含 Java 源代码。

由于当前的限制，一些由 Java 插件配置的任务将被禁用，并且 Kotlin 插件添加了<!--
-->相应的任务来代替它们：

* `jar` 被禁用，取而代之的是目标的 JAR 任务（例如 `jvmJar`）
* `test` 被禁用，并且使用目标的测试任务（例如 `jvmTest`）
* `*ProcessResources` 任务被禁用，并且资源将由编译项的等价任务处理

这个目标的发布项将由 Kotlin 插件处理，并且不需要特定于
Java 插件的步骤，例如手动创建发布项并配置它为 `from(components.java)`。

##  Android 支持

Kotlin 多平台项目通过提供 `android` 内置函数支持 Android 平台。
创建 Android 目标需要 Android Gradle 插件之一，例如手动应用`com.android.application` 或
`com.android.library` 到项目中。每个 Gradle 子项目仅可能创建一个 Android 目标：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
plugins {
    id("com.android.library")
    id("org.jetbrains.kotlin.multiplatform").version("{{ site.data.releases.latest.version }}")
}

android { /* …… */ }

kotlin {
    android { // 创建 Android 目标
        // 提供必要的附加配置
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
plugins {
    id("com.android.library")
    kotlin("multiplatform").version("{{ site.data.releases.latest.version }}")
}

android { /* …… */ }

kotlin {
    android { // 创建 Android 目标
        // 提供必要的附加配置
    }
}
```

</div>
</div>

默认创建的 Android 目标编译项与 [Android 构建变体](https://developer.android.com/studio/build/build-variants)相关联：
对于每个构建变体，将会以相同的名称创建 Kotlin 构建项。

然后，对于每个通过变体编译的 [Android 源集](https://developer.android.com/studio/build/build-variants#sourcesets)，
将在目标名称前面的那个源集名称下创建 Kotlin 源集，
例如 Kotlin 源集 `androidDebug` 用于 Android 源集 `debug`
与名为 `android` 的 Kotlin 目标。 这些 Kotlin 源集将相应地添加到变体编译项中。

默认源集 `commonMain` 将添加到每个生产项（应用或库）变体的编译项中。
类似地，`commonTest` 源集也将添加到单元测试的编译项，以及 instrumented 测试变体中。

使用 [kapt](/docs/reference/kapt.html) 进行注解处理也是受支持的，但，由于当前的限制，
它要求 Android 目标需要在配置 `kapt` 依赖之前创建，`kapt` 依赖需要在<!--
-->顶级 `dependencies { …… }` 代码块（而不是 Kotlin 源集依赖）中完成。

<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
// ...

kotlin {
    android { /* …… */ }
}

dependencies {
    kapt("com.my.annotation:processor:1.0.0")
}
```

</div>

### 发布 Android 库

为了将 Android 库发布为多平台库的一部分，需要<!--
-->[为库设定发布项](#发布多平台库)，并且为
Android 库目标提供额外的配置项。

默认情况下，不会发布 Android 库的构件。为了发布
[Android 变体](https://developer.android.com/studio/build/build-variants)生成的一系列构件，需要在 Android
目标代码块中指定变体名称，如下所示：

<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    android {
        publishLibraryVariants("release", "debug")
    }
}
```

</div>

上面的例子将在没有产品类型的 Android 库上工作。对于有产品类型的库，变体<!--
-->名称也要包含产品类型，例如 `fooBarDebug` 或是 `fooBazRelease`。

注意，如果库用户定义了库中缺失的变体，则他们需要提供<!--
-->[备用的匹配](https://developer.android.com/studio/build/dependencies#resolve_matching_errors)。例如，如果<!--
-->库不具有，或者没有发布 `staging` 构建类型，那么有必要为<!--
-->拥有这种构建类型的使用者提供备用的匹配，至少指定库发布项的一个构建类型：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
android {
    buildTypes {
        staging {
            // ...
            matchingFallbacks = ['release', 'debug']
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
android {
    buildTypes {
        val staging by creating {
            // ...
            matchingFallbacks = listOf("release", "debug")
        }
    }
}
```

</div>
</div>

类似地，如果库发布项中缺失某些备用的匹配，那么库用户也许需要为自定义产品类型<!--
-->提供它们。

你可以选择发布按产品类型分组的变体，以便将不同构建类型的输出<!--
-->放置在单独的模块中，并使构建类型成为构建的分类器（release 构建<!--
-->类型不通过分类器发布）。这个模式默认是禁用的，不过可以通过以下方式启用：

<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```groovy
kotlin {
    android {
        publishLibraryVariantsGroupedByFlavor = true
    }
}
```

</div>

不推荐发布按产品类型分组的变体，以免它们拥有不同的依赖项，因为<!--
-->这些将被合并到一个依赖项列表中。

## 使用 Kotlin/Native 目标平台

重要的是，注意某些 [Kotlin/Native 目标](#已支持平台)仅能在适当的主机上被编译：

* Linux MIPS 目标（`linuxMips32` 与 `linuxMipsel32`）需要一台 Linux 主机。其他 Linux 目标则可以在任意受支持的主机上编译。
* Windows 目标需要一台 Windows 主机；
* macOS 与 iOS 目标只能在 macOS 主机上编译；
* 64 位的 Android 原生目标需要一台 Linux 或 macOS 主机。32 位的 Android 原生目标则可以在任意受支持的主机上编译。

当前主机不支持的目标在构建期间会被忽略，因此也不会发布。库作者可能希望在目标库平台所需的不同主机上<!--
-->进行构建和发布。

### 目标快捷方式

一些原生目标经常一同创建，并且使用相同的源代码。例如，iOS 设备与模拟器<!--
-->的构建由不同的目标（分别是 `iosArm64` 与 `iosX64`）表示，但它们的源代码通常是相同的。
多平台项目模型中来表示这种共享代码的一个经典方式是创建一个中间<!--
-->源集（`iosMain`），并且在它和平台源集之间配置链接：

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
sourceSets{
    iosMain {
        dependsOn(commonMain)
        iosDeviceMain.dependsOn(it)
        iosSimulatorMain.dependsOn(it)
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
val commonMain by sourceSets.getting
val iosDeviceMain by sourceSets.getting
val iosSimulatorMain by sourceSets.getting

val iosMain by sourceSets.creating {
    dependsOn(commonMain)
    iosDeviceMain.dependsOn(this)
    iosSimulatorMain.dependsOn(this)
}
```

</div>
</div>

自 1.3.60 起，`kotlin-multiplaform` 插件提供了自动化这些配置的快捷方式：它们使用户<!--
-->可以通过单个 DSL 方法来创建一组目标以及公共源集。

可用快捷方式有这些：

 * `ios` 为 `iosArm64` 与 `iosX64` 创建目标。
 * `watchos` 为 `watchosArm32`、`watchosArm64` 以及 `watchosX86` 创建目标。
 * `tvos` 为 `tvosArm64` 与 `tvosX64` 创建目标。

<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
// 为 iOS 创建两个目标。
// 创建公共源集：iosMain 与 iosTest。
ios {
    // 配置目标。
    // 注意：将会为每个目标调用这个 lambda。
}

// 你也可以指定一个名称前缀来创建目标。
// 公共源集也将会有一个前缀：
// anotherIosMain 与 anotherIosTest。
ios("anotherIos")
```

</div>

### 构建最终原生二进制文件

默认情况下，Kotlin/Native 目标将被编译为 `*.klib` 库构件，它可以被 Kotlin/Native 自身作为<!--
-->依赖项使用，但并不能被执行，或是用作原生库。为了声明像可执行文件或是链接库的最终原生二进制文件，
需要使用原生目标的 `binaries` 属性。除默认 `*.klib` 构建外，
这个属性还代表一个为这个目标构建的原生二进制文件集合，并且提供了一系列声明和配置它们的方法。

注意，`kotlin-multiplaform` 插件默认不会创建任何生产二进制文件。默认情况下，
唯一可用的二进制文件是调试可执行文件，它允许运行来自 `test` 编译项的测试。

#### 声明二进制文件

A set of factory methods is used for declaring elements of the `binaries` collection. These methods allow one to specify what kinds of binaries are to be created and configure them. The following binary kinds are supported (note that not all the kinds are available for
all native platforms):

|**Factory method**|**Binary kind**|**Available for**|
| --- | --- | --- |
|`executable` |a product executable    |all native targets|
|`test`       |a test executable       |all native targets| 
|`sharedLib`  |a shared native library |all native targets except `wasm32`|
|`staticLib`  |a static native library  |all native targets except `wasm32`|
|`framework`  |an Objective-C framework |macOS, iOS, watchOS, and tvOS targets only|

Each factory method exists in several versions. Consider them by example of the `executable` method. All the same versions are available
for all other factory methods.

The simplest version doesn't require any additional parameters and creates one binary for each build type.
Currently there a two build types available: `DEBUG` (produces a not optimized binary with a debug information) and `RELEASE` (produces
an optimized binary without debug information). Consequently the following snippet creates two executable binaries: debug and release.

<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    linuxX64 { // Use your target instead.
        binaries {
            executable {
                // Binary configuration.
            }
        }
    }
}
```

</div>

A lambda expression accepted by the `executable` method in the example above is applied to each binary created and allows one to configure the binary
(see the [corresponding section](#configuring-binaries)). Note that this lambda can be dropped if there is no need for additional configuration:

<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
binaries {
    executable()
}
```

</div>

It is possible to specify which build types will be used to create binaries and which won't. In the following example only debug executable is created.

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
binaries {
    executable([DEBUG]) {
        // Binary configuration.
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
binaries {
    executable(listOf(DEBUG)) {
        // Binary configuration.
    }
}
```

</div>
</div>

Finally the last factory method version allows customizing the binary name.

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
binaries {
    executable('foo', [DEBUG]) {
        // Binary configuration.
    }

    // It's possible to drop the list of build types (all the available build types will be used in this case).
    executable('bar') {
        // Binary configuration.
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
binaries {
    executable("foo", listOf(DEBUG)) {
        // Binary configuration.
    }

    // It's possible to drop the list of build types (all the available build types will be used in this case).
    executable("bar") {
        // Binary configuration.
    }
}
```

</div>
</div>

The first argument in this example allows one to set a name prefix for the created binaries which is used to access them in the buildscript (see the ["Accessing binaries"](#accessing-binaries) section).
Also this prefix is used as a default name for the binary file. For example on Windows the sample above produces files `foo.exe` and `bar.exe`.

#### 访问二进制文件

The binaries DSL allows not only creating binaries but also accessing already created ones to configure them or get their properties
(e.g. path to an output file). The `binaries` collection implements the
[`DomainObjectSet`](https://docs.gradle.org/current/javadoc/org/gradle/api/DomainObjectSet.html) interface and provides methods like
`all` or `matching` allowing configuring groups of elements.

Also it's possible to get a certain element of the collection. There are two ways to do this. First, each binary has a unique
name. This name is based on the name prefix (if it's specified), build type and binary kind according to the following pattern:
`<optional-name-prefix><build-type><binary-kind>`, e.g. `releaseFramework` or `testDebugExecutable`.

> Note: static and shared libraries has suffixes `static` and `shared` respectively, e.g. `fooDebugStatic` or `barReleaseShared`

This name can be used to access the binary:

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
// Fails if there is no such a binary.
binaries['fooDebugExecutable']
binaries.fooDebugExecutable
binaries.getByName('fooDebugExecutable')

 // Returns null if there is no such a binary.
binaries.findByName('fooDebugExecutable')
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
// Fails if there is no such a binary.
binaries["fooDebugExecutable"]
binaries.getByName("fooDebugExecutable")

 // Returns null if there is no such a binary.
binaries.findByName("fooDebugExecutable")
```

</div>
</div>

The second way is using typed getters. These getters allow one to access a binary of a certain type by its name prefix and build type.

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
// Fails if there is no such a binary.
binaries.getExecutable('foo', DEBUG)
binaries.getExecutable(DEBUG)          // Skip the first argument if the name prefix isn't set.
binaries.getExecutable('bar', 'DEBUG') // You also can use a string for build type.

// Similar getters are available for other binary kinds:
// getFramework, getStaticLib and getSharedLib.

// Returns null if there is no such a binary.
binaries.findExecutable('foo', DEBUG)

// Similar getters are available for other binary kinds:
// findFramework, findStaticLib and findSharedLib.
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
// Fails if there is no such a binary.
binaries.getExecutable("foo", DEBUG)
binaries.getExecutable(DEBUG)          // Skip the first argument if the name prefix isn't set.
binaries.getExecutable("bar", "DEBUG") // You also can use a string for build type.

// Similar getters are available for other binary kinds:
// getFramework, getStaticLib and getSharedLib.

// Returns null if there is no such a binary.
binaries.findExecutable("foo", DEBUG)

// Similar getters are available for other binary kinds:
// findFramework, findStaticLib and findSharedLib.
```

</div>
</div>

> Before 1.3.40, both test and product executables were represented by the same binary type. Thus to access the default test binary created by the plugin, the following line was used:
> ```
> binaries.getExecutable("test", "DEBUG")
> ``` 
> Since 1.3.40, test executables are represented by a separate binary type and have their own getter. To access the default test binary, use:
> ```
> binaries.getTest("DEBUG")
> ```
{:.note}
 

#### 配置二进制文件

Binaries have a set of properties allowing one to configure them. The following options are available:

 - **Compilation.** Each binary is built on basis of some compilation available in the same target. The default value of this parameter depends
 on the binary type: `Test` binaries are based on the `test` compilation while other binaries - on the `main` compilation.
 - **Linker options.** Options passed to a system linker during binary building. One can use this setting for linking against some native library.
 - **Output file name.** By default the output file name is based on binary name prefix or, if the name prefix isn't specified, on a project name.
 But it's possible to configure the output file name independently using the `baseName` property. Note that final file name will be formed
 by adding system-dependent prefix and postfix to this base name. E.g. a `libfoo.so` is produced for a Linux shared library with the base name `foo`.
 - **Entry point** (for executable binaries only). By default the entry point for Kotlin/Native programs is a `main` function located in the root
 package. This setting allows one to change this default and use a custom function as an entry point. For example it can be used to move the `main`
 function from the root package.
 - **Access to the output file.**
 - **Access to a link task.**
 - **Access to a run task** (for executable binaries only). The `kotlin-multiplatform` plugin creates run tasks for all executable binaries of host
 platforms (Windows, Linux and macOS). Names of such tasks are based on binary names, e.g. `runReleaseExecutable<target-name>`
 or `runFooDebugExecutable<target-name>`. A run task can be accessed using the `runTask` property of an executable binary.
- **Framework type** (only for Objective-C frameworks). By default a framework built by Kotlin/Native contains a dynamic library. But it's possible
 to replace it with a static library.

The following example shows how to use these settings.

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
binaries {
    executable('my_executable', [RELEASE]) {
        // Build a binary on the basis of the test compilation.
        compilation = compilations.test

        // Custom command line options for the linker.
        linkerOpts = ['-L/lib/search/path', '-L/another/search/path', '-lmylib']

        // Base name for the output file.
        baseName = 'foo'

        // Custom entry point function.
        entryPoint = 'org.example.main'

        // Accessing the output file.
        println("Executable path: ${outputFile.absolutePath}")

        // Accessing the link task.
        linkTask.dependsOn(additionalPreprocessingTask)

        // Accessing the run task.
        // Note that the runTask is null for non-host platforms.
        runTask?.dependsOn(prepareForRun)
    }

    framework('my_framework' [RELEASE]) {
        // Include a static library instead of a dynamic one into the framework.
        isStatic = true
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
binaries {
    executable("my_executable", listOf(RELEASE)) {
        // Build a binary on the basis of the test compilation.
        compilation = compilations["test"]

        // Custom command line options for the linker.
        linkerOpts = mutableListOf("-L/lib/search/path", "-L/another/search/path", "-lmylib")

        // Base name for the output file.
        baseName = "foo"

        // Custom entry point function.
        entryPoint = "org.example.main"

        // Accessing the output file.
        println("Executable path: ${outputFile.absolutePath}")

        // Accessing the link task.
        linkTask.dependsOn(additionalPreprocessingTask)

        // Accessing the run task.
        // Note that the runTask is null for non-host platforms.
        runTask?.dependsOn(prepareForRun)
    }

    framework("my_framework" listOf(RELEASE)) {
        // Include a static library instead of a dynamic one into the framework.
        isStatic = true
    }
}
```

</div>
</div>

#### 导出依赖项为二进制文件

When building an Objective-C framework or a native library (shared or static), it is often necessary to pack not just the
classes of the current project, but also the classes of some of its dependencies. The Binaries DSL allows one to specify
which dependencies will be exported to a binary using the `export` method. Note that only API dependencies of a corresponding source set can be exported.

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    sourceSets {
        macosMain.dependencies {
            // Will be exported.
            api project(':dependency')
            api 'org.example:exported-library:1.0'

            // Will not be exported.
            api 'org.example:not-exported-library:1.0'
        }
    }

    macosX64("macos").binaries {
        framework {
            export project(':dependency')
            export 'org.example:exported-library:1.0'
        }

        sharedLib {
            // It's possible to export different sets of dependencies to different binaries.
            export project(':dependency')
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    sourceSets {
        macosMain.dependencies {
            // Will be exported.
            api(project(":dependency"))
            api("org.example:exported-library:1.0")

            // Will not be exported.
            api("org.example:not-exported-library:1.0")
        }
    }

    macosX64("macos").binaries {
        framework {
            export(project(":dependency"))
            export("org.example:exported-library:1.0")
        }

        sharedLib {
            // It's possible to export different sets of dependencies to different binaries.
            export(project(':dependency'))
        }
    }
}
```

</div>
</div>

> As shown in this example, maven dependency also can be exported. But due to current limitations of Gradle metadata such a dependency
should be either a platform one (e.g. `kotlinx-coroutines-core-native_debug_macos_x64` instead of `kotlinx-coroutines-core-native`)
or be exported transitively (see below).

By default, export works non-transitively. If a library `foo` depending on library `bar` is exported, only methods of `foo` will
be added in the output framework. This behaviour can be changed by the `transitiveExport` flag.

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
binaries {
    framework {
        export project(':dependency')
        // Export transitively.
        transitiveExport = true
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
binaries {
    framework {
        export(project(":dependency"))
        // Export transitively.
        transitiveExport = true
    }
}
```

</div>
</div>

#### 构建通用框架

By default, an Objective-C framework produced by Kotlin/Native supports only one platform. However, such frameworks can be merged
into a single universal (fat) binary using the `lipo` utility. Particularly, such an operation makes sense for 32-bit and 64-bit iOS
frameworks. In this case the resulting universal framework can be used on both 32-bit and 64-bit devices.

The Gradle plugin provides a separate task that creates a universal framework for iOS targets from several regular ones.
The example below shows how to use this task. Note that the fat framework must have the same base name as the initial frameworks.

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
import org.jetbrains.kotlin.gradle.tasks.FatFrameworkTask

kotlin {
    // Create and configure the targets.
    targets {
        iosArm32("ios32")
        iosArm64("ios64")

        configure([ios32, ios64]) {
            binaries.framework {
                baseName = "my_framework"
            }
        }
    }

    // Create a task building a fat framework.
    task debugFatFramework(type: FatFrameworkTask) {
        // The fat framework must have the same base name as the initial frameworks.
        baseName = "my_framework"

        // The default destination directory is '<build directory>/fat-framework'.
        destinationDir = file("$buildDir/fat-framework/debug")

        // Specify the frameworks to be merged.
        from(
            targets.ios32.binaries.getFramework("DEBUG"),
            targets.ios64.binaries.getFramework("DEBUG")
        )
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
import org.jetbrains.kotlin.gradle.tasks.FatFrameworkTask

kotlin {
    // Create and configure the targets.
    val ios32 = iosArm32("ios32")
    val ios64 = iosArm64("ios64")

    configure(listOf(ios32, ios64)) {
        binaries.framework {
            baseName = "my_framework"
        }
    }

    // Create a task building a fat framework.
    tasks.create("debugFatFramework", FatFrameworkTask::class) {
        // The fat framework must have the same base name as the initial frameworks.
        baseName = "my_framework"

        // The default destination directory is '<build directory>/fat-framework'.
        destinationDir = buildDir.resolve("fat-framework/debug")

        // Specify the frameworks to be merged.
        from(
            ios32.binaries.getFramework("DEBUG"),
            ios64.binaries.getFramework("DEBUG")
        )
    }
}
```

</div>
</div>

### C 互操作支持

Since Kotlin/Native provides [interoperability with native languages](native/c_interop.html),
there is a DSL allowing one to configure this feature for a specific compilation.

A compilation can interact with several native libraries. Interoperability with each of them can be configured in
the `cinterops` block of the compilation:

<div class="multi-language-sample" data-lang="groovy">
<div class="sample" markdown="1" theme="idea" mode='groovy'>

```groovy
kotlin {
    linuxX64 { // Replace with a target you need.
        compilations.main {
            cinterops {
                myInterop {
                    // Def-file describing the native API.
                    // The default path is src/nativeInterop/cinterop/<interop-name>.def
                    defFile project.file("def-file.def")

                    // Package to place the Kotlin API generated.
                    packageName 'org.sample'

                    // Options to be passed to compiler by cinterop tool.
                    compilerOpts '-Ipath/to/headers'

                    // Directories for header search (an analogue of the -I<path> compiler option).
                    includeDirs.allHeaders("path1", "path2")

                    // Additional directories to search headers listed in the 'headerFilter' def-file option.
                    // -headerFilterAdditionalSearchPrefix command line option analogue.
                    includeDirs.headerFilterOnly("path1", "path2")

                    // A shortcut for includeDirs.allHeaders.
                    includeDirs("include/directory", "another/directory")
                }

                anotherInterop { /* …… */ }
            }
        }
    }
}
```

</div>
</div>

<div class="multi-language-sample" data-lang="kotlin">
<div class="sample" markdown="1" theme="idea" mode='kotlin' data-highlight-only>

```kotlin
kotlin {
    linuxX64 {  // Replace with a target you need.
        compilations.getByName("main") {
            val myInterop by cinterops.creating {
                // Def-file describing the native API.
                // The default path is src/nativeInterop/cinterop/<interop-name>.def
                defFile(project.file("def-file.def"))

                // Package to place the Kotlin API generated.
                packageName("org.sample")

                // Options to be passed to compiler by cinterop tool.
                compilerOpts("-Ipath/to/headers")

                // Directories to look for headers.
                includeDirs.apply {
                    // Directories for header search (an analogue of the -I<path> compiler option).
                    allHeaders("path1", "path2")

                    // Additional directories to search headers listed in the 'headerFilter' def-file option.
                    // -headerFilterAdditionalSearchPrefix command line option analogue.
                    headerFilterOnly("path1", "path2")
                }
                // A shortcut for includeDirs.allHeaders.
                includeDirs("include/directory", "another/directory")
            }

            val anotherInterop by cinterops.creating { /* …… */ }
        }
    }
}

```

</div>
</div>

Often it's necessary to specify target-specific linker options for a binary which uses a native library. It can by done
using the `linkerOpts` property of the binary. See the [Configuring binaries](#configuring-binaries) section for details.
