---
type: tutorial
layout: tutorial
title:  "多平台项目: iOS 与 Android"
description: "在 iOS 与 Android 之间共享 Kotlin 代码"
authors: Eugene Petrenko
date: 2018-10-04
showAuthorInfo: false
issue: EVAN-6029
---

在本教程中我们将通过创建一个 iOS 与一个 Android 应用，来展示 Kotlin 代码的共享功能。
在 Android 上我们将使用 Kotlin/JVM，而在 iOS 上将是 Kotlin/Native。

我们将学习到如何：
 - 使用 Android Studio 创建一个 [Android app](#创建一个-Android-工程)
 - 创建一个共享的 [Kotlin library](#创建共享模块)
   - 在 [Android app](#在-android-中使用共享代码) 中使用它
   - 运行 [Android 应用程序](#运行-android-应用程序)
 - 使用 Xcode 创建一个 [iOS app](#creating-ios-application)
   - 在 [iOS app](#setting-up-framework-dependency-in-xcode) 上使用共享的 Kotlin 库
   - 在 [Swift 中使用 Kotlin](#在-swift-中调用-kotlin-代码)
   - 运行 [iOS 应用程序](#running-the-ios-application)

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
[Apple 开发者网站](https://developer.apple.com/xcode/) 来获取更多细节。

*注意：我们将使用 IntelliJ IDEA 2018.3 EAP、Android Studio 3.2、Kotlin 1.3.21、Xcode 10.0、macOS 10.14、Gradle 4.7*

# 创建一个 Android 工程

我们将通过 *Start New Android Project* 来创建一个 Android 工程。如果使用 IntelliJ IDEA，我们需要在左边的 *New Project* 
向导面板中选择 *Android*。

重要的一点是你需要确保勾选了 *Include Kotlin support* 选择框。现在我们可以在向导的下一步中<!--
-->保留默认设置。我们接下来选择 *Empty Activity* 选项并点击 *Next*，最后点击 *Finish*。

**注意** 如果使用早期发行版或者 EAP 版本的 Kotlin plugin，IDE 在生成工程的时候可能会失败， 
给 Gradle 抛出 [error](https://youtrack.jetbrains.com/issue/KT-18835#focus=streamItem-27-2718879-0-0)。
这是因为 `build.gradle` 文件中没有引用正确的 Maven 库，在每一个 `repositories { .. }`
块中它可以通过添加来解决以下条目 *两次*。

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

此刻，我们应该可以编译并运行这个 Android 应用了。

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

我们为 iOS 创建了一个相似的文件 `SharedCode/src/iosMain/kotlin/actual.kt`：
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

`SharedCode` 应该为我们生成一系列产物：
 - 在 `androidMain` 源集中为 Android 工程生成 JAR 文件
 - Apple framework 
   - 面向 iOS 设备以及 App Store (`arm64` target)
   - 面向 iOS emulator (`x86_64` target)

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
在这个文件中，我们定义了一些目标：`common`、`android` 以及 `iOS`。 每一个<!--
-->都有它自己的平台。`common` 目标包含了 Kotlin 的通用代码，
它会被导入每一个平台的编译中。它允许拥有 `expect` 声明。
其它的目标为 `common` 目标中的所有 `expect`-actions 提供了 `actual` 实现。
关于更多多平台项目的细节说明可以在<!--
-->[多平台项目](/docs/reference/building-mpp-with-gradle.html)文档页中找到。

让我们用下面的表格总结一下：

| 名称 | 源路径 | 目标 | 产物 |
|---|---|---|---|
| common | `SharedCode/commonMain/kotlin` |  - | Kotlin metadata |
| android | `SharedCode/androidMain/kotlin` | JVM 6 | `.jar` file or `.class` files |
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
到类似的文件中。

现在我们拥有一个 `TextView`，它将使用可共享的代码函数 `createApplicationScreenMessage()`
为我们展示文本。它将显示 `Kotlin Rocks on Android`。
让我们看看它是如何工作的。

## 运行 Android 应用程序

让我们点击 `App` 运行配置<!--
-->来让我们的项目在真正的 Android 设备或模拟器上运行。

![Start the Application]({{ url_for('tutorial_img', filename='native/mpp-ios-android/studio-start-app.png') }})

现在我们可以看到应用程序运行在 Android 模拟器上。
    
![Emulator App]({{ url_for('tutorial_img', filename='native/mpp-ios-android/android-emulator-kotlin-rocks-android.png') }}){: width="30%"}


# Creating iOS Application

We open Xcode and select *Create a new Xcode project* option. In 
the dialog, we choose the iOS target and select the *Single View App*. Fill the next page with defaults, 
and use the `KotlinIOS` (or something else) as the *Product Name*. Let's select Swift as the language (it is possible to use
Objective-C too). We should instruct Xcode to place the project into the `native` folder under our project, later we
will use relative paths in the configuration files. 

The created iOS application is ready to run on the iOS emulator or on the iOS device. The device run
may require an Apple developer account and to issue a developer certificate. Xcode does its
best to guide us through the process. 

Let's make sure we can run the application on the iPhone emulator or device. 


# Setting up Framework Dependency in Xcode

The `SharedCode` build generates iOS frameworks for use with the Xcode project.
All frameworks are in the `SharedCode/build/bin` folder. 
It creates a *debug* and *release* version for every framework target.
The frameworks are in the following paths:
```
SharedCode/build/bin/iOS/main/debug/framework/SharedCode.framework
SharedCode/build/bin/iOS/main/release/framework/SharedCode.framework
```

We use the condition in the Gradle script to select the target platform for the framework.
It is either `iOS arm64` or `iOS x86_64` depending on environment variables.

## Tuning the Gradle Build Script
We need to supply the right Framework out of those four depending on the selected target in the Xcode
project. It depends on the target configuration selected in Xcode. Also,
we'd like to make Xcode compile the Framework for us before the build.
We need to include the additional task to the end of the `SharedCode/build.gradle` Gradle file:

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

Note, the task may not work [correctly](https://github.com/gradle/gradle/issues/6330)
if you use Gradle older than 4.10. 
In this tutorial we have already [upgraded it to 4.7](#gradle-upgrade).

Let's switch back to the Android Studio and execute the `build` target of the `SharedCode` project from
the *Gradle* tool window. The task looks for environment variables set by the Xcode build and copies
the right variant of the framework into the `SharedCode/build/xcode-frameworks` folder. We then include the
framework from that folder into the build

## Setting up Xcode

We add the `SharedCode` framework to the Xcode project.
For that let's click on the root node of the *project navigator* and select the *target* settings.
Next, we click on the `+` in the *Embedded Binaries* section, click *Add Other...* button in the dialog
to choose the framework from the disk. We can point to the following folder: 
```
SharedCode/build/xcode-frameworks/SharedCode.framework
```

We will then see something similar to this: 
![Xcode General Screen]({{ url_for('tutorial_img', filename='native/mpp-ios-android/xcode-general.png') }})

Now we need to explain to Xcode, where to look for frameworks. We need to add the *relative* path 
`$(SRCROOT)/../../SharedCode/build/xcode-frameworks` into the *Search Paths | Framework Search Paths* section.
Open the *Build Settings* tab again, pick the *All* sub-tab below, and type the *Framework Search Paths* into
the search field to easily find the option.
Xcode will then show the substituted path in the UI for it.

![Xcode Build Settings]({{ url_for('tutorial_img', filename='native/mpp-ios-android/xcode-search-path.png') }})

The final step is to make Xcode call our Gradle build to prepare the `SharedCode` framework before each run.
We open the *Build Phases* tab and click `+` to add the *New Run Script Phase* and add the following code into it:

<div class="sample" markdown="1" mode="bash" theme="idea" data-highlight-only="1" auto-indent="false">

```bash
cd "$SRCROOT/../../SharedCode/build/xcode-frameworks"
./gradlew :SharedCode:packForXCode -PXCODE_CONFIGURATION=${CONFIGURATION}
```
</div>

Note, here we use the `$SRCROOT/../..` as the path to the root of our Gradle project.
It can depend on the way the Xcode project was created. Also, we use the generated
`SharedCode/build/xcode-frameworks/gradlew` script,
the `packForXCode` task generates it. We assumed that the Gradle build is executed at least once,
before opening the Xcode project on a fresh machine

![Xcode Build Phases]({{ url_for('tutorial_img', filename='native/mpp-ios-android/xcode-run-script.png') }})

We should drag the created build phase to the top of the list

![Xcode Build Phases]({{ url_for('tutorial_img', filename='native/mpp-ios-android/xcode-run-script-order.png') }})

We are now ready to start coding the iOS application and to use the Kotlin code from it

## 在 Swift 中调用 Kotlin 代码

Remember, our goal is to show the text message on the screen. As we see, our iOS application does not draw
anything on the screen. Let's make it show the `UILabel` with the text message. 
We need to replace the contents
of the `ViewController.swift` file with the following code:
 
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
 
We use the `import SharedCode` to import our Framework, which we compiled with Kotlin/Native from Kotlin code.
Next, we call the Kotlin function from it as `CommonKt.createApplicationScreenMessage()`. Follow the 
[Kotlin/Native as an Apple Framework](/docs/tutorials/native/apple-framework.html) tutorial for
more details on the Kotlin/Native to Swift (or Objective-C) interop.

Right now, we are ready to start the application in the emulator or on an iOS device.

## Running the iOS Application

Let's click the *Run* button in Xcode, and we'll see our application running 

![Emulator App]({{ url_for('tutorial_img', filename='native/mpp-ios-android/iPhone-emulator-kotlin-rocks.png') }}){: width="30%"}

# Summary

In the tutorial we:
 - created an Android application in Android Studio
 - created an iOS application in Xcode
 - added Kotlin multiplatform sub-project  
   - with shared Kotlin code
   - compiled it to Android Jar
   - compiled it to iOS Framework
 - put it all together and re-used Kotlin code
 
We may find the whole sources from that tutorial on [GitHub](https://github.com/JetBrains/kotlin-examples/tree/master/tutorials/mpp-iOS-Android).

# Next Steps

This is only the beginning and a small example of Kotlin code sharing
between iOS and Android (and other platforms) with Kotlin, Kotlin/Native
and Kotlin multiplatform projects. The same approach works for real applications,
independent of their size or complexity.

Kotlin/Native interop with Swift and Objective-C is covered in the
[documentation](/docs/reference/native/objc_interop.html) article. Also,
the same topic is covered in the [Kotlin/Native as an Apple Framework](apple-framework.html)
tutorial.

The multiplatform projects and multiplatform libraries are discussed in the [documentation](/docs/reference/multiplatform.html) too.

Sharing code between platforms is a powerful technique, but it may be hard to
accomplish without rich APIs that we have in Android, JVM, or iOS platforms.
Multiplatform libraries can be used to fix that. They bring rich APIs
directly in the common Kotlin code. There are several examples of such libraries:  

- [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines/blob/master/native/README.md)
- [kotlinx.io](https://github.com/Kotlin/kotlinx-io)
- [kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization)
- [ktor](https://ktor.io/)
- [ktor-http-client](https://ktor.io/clients/http-client.html)

Looking for more APIs? It is easy to create a [multiplatform library](/docs/tutorials/multiplatform-library.html) and share it!
