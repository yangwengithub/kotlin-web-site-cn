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

我们将学习到如何去：
 - 使用 Android Studio 创建一个 [Android app](#创建一个-Android-工程)
 - 创建一个共享的 [Kotlin library](#creating-the-shared-module)
   - 在 [Android app](#using-sharedcode-from-android) 中使用它
   - 运行 [Android application](#running-the-android-application)
 - 使用 Xcode 创建一个 [iOS app](#creating-ios-application)
   - 在 [iOS app](#setting-up-framework-dependency-in-xcode) 上使用共享的 Kotlin library
   - 使用 [Kotlin from Swift](#calling-kotlin-code-from-swift)
   - 运行 [iOS application](#running-the-ios-application)

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
给 Gradle 导入 [error](https://youtrack.jetbrains.com/issue/KT-18835#focus=streamItem-27-2718879-0-0)。
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

# Creating the Shared Module

The goal of the tutorial is to demonstrate Kotlin code re-use between Android and iOS. Let's start
by creating the `SharedCode` sub-project in our Gradle project. The source code from the `SharedCode`
project will be shared between platforms.
We will create several new files in our project to implement that.

## Adding Kotlin Sources

The idea is to make every platform show similar text: `Kotlin Rocks on Android` and 
`Kotlin Rocks on iOS`, depending on the platform. We will reuse the way we generate the message. 
Let's create the main file under `SharedCode/src/commonMain/kotlin/common.kt`

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
package org.kotlin.mpp.mobile

expect fun platformName(): String

fun createApplicationScreenMessage() : String {
  return "Kotlin Rocks on ${platformName()}"
}

```
</div>

That is the common part. The code to generate the final message. It `expect`s the platform
to provide the platform name from the `expect fun platformName(): String` function. We will use
the `createApplicationScreenMessage` from both Android and iOS applications.

Now, we need to create the implementation for Android in the `SharedCode/src/androidMain/kotlin/actual.kt`:
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
package org.kotlin.mpp.mobile

actual fun platformName(): String {
  return "Android"
}

```
</div>

We create a similar file for the iOS target in the `SharedCode/src/iosMain/kotlin/actual.kt`:
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

Here we can use the [UIDevice](https://developer.apple.com/documentation/uikit/uidevice?language=objc)
class from the Apple UIKit Framework, which is not available in Java, it is only usable in Swift and Objective-C.
Kotlin/Native compiler comes with a set of pre-imported frameworks, so we can use
the UIKit Framework without additional steps.
Objective-C and Swift Interop is covered in details in the [documentation](/docs/reference/native/objc_interop.html)

## Updating Gradle Scripts

The `SharedCode` project should generate several artifacts for us:
 - JAR file for the Android project, from the `androidMain` source set
 - Apple framework 
   - for iOS device and App Store (`arm64` target)
   - for iOS emulator (`x86_64` target)

Let's update the Gradle scripts. 

First, we add the new project into the `settings.gradle` file, simply by adding the following line to the end of the file:
<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
include ':SharedCode'
```
</div>

Next,
we need to create the `SharedCode/build.gradle` file with the following content:
 
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

## Multiplatform Gradle Project

The `SharedCode/build.gradle` file uses the `kotlin-multiplatform` plugin to implement 
what we need. 
In the file, we define several targets `common`, `android`, and `iOS`. Each
target has its own platform. The `common` target contains the Kotlin common code 
which is included into every platform compilation. It is allowed to have `expect` declarations.
Other targets provide `actual` implementations for all `expect`-actions from the `common` target. 
The more detailed explanation of the multiplatform projects can be found on the
[Multiplatform Projects](/docs/reference/building-mpp-with-gradle.html) documentation page.

Let's summarize what we have in the table:

| name | source folder | target | artifact |
|---|---|---|---|
| common | `SharedCode/commonMain/kotlin` |  - | Kotlin metadata |
| android | `SharedCode/androidMain/kotlin` | JVM 6 | `.jar` file or `.class` files |
| iOS | `SharedCode/iosMain` | iOS arm64 or x86_64| Apple framework |

Now it is time to refresh the Gradle project again in Android Studio. Click *Sync Now* on the yellow stripe 
or use the *Gradle* tool window and click the `Refresh` action in the context menu on the root Gradle project.
The `:SharedCode` project should be recognized by the IDE now.

We are ready to use the `SharedCode` library from our Android and iOS applications.

# Using SharedCode from Android

For this tutorial, we want to minimize Android project changes, so we add an ordinary dependency from that 
project to the `SharedCode` project.
It is also possible to use the `kotlin-multiplatform` plugin directly in an Android 
Gradle project, instead of the `kotlin-android` plugin. For more information, please refer to the
[Multiplatform Projects](/docs/reference/multiplatform.html) documentation.  

Let's include the dependency from the `SharedCode` project to the Android project. We need to patch
the `app/build.gradle` file and include the following line into the `dependencies { .. }` block:

<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
    implementation project(':SharedCode')
```
</div>

We need to
assign the `id` to the `TextView` control of our activity to access it from the code.
Let's patch the
`app/src/main/res/layout/activity_main.xml` file
(the name may be different if we changed it in the new project wizard)
and add several more attributes to the `<TextView>` element: 
```
        android:id="@+id/main_text"
        android:textSize="42sp"
        android:layout_margin="5sp"
        android:textAlignment="center"
```

Next, let's include the following line of code into the `MainActivity` class
from the `/app/src/main/java/<package>/MainActivity.kt` file, to 
the end of the `onCreate` method:

```
findViewById<TextView>(R.id.main_text).text = createApplicationScreenMessage()
```

Use the intention from the IDE to include the missing import line:
```kotlin
import org.kotlin.mpp.mobile.createApplicationScreenMessage
```
into the same file. 

Now we have the `TextView` that will show us the text created by the shared
code function `createApplicationScreenMessage()`. It shows `Kotlin Rocks on Android`.
Let's see how it works. 

## Running the Android Application

Let's click on the `App` run configuration
to get our project running either on a real Android Device or on the emulator. 

![Start the Application]({{ url_for('tutorial_img', filename='native/mpp-ios-android/studio-start-app.png') }})

And so now we can see the Application running in the Android emulator:
    
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

## Calling Kotlin Code from Swift

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
