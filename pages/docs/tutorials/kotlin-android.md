---
type: tutorial
layout: tutorial
title: "Android 与 Kotlin 入门"
description: "本教程将引导我们为 Android 创建一个简单的 Kotlin 应用程序。"
authors: 
showAuthorInfo: false
---

使用 Kotlin 进行 Android 开发非常简单。

在本教程中，我们将按照 Android Studio 的熟悉流程。如果你使用 Intellij IDEA 进行 Android 开发，这个过程几乎是一样的。

### 创建一个工程

首先，为你的应用创建一个新的 Kotlin Android 工程。

1. 打开 Android Studio，在欢迎页面点击 **Start a new Android Studio project**  或者 **File \| New \| New project**.

2. 选择一个定义应用程序行为的 [activity](https://developer.android.com/guide/components/activities/intro-activities) 。对于第一个 "Hello world" 应用程序，

   选择仅显示空白屏幕的 __Empty Activity__，然后点击 __Next__。

   ![Choosing empty activity]({{ url_for('tutorial_img', filename='kotlin-android/0-create-new-project.png') }})

3. 在下一个对话框中，填写工程的详细信息：

   - 名字和包名
   - 位置
   - 开发语音：选择 __Kotlin__

   保留其他选项的默认值，然后单击 __Finish__。

   ![Project configuration]({{ url_for('tutorial_img', filename='kotlin-android/1-create-new-project.png') }})

完成这些步骤后，Android Studio 会创建一个项目。 该项目已包含用于构建可在 Android 设备或模拟器上运<!---->行的应用程序的所有代码和资源。

### 构建和运行应用程序

在 Android Studio 中构建和运行 Kotlin 应用程序的过程与 Java 完全相同。

如需在模拟器上构建和运行应用程序：
1. 点击工具栏上的 __Run__ 运行预定义的 __app__ 配置，或者 __Run \| Run 'app'__。

2. 选择 __Create New Virtual Device__。

   ![Select target]({{ url_for('tutorial_img', filename='kotlin-android/select-target.png') }})

3. 选择你喜欢的设备，然后点击 __Next__.

   ![Select device]({{ url_for('tutorial_img', filename='kotlin-android/select-device.png') }})

4. 选择系统版本并下载其映像。

   ![System image]({{ url_for('tutorial_img', filename='kotlin-android/system-image.png') }})

5. 验证模拟器配置，然后点击 __Finish__。

6. 点击 __OK__，这就是—— 你的第一个 Android 版 Kotlin 应用程序！


<div style="display: flex; align-items: center; margin-bottom: 10px;">
    <img
    src="{{ url_for('asset', path='images/tutorials/kotlin-android/hello-app.png') }}"
    data-gif-src="{{ url_for('asset', path='images/tutorials/kotlin-android/hello-app.gif') }}"
    class="gif-image">
</div>

Kotlin有着极小的运行时文件体积：整个库的大小约 {{ site.data.releases.latest.runtime_size }}（{{ site.data.releases.latest.version }} 版本）。这意味着 Kotlin 对 apk 文件大小影响微乎其微。

就对比 Kotlin 与 Java所编写的程序而言，Kotlin 编译器所生成的字节码看上去几乎毫无差异。

如果要自定义构建或运行配置，请参考 Android Studio [文档](https://developer.android.com/studio/run)。

### 后续？

* 更多内容请查阅 [Kotlin Android 扩展插件](android-plugin.html) 以及 [Android 框架使用注解处理](android-frameworks.html)。
* 想尝试 Kotlin的不同特性？试试 [Kotlin Koans](koans.html)。
* 【译者补充】查看 [Google 的 Kotlin 示例工程](https://developer.android.com/samples/index.html?language=kotlin)。
