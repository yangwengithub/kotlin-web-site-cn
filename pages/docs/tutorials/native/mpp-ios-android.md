---
type: tutorial
layout: tutorial
title:  "多平台项目: iOS 与 Android"
description: "在 iOS 与 Android 之间共享 Kotlin 代码"
authors: Eugene Petrenko，乔禹昂（翻译）
date: 2018-10-04
showAuthorInfo: true
issue: EVAN-6029
---

在本教程中我们将通过创建一个 iOS 与一个 Android 应用，来展示 Kotlin 代码的共享功能。
在 Android 上我们将使用 Kotlin/JVM，而在 iOS 上将是 Kotlin/Native。

我们将学习到如何：
 - 使用 Android Studio 创建一个 [Android app](#创建一个-Android-工程)
 - 创建一个共享的 [Kotlin library](#创建共享模块)
   - 在 [Android app](#在-android-中使用共享代码) 中使用它
   - 运行 [Android 应用程序](#运行-android-应用程序)
 - 使用 Xcode 创建一个 [iOS app](#创建-ios-应用程序)
   - 在 [iOS app](#在-xcode-中配置-framework-依赖) 上使用共享的 Kotlin 库
   - 在 [Swift 中使用 Kotlin](#在-swift-中调用-kotlin-代码)
   - 运行 [iOS 应用程序](#运行-ios-应用程序)

本教程的目标是展示 Kotlin 共享代码的能力以及它带来的优势。
我们将会看到的是一个简化的应用程序，但这里展示的内容可以用于真实的应用，
而与它的大小或复杂度无关。

我们要创建的应用程序将只在 Android 上显示
`Kotlin Rocks on Android` 或在 iOS 上显示 `Kotlin Rocks on iOS <version>`。
我们希望共享生成此消息的代码。

通用的代码是 `"Kotlin Rocks on ${platformName()}"`，`platformName()`
是一个使用 `expect` 关键字声明的函数。而 `actual` 的实现将根据特定的平台而异。

# 搭建本地环境

* 我们将使用 [Android Studio](https://developer.android.com/studio/) 来讲解 Android 部分的内容. 
当然也可以使用 [IntelliJ IDEA](https://jetbrains.com/idea) 社区版或终级版。

* IDE 应该安装了 Kotlin 插件 1.3.21 或更高版本。这个可以通过
IDE 的 *Settings*（或*Preferences*）中的 *Language & Frameworks | Kotlin Updates* 部分验证。

* 编译 iOS 以及 macOS 设备的代码需要 masOS 系统。我们需要安装以及配置
[Xcode](https://developer.apple.com/xcode/) 工具。查看
[Apple 开发者网站](https://developer.apple.com/xcode/)来获取更多细节。

*注意：我们将使用 IntelliJ IDEA 2018.3 EAP、Android Studio 3.2、Kotlin 1.3.21、Xcode 10.0、macOS 10.14、Gradle 4.7*

# 创建一个 Android 工程

我们将通过 *Start New Android Project* 来创建一个 Android 工程。如果使用 IntelliJ IDEA，我们需要在左边的 *New Project* 
向导面板中选择 *Android*。

重要的一点是你需要确保勾选了 *Include Kotlin support* 选择框。现在我们可以在向导的下一步中<!--
-->保留默认设置。我们接下来选择 *Empty Activity* 选项并点击 *Next*，最后点击 *Finish*。

**注意** 如果使用早期发行版或者 EAP 版本的 Kotlin plugin，IDE 在生成工程的时候可能会失败， 
给 Gradle 抛出 [error](https://youtrack.jetbrains.com/issue/KT-18835#focus=streamItem-27-2718879-0-0)。
这是因为 `build.gradle` 文件中没有引用正确的 Maven 库，可以通过将以下代码 *两次* 添加到每个 `repositories { .. }`
块中来解决。

<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
maven { url 'https://dl.bintray.com/kotlin/kotlin-eap' }
```
</div>

<a name="gradle-upgrade"/>
Kotlin/Native 插件需要更新版本的 Gradle，让我们修改 `gradle/wrapper/gradle-wrapper.properties`
并且使用下面的 `distrubutionUrl`：
```
distributionUrl=https\://services.gradle.org/distributions/gradle-4.7-all.zip
```

我们需要通过刷新 Gradle Project 来让其接受设置变更。点击 `Sync Now` 链接或者<!--
-->在 Gradle 根工程的上下文菜单中使用 *Gradle* 工具窗口并且点击刷新按钮。

此刻，我们应该可以编译并运行这个 Android 应用了

# 创建共享模块

这部分教程的目标是演示在 Android 与 iOS 之间复用 Kotlin 源码。让我们从在
Gradle 工程中创建一个 `SharedCode` 子工程开始。`SharedCode` 工程中的源码<!--
-->将被在两个平台之间共享。
我们将在工程中创建几个新文件来实现这个目标。

## 添加 Kotlin 源码

我们想要使每个平台都根据平台自身展示相似的文本：`Kotlin Rocks on Android` 以及
`Kotlin Rocks on iOS`。我们将复用生成消息的方式。
让我们在 `SharedCode/src/commonMain/kotlin/common.kt` 下创建一个 main 文件。

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
package org.kotlin.mpp.mobile

expect fun platformName(): String

fun createApplicationScreenMessage() : String {
  return "Kotlin Rocks on ${platformName()}"
}

```
</div>

这是通用的部分。这段代码生成了最终的消息。它 `expect` 平台<!--
-->提供来自 `expect fun platformName（）：String` 函数的平台名称。我们将同时在
Android 与 iOS 应用中使用 `createApplicationScreenMessage`。

现在，我们需要在 `SharedCode/src/androidMain/kotlin/actual.kt` 中为 Android 创建相应的实现：
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
package org.kotlin.mpp.mobile

actual fun platformName(): String {
  return "Android"
}

```
</div>

我们为 iOS 也创建了一个相似的文件 `SharedCode/src/iosMain/kotlin/actual.kt`：
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
package org.kotlin.mpp.mobile

import platform.UIKit.UIDevice

actual fun platformName(): String {
  return UIDevice.currentDevice.systemName() +
         " " +
         UIDevice.currentDevice.systemVersion
}
```
</div>

这里我们可以使用 Apple UIKit Framework 中的 [UIDevice](https://developer.apple.com/documentation/uikit/uidevice?language=objc)
类，这是一个仅仅在 Swift 以及 Objective-C 中使用而在 Java 中没有的类。
Kotlin/Native 编译器带有一组预先导入的框架，所以我们可以无需额外步骤的<!--
-->使用 UIKit Framework。
Objective-C 与 Swift 互操作的细节被包含在这篇[文档](/docs/reference/native/objc_interop.html)中

## 更新 Gradle 脚本

`SharedCode` 应该为我们生成一系列构件：
 - 在 `androidMain` 源集中为 Android 工程生成 JAR 文件
 - Apple framework 
   - 面向 iOS 设备以及 App Store（`arm64` 目标平台）
   - 面向 iOS 模拟器（`x86_64` 目标平台）

让我们更新该 Gradle 脚本。

首先，我们将一个新工程添加到 `settings.gradle` 文件，只需要将下面这行代码添加到文件末尾：
<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
include ':SharedCode'
```
</div>

接下来，
我们需要使用下面的内容来创建 `SharedCode/build.gradle` 文件：
 
<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
apply plugin: 'kotlin-multiplatform'

kotlin {
    targets {
        final def iOSTarget = System.getenv('SDK_NAME')?.startsWith("iphoneos") \
                              ? presets.iosArm64 : presets.iosX64

        fromPreset(iOSTarget, 'ios') {
             binaries {
                framework('SharedCode')
            }
        }

        fromPreset(presets.jvm, 'android')
    }

    sourceSets {
        commonMain.dependencies {
            api 'org.jetbrains.kotlin:kotlin-stdlib-common'
        }

        androidMain.dependencies {
            api 'org.jetbrains.kotlin:kotlin-stdlib'
        }
    }
}

// workaround for https://youtrack.jetbrains.com/issue/KT-27170
configurations {
    compileClasspath
}

```
</div>

## 多平台 Gradle 工程  

`SharedCode/build.gradle` 文件使用了 `kotlin-multiplatform` 插件来实现<!--
-->我们所需的功能。
在这个文件中，我们定义了一些平台目标：`common`、`android` 以及 `iOS`。 每一个<!--
-->都对应它自己的平台。`common` 目标平台包含了 Kotlin 的通用代码，
它会被导入每一个平台的编译中。它允许拥有 `expect` 声明。
其它的目标为 `common` 目标中的所有 `expect`-actions 提供了 `actual` 实现。
关于更多多平台项目的细节说明可以在<!--
-->[多平台项目](/docs/reference/building-mpp-with-gradle.html)文档页中找到。

让我们用下面的表格总结一下：

| 名称 | 源路径 | 目标 | 构件 |
|---|---|---|---|
| common | `SharedCode/commonMain/kotlin` |  - | Kotlin metadata |
| android | `SharedCode/androidMain/kotlin` | JVM 6 | `.jar` file 或 `.class` files |
| iOS | `SharedCode/iosMain` | iOS arm64 or x86_64| Apple framework |

现在是时候再次在 Android Studio 中刷新这个 Gradle 工程了。在黄色条目上点击 *Sync Now*
或者在根 Gradle 工程的上下文菜单中使用 *Gradle* 工具窗口并点击 `Refresh` 按钮。
现在 `:SharedCode` 工程应该被 IDE 识别了。

我们已经准备好在我们的 Android 与 iOS 应用中使用 `SharedCode` 库了。

# 在 Android 中使用共享代码

在这部分教程中，我想将 Android 工程的改动降到最低，所以我们在主工程中添加了对
`SharedCode` 工程的普通依赖。
也可以直接在 Android Gradle 工程中使用 `kotlin-multiplatform`
插件，来代替 `kotlin-android` 插件。关于更多信息，请参考<!--
-->[多平台项目](/docs/reference/multiplatform.html)文档。  

让我们将对 `SharedCode` 工程的依赖引入 Android 工程。我们需要修改
`app/build.gradle` 文件并在 `dependencies { .. }` 块中引入下面这行代码：

<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
    implementation project(':SharedCode')
```
</div>

我们需要<!--
-->给 `TextView` 指定 `id` 以用来在我们控制它的 activity 的代码中访问它。
让我们修改
`app/src/main/res/layout/activity_main.xml` 文件<!--
-->（如果我们在新项目向导中更改了名称，则名称可能会有所不同）
并且为 `<TextView>` 元素添加几个更多的属性：
```
        android:id="@+id/main_text"
        android:textSize="42sp"
        android:layout_margin="5sp"
        android:textAlignment="center"
```

接下来，让我们在 `/app/src/main/java/<package>/MainActivity.kt` 文件的 `MainActivity`
类中将下面这行代码添加到
`onCreate` 方法的末尾：

```
findViewById<TextView>(R.id.main_text).text = createApplicationScreenMessage()
```

使用 IDE 中的联想功能来引入缺少的导入行：
```kotlin
import org.kotlin.mpp.mobile.createApplicationScreenMessage
```
到这个文件中。

现在我们拥有一个 `TextView`，它将使用可共享的函数 `createApplicationScreenMessage()`
为我们展示文本。它将显示 `Kotlin Rocks on Android`。
让我们看看它是如何工作的。

## 运行 Android 应用程序

让我们点击 `App` 运行配置<!--
-->来让我们的项目在真正的 Android 设备或模拟器上运行。

![Start the Application]({{ url_for('tutorial_img', filename='native/mpp-ios-android/studio-start-app.png') }})

现在我们可以看到应用程序运行在 Android 模拟器上。
    
![Emulator App]({{ url_for('tutorial_img', filename='native/mpp-ios-android/android-emulator-kotlin-rocks-android.png') }}){: width="30%"}


# 创建 iOS 应用程序

我们打开 Xcode 并选择 *Create a new Xcode project* 选项。在<!--
-->该对话框中，我们选择 iOS 选项并选择 *Single View App*。下一页全部使用默认选项，
然后使用 `KotlinIOS`（或其它的）作为 *Product Name*。让我们选择 Swift 作为编程语言（也可以选择使用
Objective-C）。我们应该指示 Xcode 将工程放入我们工程下的 `native` 文件夹中，稍后我们<!--
-->将在配置文件中使用相对路径。 

这个已经被创建好的 iOS 应用已经准备好可以运行在 iOS 模拟器或者 iOS 设备上。在设备上运行<!--
-->也许需要一个 Apple 开发者账号并申请一个开发者证书。Xcode
会引导我们完成整个流程。

让我们确保我们可以在 iPhone 模拟器或真实设备上运行该应用程序。


# 在 Xcode 中配置 Framework 依赖

`SharedCode` 构建生成用于 Xcode 工程的 iOS frameworks。
所有的 framework 都位于 `SharedCode/build/bin` 文件夹。
它为每个 framework 目标都创建了 *debug* 和 *release* 版本。
这些 frameworks 都位于下面的路径：
```
SharedCode/build/bin/iOS/main/debug/framework/SharedCode.framework
SharedCode/build/bin/iOS/main/release/framework/SharedCode.framework
```

我们在 Gradle 脚本中使用条件来选择 framework 的目标平台。
根据环境变量，它可能是 `iOS arm64` 或者 `iOS x86_64`。

## 修改 Gradle 构建脚本
我们需要根据 Xcode 中选定的目标从这四个中提供正确的 Framewrok
工程。它取决于在 Xcode 中选择的目标配置。当然，
我们想让 Xcode 在构建之前为我们编译 Framework。
我们需要在 `SharedCode/build.gradle` Gradle 文件的末尾包含附加 task：

<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
task packForXCode(type: Sync) {
    final File frameworkDir = new File(buildDir, "xcode-frameworks")
    final String mode = project.findProperty("XCODE_CONFIGURATION")?.toUpperCase() ?: 'DEBUG'
    final def framework = kotlin.targets.ios.binaries.getFramework("SharedCode", mode)

    inputs.property "mode", mode
    dependsOn framework.linkTask

    from { framework.outputFile.parentFile }
    into frameworkDir

    doLast {
        new File(frameworkDir, 'gradlew').with {
            text = "#!/bin/bash\nexport 'JAVA_HOME=${System.getProperty("java.home")}'\ncd '${rootProject.rootDir}'\n./gradlew \$@\n"
            setExecutable(true)
        }
    }
}
tasks.build.dependsOn packForXCode
```
</div>

注意，如果你使用 Gradle 4.10 之前的版本，
这个任务也许不能[正确地](https://github.com/gradle/gradle/issues/6330)工作。
在这篇教程中我们已经[升级到 4.7](#gradle-upgrade)。

让我们切换回 Android Studio 并在 `SharedCode` 工程的 *Gradle* 工具窗口中执行
`build` 目标。该 task 查找在 `SharedCode/build/xcode-frameworks` 文件夹下的由
Xcode 构建以及 framework 的正确变体设置的环境变量。接下来我们导入
build 文件夹下的 framework。

## 配置 Xcode

我们将 `SharedCode` framework 添加到 Xcode 工程中。
为此我们点击 *project navigator* 的根节点并选择 *target* 设置。
接下来，我们在 *Embedded Binaries* 中的 `+`，在弹窗中点击 *Add Other...* 按钮<!--
-->来在硬盘中选择 framework。我们可以指向以下文件夹：
```
SharedCode/build/xcode-frameworks/SharedCode.framework
```

接下来我们会看到如下的界面：
![Xcode General Screen]({{ url_for('tutorial_img', filename='native/mpp-ios-android/xcode-general.png') }})

现在我们需要向 Xcode 说明去哪里寻找 frameworks。我们需要添加 *相对* 路径
`$(SRCROOT)/../../SharedCode/build/xcode-frameworks` 到 *Search Paths | Framework Search Paths*。
再次打开 *Build Settings* 选项卡，选择 *All* 子选项卡，并输入 *Framework Search Paths*
搜索字段可以轻松找到该选项。
然后 Xcode 将在 UI 中显示替换路径。

![Xcode Build Settings]({{ url_for('tutorial_img', filename='native/mpp-ios-android/xcode-search-path.png') }})

最后一步是让 Xcode 调用我们的 Gradle build 任务在每次运行前准备 `SharedCode` framework。
我们打开 *Build Phases* 选项卡并点击 `+` 来添加 *New Run Script Phase* 并且将下面的代码添加进去：

<div class="sample" markdown="1" mode="bash" theme="idea" data-highlight-only="1" auto-indent="false">

```bash
cd "$SRCROOT/../../SharedCode/build/xcode-frameworks"
./gradlew :SharedCode:packForXCode -PXCODE_CONFIGURATION=${CONFIGURATION}
```
</div>

注意，这里我们使用 `$SRCROOT/../..` 作为我们的 Gradle 工程的根路径。
它取决于 Xcode 工程的创建方式。另外，我们使用生成的
`SharedCode/build/xcode-frameworks/gradlew` 脚本，
`packForXCode` 任务会生成它。在新机器上打开 Xcode 工程之前，
我们假设 Gradle 的 build 任务至少执行一次。

![Xcode Build Phases]({{ url_for('tutorial_img', filename='native/mpp-ios-android/xcode-run-script.png') }})

我们应该将创建好的 build phase 拖到列表的顶部

![Xcode Build Phases]({{ url_for('tutorial_img', filename='native/mpp-ios-android/xcode-run-script-order.png') }})

我们现在已经准备好编写 iOS 应用程序并在其中使用 Kotlin 代码

## 在 Swift 中调用 Kotlin 代码

请牢记，我们的目标是在屏幕上展示这条信息。我们可以看到，我们的 iOS 应用没有在屏幕上<!--
-->绘制任何内容。让我们使用 `UILabel` 展示这条消息。
我们需要使用下面的代码替换
`ViewController.swift` 文件中的内容：
 
<div class="sample" markdown="1" mode="swift" theme="idea" data-highlight-only="1" auto-indent="false">

```swift
import UIKit
import SharedCode

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let label = UILabel(frame: CGRect(x: 0, y: 0, width: 300, height: 21))
        label.center = CGPoint(x: 160, y: 285)
        label.textAlignment = .center
        label.font = label.font.withSize(25)
        label.text = CommonKt.createApplicationScreenMessage()
        view.addSubview(label)
    }
}

```
</div>
 
我们使用 `import SharedCode` 来导入我们使用 Kotlin/Native 编译的 Kotlin 代码而生成的 Framework。
接下来，我们调用库中的 Kotlin 函数作为`CommonKt.createApplicationScreenMessage()`。然后
[Kotlin/Native 开发 Apple Framework](/docs/tutorials/native/apple-framework.html) 教程中<!--
-->有更多关于 Kotlin/Native 与 Swift（或 Objective-C）互操作的细节。

现在，我们已准备好在模拟器或 iOS 设备上启动应用程序。

## 运行 iOS 应用程序

让我们在 Xcode 中点击 *Run* 按钮，接下来我们将看到应用程序运行

![Emulator App]({{ url_for('tutorial_img', filename='native/mpp-ios-android/iPhone-emulator-kotlin-rocks.png') }}){: width="30%"}

# 总结

在本篇教程中我们：
 - 在 Android Studio 中创建了一个 Android 应用程序
 - 在 Xcode 中创建了一个 iOS 应用程序
 - 添加了 Kotlin 多平台项目子工程 
   - 共享 Kotlin 代码
   - 将它编译成 Android Jar
   - 将它编译成 iOS Framework
 - 将它们放在一起并复用 Kotlin 代码
 
我们可以在 [GitHub](https://github.com/JetBrains/kotlin-examples/tree/master/tutorials/mpp-iOS-Android) 上找到这篇教程的所有源码。

# 接下来

这只是一个开始，并且仅仅是一个关于在
iOS 与 Android（以及其它平台）上使用 Kotlin、Kotlin/Native
以及 Kotlin 多平台项目的小示例。相同的方法适用于真实的应用程序，
而与它们的大小以及复杂度无关。

Kotlin/Native 与 Swift 以及 Objective-C 的互操作被包含在这篇<!--
-->[文档](/docs/reference/native/objc_interop.html)中。另外，
类似的话题被包含在 [Kotlin/Native 开发 Apple Framework](apple-framework.html)
教程中。

多平台项目以及多平台库在这篇[文档](/docs/reference/multiplatform.html)中讨论。

在平台之间共享代码是一种强大的技术，但是它可能很难在
Android、JVM 或 iOS 这些拥有丰富不同 API 的平台上办到。
而多平台库可以被用来解决这一问题。它们直接使用<!--
-->通用的 Kotlin 代码来提供丰富的 API。举例来说我们有如下的多平台库：

- [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines/blob/master/native/README.md)
- [kotlinx.io](https://github.com/Kotlin/kotlinx-io)
- [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization)
- [ktor](https://ktor.io/)
- [ktor-http-client](https://ktor.io/clients/http-client.html)

想了解更多 API？来轻松的创建一个[多平台项目](/docs/tutorials/multiplatform-library.html)并共享它吧！
