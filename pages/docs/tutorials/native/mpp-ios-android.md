---
type: tutorial
layout: tutorial
title:  "多平台项目: iOS 与 Android"
description: "在 iOS 与 Android 之间共享 Kotlin 代码"
authors: Eugene Petrenko，乔禹昂（翻译）
date: 2019-08-11
showAuthorInfo: true
issue: EVAN-6029
---

在本教程中，我们会创建一个 iOS 与 Android 两用的应用，来展示 Kotlin 代码的共享能力。
对于 Android，我们会使用 Kotlin/JVM，而对于 iOS 会使用 Kotlin/Native。

我们将学习到如何：
 - 使用 Android Studio 创建一个 [Android app](#创建一个-Android-项目)
 - 创建一个共享的 [Kotlin library](#创建共享模块)
   - 在 [Android app](#在-android-中使用共享代码) 中使用 Kotlin 库
   - 运行 [Android 应用程序](#运行-android-应用程序)
 - 使用 Xcode 创建一个 [iOS app](#创建-ios-应用程序)
   - 在 [iOS app](#在-xcode-中配置-framework-依赖) 上使用共享的 Kotlin 库
   - 在 [Swift 中调用 Kotlin](#在-swift-中调用-kotlin-代码)
   - 运行 [iOS 应用程序](#运行-ios-应用程序)

我们即将创建的应用程序会在 Android 设备上显示
`Kotlin Rocks on Android` 而在 iOS 设备上显示 `Kotlin Rocks on iOS <version>`。
Our goal is to demonstrate the ability to share Kotlin code between the platforms, the project setup, and the benefits that
this provides. While we'll be demonstrating this with a simple application, what is shown here can be applied to real-world applications, no matter their size or complexity.

通用的代码是 `"Kotlin Rocks on ${platformName()}"`，`platformName()`
是一个使用 `expect` 关键字声明的函数。而 `actual` 的实现将根据特定的平台而异。

# 搭建本地环境

* 我们将使用 [Android Studio](https://developer.android.com/studio/) 来讲解 Android 部分的内容.
当然也可以使用 [IntelliJ IDEA](https://jetbrains.com/idea) 社区版或终级版。

* IDE 应该安装了 Kotlin 插件 {{ site.data.releases.latest.version }} 或更高版本。这个可以通过
*Settings*（或 *Preferences*）窗口中的 *Language & Frameworks | Kotlin Updates* 部分验证。

* 编译 iOS 以及 macOS 设备的代码需要 masOS 系统。我们需要安装以及配置
[Xcode](https://developer.apple.com/xcode/) 工具。查看
[Apple 开发者网站](https://developer.apple.com/xcode/)来获取更多细节。

*注意：我们会使用 IntelliJ IDEA 2019.2、Android Studio 3.4、
Kotlin {{ site.data.releases.latest.version }}、Xcode 10.3、macOS 10.14、Gradle 5.5.1*

# 创建一个 Android 项目

We need Android Studio for the tutorial. We can download and install it from the
[https://developer.android.com/studio/](https://developer.android.com/studio/). Let's open
the IDE and check that we see the newest Kotlin version, namely 
{{ site.data.releases.latest.version }}
or newer under the Kotlin section _Languages & Frameworks_ | _Kotlin_
in the _Settings_ (or _Preferences_) dialog of Android Studio.

Our first step is to create a new Android project via the *Start a new Android project* item on the Android Studio home screen. 
We then proceed to select the *Empty Activity* option and click *Next*. It's important to pick the _Kotlin_
language in the wizard. Let's use the `com.jetbrains.handson.mpp.mobile` package
for the tutorial. Now we can press the *Finish* button and create our new Android project.

At this point, we should be able to compile and run the Android application. Let's check that it works!

# 创建共享模块

本教程的目标是演示在 Android 与 iOS 之间 Kotlin 代码的复用性。我们了来以在
Gradle 项目中手动创建一个 `SharedCode` 子项目开始。`SharedCode` 项目的源代码<!--
-->会在两个平台之间共享。
我们会在项目中创建几个新文件来实现这个目标。

## Updating Gradle Scripts

The `SharedCode` sub-project should generate several artifacts for us:
 - A JAR file for the Android project, from the `androidMain` source set
 - The Apple framework 
   - for iOS device and App Store (`arm64` target)
   - for iOS simulator (`x86_64` target)

Let's update the Gradle scripts now to implement this and configure our IDE.
First, we add the new project to the `settings.gradle` file, simply by adding the following line to the end of the file:
<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
include ':SharedCode'
```
</div>

Next,
we need to create a `SharedCode/build.gradle.kts` file with the following content:
 
<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
import org.jetbrains.kotlin.gradle.plugin.mpp.KotlinNativeTarget

plugins {
    kotlin("multiplatform")
}

kotlin {
    //select iOS target platform depending on the Xcode environment variables
    val iOSTarget: (String, KotlinNativeTarget.() -> Unit) -> KotlinNativeTarget =
        if (System.getenv("SDK_NAME")?.startsWith("iphoneos") == true)
            ::iosArm64
        else
            ::iosX64

    iOSTarget("ios") {
        binaries {
            framework {
                baseName = "SharedCode"
            }
        }
    }

    jvm("android")

    sourceSets["commonMain"].dependencies {
        implementation("org.jetbrains.kotlin:kotlin-stdlib-common")
    }

    sourceSets["androidMain"].dependencies {
        implementation("org.jetbrains.kotlin:kotlin-stdlib")
    }
}

val packForXcode by tasks.creating(Sync::class) {
    val targetDir = File(buildDir, "xcode-frameworks")

    /// selecting the right configuration for the iOS 
    /// framework depending on the environment
    /// variables set by Xcode build
    val mode = System.getenv("CONFIGURATION") ?: "DEBUG"
    val framework = kotlin.targets
                          .getByName<KotlinNativeTarget>("ios")
                          .binaries.getFramework(mode)
    inputs.property("mode", mode)
    dependsOn(framework.linkTask)

    from({ framework.outputDirectory })
    into(targetDir)

    /// generate a helpful ./gradlew wrapper with embedded Java path
    doLast {
        val gradlew = File(targetDir, "gradlew")
        gradlew.writeText("#!/bin/bash\n" 
            + "export 'JAVA_HOME=${System.getProperty("java.home")}'\n" 
            + "cd '${rootProject.rootDir}'\n" 
            + "./gradlew \$@\n")
        gradlew.setExecutable(true)
    }
}

tasks.getByName("build").dependsOn(packForXcode)
```
</div>

We need to refresh the Gradle project to apply these changes. Click on the `Sync Now` link or 
use the *Gradle* tool window and click the refresh action from the context menu on the root Gradle project.
The `packForXcode` Gradle task is used for Xcode project integration. We will discuss this later in the
tutorial.  

## 添加 Kotlin 源码

我们想要使每个平台都根据平台自身展示相似的文本：`Kotlin Rocks on Android` 以及
`Kotlin Rocks on iOS`。我们将复用生成消息的方式。
让我们在项目根目录下用以下内容创建文件（以及缺少的目录）：
`SharedCode/src/commonMain/kotlin/common.kt`。

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
package com.jetbrains.handson.mpp.mobile

expect fun platformName(): String

fun createApplicationScreenMessage() : String {
  return "Kotlin Rocks on ${platformName()}"
}

```
</div>

这是通用的部分。这段代码生成了最终的消息。它期待（`expect`）平台部分<!--
-->提供来自 `expect fun platformName（）：String` 函数的平台相关名称。我们会同时在
Android 与 iOS 应用中使用 `createApplicationScreenMessage`。

现在我们需要在 `SharedCode/src/androidMain/kotlin/actual.kt` 中为 Android 创建相应的实现文件（与缺失的目录）：
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
package com.jetbrains.handson.mpp.mobile

actual fun platformName(): String {
  return "Android"
}

```
</div>

我们为 iOS 也创建了一个相似的实现文件（与缺失的目录） `SharedCode/src/iosMain/kotlin/actual.kt`：
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
package com.jetbrains.handson.mpp.mobile

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
Kotlin/Native 编译器带有一组预先导入的 framework，所以我们可以<!--
-->使用 UIKit Framework 而无需任何额外步骤。
Objective-C 与 Swift 互操作在这篇[文档](/docs/reference/native/objc_interop.html)中有详细介绍

## 多平台 Gradle 项目

`SharedCode/build.gradle.kts` 文件使用了 `kotlin-multiplatform` 插件来实现<!--
-->我们所需的功能。
在这个文件中，我们定义了一些平台目标：`common`、`android` 以及 `iOS`。 每一个<!--
-->都对应它自己的平台。`common` 目标平台包含了公共 Kotlin 代码，
它会被导入每一个平台的编译中。它允许拥有 `expect` 声明。
其它的目标为 `common` 目标中的所有 `expect` 函数提供了 `actual` 实现。
关于更多多平台项目的细节说明可以在<!--
-->[多平台项目](/docs/reference/building-mpp-with-gradle.html)文档页中找到。

让我们用下面的表格总结一下：

| 名称 | 源路径 | 目标 | 构件 |
|---|---|---|---|
| common | `SharedCode/commonMain/kotlin` |  - | Kotlin metadata |
| android | `SharedCode/androidMain/kotlin` | JVM 1.6 | `.jar` file 或 `.class` files |
| iOS | `SharedCode/iosMain` | iOS arm64 or x86_64| Apple framework |

现在是时候再次在 Android Studio 中刷新这个 Gradle 项目了。在黄色条目上点击 *Sync Now*
或者在根 Gradle 项目的上下文菜单中使用 *Gradle* 工具窗口并点击 `Refresh` 按钮。
现在 `:SharedCode` 项目应该被 IDE 识别了。

We can use the `step-004` branch from the 
[github.com/kotlin-hands-on/mpp-ios-android](https://github.com/kotlin-hands-on/mpp-ios-android/tree/step-004)
repository as a solution for the tasks that we've done above. We can also download the
[archive](https://github.com/kotlin-hands-on/mpp-ios-android/archive/step-004.zip) from GitHub directly
or check out the repository and select the branch.

Let's use the `SharedCode` library from our Android and iOS applications.

# 在 Android 中使用共享代码

在这部分教程中，我想将 Android 项目的改动降到最低，所以我们在主项目中添加了对
`SharedCode` 项目的普通依赖。
也可以直接在 Android Gradle 项目中使用 `kotlin-multiplatform`
插件，来代替 `kotlin-android` 插件。关于更多信息，请参考<!--
-->[多平台项目](/docs/reference/multiplatform.html)文档。

让我们将对 `SharedCode` 项目的依赖引入 Android 项目。我们需要修改
`app/build.gradle` 文件并在 `dependencies { .. }` 块中加入以下这行代码：

<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
    implementation project(':SharedCode')
```
</div>

我们需要<!--
-->给 `TextView` 指定 `id` 以用来在我们控制它的 activity 的代码中访问它。
让我们修改
`app/src/main/res/layout/activity_main.xml` 文件<!--
-->（如果我们在新项目向导中更改了名称，则名称可能会有所不同）。
Select the _Text_ tab at the bottom of the preview to switch it to XML
并且为 `<TextView>` 元素添加几个更多的属性：
```
        android:id="@+id/main_text"
        android:textSize="42sp"
        android:layout_margin="5sp"
        android:textAlignment="center"
```

Next, let's add the following line of code to the end of the `onCreate` method from the `MainActivity` class
in the `/app/src/main/java/com/jetbrains/handson/mpp/mobile/MainActivity.kt` file, :

```
findViewById<TextView>(R.id.main_text).text = createApplicationScreenMessage()
```

You will need to add the import for the `android.widget.TextView` class. Android Studio
will automatically suggest adding the import. Depending on the Android application package,
we may also need to add the import for the `createApplicationScreenMessage()` function too.
We should see these two lines at the beginning of the `MainActivity.kt` file:
```kotlin
import com.jetbrains.handson.mpp.mobile.createApplicationScreenMessage
import android.widget.TextView
```

现在我们拥有一个 `TextView`，它将使用可共享的函数 `createApplicationScreenMessage()`
为我们展示文本。它将显示 `Kotlin Rocks on Android`。
我们通过运行该 Android 程序来看下它是如何工作的。

We can use the `step-005` branch of the 
[github.com/kotlin-hands-on/mpp-ios-android](https://github.com/kotlin-hands-on/mpp-ios-android/tree/step-005)
repository as a solution for the tasks we've done above. We can also download the
[archive](https://github.com/kotlin-hands-on/mpp-ios-android/archive/step-005.zip) from GitHub directly
or check out the repository and select the branch.

## 运行 Android 应用程序

让我们点击 `App` 运行配置<!--
-->来让我们的项目在真正的 Android 设备或模拟器上运行。

![Start the Application]({{ url_for('tutorial_img', filename='native/mpp-ios-android/studio-start-app.png') }})

现在我们可以看到应用程序运行在 Android 模拟器上。
    
![Emulator App]({{ url_for('tutorial_img', filename='native/mpp-ios-android/android-emulator-kotlin-rocks-android.png') }}){: width="30%"}


# 创建 iOS 应用程序

We can start by opening Xcode and selecting the *Create a new Xcode project* option. In 
the dialog, we need to choose the iOS target and select the *Single View App* and click next. Fill in the next page with the defaults, 
and use `KotlinIOS` as the *Product Name*. Let's select _Swift_ as the language (it is possible to use
Objective-C too). Use the `com.jetbrains.handson.mpp.mobile` string for the _Organization Identifier_ field.
Now the _Next_ button should be available, let's click it to move on.
In the file dialog, shown after clicking the _Next_ button, we need to select the root folder
of our project, click the _New Folder_ button and create a folder called `native` in it. 
The folder should be selected now, and we can click the _Create_
button to complete the dialog. We will use relative paths in the configuration files later in this tutorial. 

The created iOS application is ready to run on the iOS simulator or iOS device. For the device to run
it may require an Apple developer account and a developer certificate. Xcode does its
best to get us through the process. 

Let's make sure we can run the application on the iPhone simulator or device by clicking the play button
from the XCode window title bar. 

The `step-006` branch of the 
[github.com/kotlin-hands-on/mpp-ios-android](https://github.com/kotlin-hands-on/mpp-ios-android/tree/step-006)
repository contains a possible solution for the tasks that we have done above.  We can also download the
[archive](https://github.com/kotlin-hands-on/mpp-ios-android/archive/step-006.zip) from GitHub directly or
check out the repository and select the branch.

# 在 Xcode 中配置 Framework 依赖

Let's run the `packForXcode` Gradle task of the `SharedCode` project. We can do it either from
the _Gradle_ tab in AndroidStudio or by running the `./gradlew :SharedCode:packForXcode` command
from console. This task is designed to help to simplify the setup of our iOS Framework
in the Xcode project model.

We need several binaries from the framework to use it with Xcode:
- `iOS arm64 debug` --- the binary to run the iOS device in debug mode
- `iOS arm64 release` --- the binary to include into a release version of an app
- `iOS x64 debug` --- the binary for iOS simulator, which uses the desktop mac CPU

The easiest way to configure Xcode to use a custom-built framework is to
place the framework under the same folder for all configurations and targets.
Then add a custom step to the Xcode project build to update or build the
framework before the actual Xcode build is started. Xcode sets several environment
variables for custom steps; we can use these variables in the Gradle script to select
the requested target platform and the configuration. For more details,
please refer to the `packForXcode` task sources in the `SharedCode/build.gradle.kts` file. 

Let's now switch back to Android Studio and execute the `build` target of the `SharedCode` project from
the *Gradle* tool window. The task looks for environment variables set by the Xcode build and copies
the right variant of the framework to the `SharedCode/build/xcode-frameworks` folder. We can then include the
framework from that folder into the build

## 配置 Xcode

We need to add the `SharedCode` framework to the Xcode project.
To do this we can double-click on the `KotlinIOS` node (or root node) of the *project navigator (⌘1)* tree
to open the *target* settings.
Next we then click on the `+` in the *Embedded Binaries* section, click *Add Other...* button in the dialog
to choose the framework from the disk. We can then point to the following folder: 
```
SharedCode/build/xcode-frameworks/SharedCode.framework
```

In the next dialog, we need to select the _Create folder references_ option and make sure that the _Copy items if needed_
checkbox isn't checked. We should then see something like this: 
![Xcode General Screen]({{ url_for('tutorial_img', filename='native/mpp-ios-android/xcode-general.png') }})

Now we need to explain to Xcode, where to look for the framework.
For this, we open the *Build Settings* tab again, pick the *All* sub-tab below, and type the *Framework Search Paths* into
the search field to find the option easily. We need to add the *relative* path 
`$(SRCROOT)/../../SharedCode/build/xcode-frameworks` to the *Search Paths | Framework Search Paths* section.
Xcode will then show the substituted path in the UI for it.

![Xcode Build Settings]({{ url_for('tutorial_img', filename='native/mpp-ios-android/xcode-search-path.png') }})

最后一步是让 Xcode 调用我们的 Gradle build 任务在每次运行前准备 `SharedCode` framework。
We can open the *Build Phases* tab and click `+` to add the *New Run Script Phase*, drag that step to the very
first position, and add the following code to the shell script text:

<div class="sample" markdown="1" mode="bash" theme="idea" data-highlight-only="1" auto-indent="false">

```bash
cd "$SRCROOT/../../SharedCode/build/xcode-frameworks"
./gradlew :SharedCode:packForXCode -PXCODE_CONFIGURATION=${CONFIGURATION}
```
</div>

注意，这里我们使用 `$SRCROOT/../..` 作为我们的 Gradle 项目的根路径。
它取决于 Xcode 项目的创建方式。另外，我们使用生成的
`SharedCode/build/xcode-frameworks/gradlew` 脚本，
`packForXCode` 任务会生成它。在新机器上打开 Xcode 项目*之前*，
我们假设 Gradle 的 build 任务至少执行一次。

The `step-007` branch of the 
[github.com/kotlin-hands-on/mpp-ios-android](https://github.com/kotlin-hands-on/mpp-ios-android/tree/step-007)
repository contains a possible solution for the tasks that we have done above. We can also download the
[archive](https://github.com/kotlin-hands-on/mpp-ios-android/archive/step-007.zip) from GitHub directly or
check out the repository and select the branch.

![Xcode Build Phases]({{ url_for('tutorial_img', filename='native/mpp-ios-android/xcode-run-script.png') }})

我们应该将创建好的 build phase 拖到列表的顶部

![Xcode Build Phases]({{ url_for('tutorial_img', filename='native/mpp-ios-android/xcode-run-script-order.png') }})

We are now ready to start coding the iOS application with Kotlin

## 在 Swift 中调用 Kotlin 代码

请牢记，我们的目标是在屏幕上展示这条信息。我们可以看到，我们的 iOS 应用没有在屏幕上<!--
-->绘制任何内容。让我们使用 `UILabel` 展示这条消息。
Let's open the `ViewController.swift` file from the *project navigator (⌘1)* tree.
我们需要使用下面的代码替换 `ViewController.swift` 文件中的内容：
 
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

The `step-008` branch of the 
[github.com/kotlin-hands-on/mpp-ios-android](https://github.com/kotlin-hands-on/mpp-ios-android/tree/step-008)
repository contains a possible solution for the tasks that we have done above. We can also download the
[archive](https://github.com/kotlin-hands-on/mpp-ios-android/archive/step-008.zip) from GitHub directly or
check out the repository and select the branch.

现在，我们已准备好在模拟器或 iOS 设备上启动应用程序。

## 运行 iOS 应用程序

让我们在 Xcode 中点击 *Run* 按钮，接下来我们将看到应用程序运行

![Emulator App]({{ url_for('tutorial_img', filename='native/mpp-ios-android/iPhone-emulator-kotlin-rocks.png') }}){: width="30%"}

# 总结

在本篇教程中我们：
 - 在 Android Studio 中创建了一个 Android 应用程序
 - 在 Xcode 中创建了一个 iOS 应用程序
 - 添加了 Kotlin 多平台项目子项目
   - 共享 Kotlin 代码
   - 将它编译成 Android Jar
   - 将它编译成 iOS Framework
 - 将它们放在一起并复用 Kotlin 代码
 
我们可以在 [GitHub](https://github.com/JetBrains/kotlin-examples/tree/master/tutorials/mpp-iOS-Android) 上找到这篇教程的所有源码。

# 接下来

This small example of Kotlin code sharing between iOS and Android (and other platforms) 
with Kotlin, Kotlin/Native, and Kotlin multiplatform projects is only the beginning. 
The same approach can work for real applications, independent of their size or complexity.

Kotlin/Native 与 Swift 以及 Objective-C 的互操作被包含在这篇<!--
-->[文档](/docs/reference/native/objc_interop.html)中。
[Kotlin/Native 开发 Apple Framework](apple-framework.html) 教程也涵盖这个主题<!--
-->。

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
