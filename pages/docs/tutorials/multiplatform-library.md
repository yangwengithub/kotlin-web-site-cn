---
type: tutorial
layout: tutorial
title:  "Kotlin 多平台库"
description: "在 JVM、JS 以及 Native 的世界中共享 Kotlin 库"
authors: Vsevolod Tolstopyatov，乔禹昂（翻译）
date: 2018-10-04
showAuthorInfo: true
issue: EVAN-6031
---

在本教程中，我们将创建一个在 JVM、JS 以及 Native 世界中可用的库。 
您将逐步了解如何创建可从任何其他公共代码使用的多平台库（例如，在 Android 与 iOS 之间共享），
以及如何编写可以在所有平台上执行的测试，并使用具体平台提供的有效实现。

# 我们将创建什么？

我们的目标是构建一个小型多平台库，以展示在平台之间共享代码的能力以及其优势。
为了有一个小的实现来关注多平台机制，我们将编写一个库<!--
-->将原始数据（字符串和字节数组）转换为 [Base64](https://en.wikipedia.org/wiki/Base64) 格式，该格式可用于JVM、JS和任何可用的 K/N 平台。
在 JVM 上的实现我们将使用 [`java.util.Base64`](https://docs.oracle.com/javase/8/docs/api/java/util/Base64.html)，众所周知这是非常有效的，
因为 JVM 可以识别这个特定的类并以特殊的方式编译它。
在 JS 上我们将使用原生的 [Buffer](https://nodejs.org/docs/latest/api/buffer.html) API 而在 Kotlin/Native 我们将编写自己的实现。
我们将使用常见测试来介绍此功能，然后将生成的库发布到Maven。


# 配置本地环境

在本教程中我们将使用 IntelliJ IDEA 社区版，而使用终极版也同样可以做到。IDE 应该已经安装了 Kotlin plugin 1.3.x 或者更高的版本。 
这个可以通过 IDE 的 *Settings*（或 *Preferences*）中的 *Language & Frameworks | Kotlin Updates* 部分来验证。
此项目的原生部分是使用 Mac OS X 编写的，但如果您使用的是其他平台，请不要担心，该平台仅影响此特定教程中的目录名称。

# 创建一个工程

我们将使用 IntelliJ IDEA 社区版来演示。你需要确保你已经安装了最新版本的 Kotlin plugin，1.3.x 或者更新。
我们选择 *File | New | Project*，选择 *Kotlin | Kotlin (Multiplatform Library)* 并以我们想要的方式配置工程。

![Wizard]({{ url_for('tutorial_img', filename='multiplatform/wizard.png') }})

现在创建了一个多平台样本库并将其导入 IntelliJ IDEA。让我们将所有的 `.kt` 使用 IntelliJ IDEA 的功能 *Refactor | Rename* 将它们的包名修改为 `org.jetbrains.base64`，
让我们到目前为止检查项目的一切是否正确，项目结构应该是：

```
└── src
    ├── commonMain
    │   └── kotlin
    ├── commonTest
    │   └── kotlin
    ├── jsMain
    │   └── kotlin
    ├── jsTest
    │   └── kotlin
    ├── jvmMain
    │   └── kotlin
    ├── jvmTest
    │   └── kotlin
    ├── macosMain
    │   └── kotlin
    └── macosTest
        └── kotlin
```

`kotlin` 文件夹应包含 `org.jetbrains.base64` 子文件夹。

# 通用部分

现在我们需要定义我们想要实现的类以及接口。在 `commonMain/kotlin/jetbrains/base64` 文件夹下创建文件 `Base64.kt`。
核心原语将是 `Base64Encoder` 接口，它知道如何将字节转换为 `Base64` 格式的字节：

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
interface Base64Encoder {
    fun encode(src: ByteArray): ByteArray
}
```

</div>

但是公共代码应该以某种方式获取此接口的实例，为此我们定义工厂对象 `Base64Factory`：

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
expect object Base64Factory {
    fun createEncoder(): Base64Encoder
}
```

</div>

我们的工厂函数被使用 `expect` 关键字标记。`expect` 是一种定义需求的机制，每个平台都应提供这种机制，以使公共部分正常工作。
所以在每个平台上我们都应该提供 `actual` `Base64Factory`，它知道如何创建特定于平台的编码器。
你可以在[这里](/docs/reference/platform-specific-declarations.html)阅读更多关于平台特定的声明。


# 平台指定实现

现在是时候为每个平台提供 `Base64Factory` 的 `actual` 实现了。


## JVM
我们开始编写 JVM 上的实现。让我们在 `jvmMain/kotlin/jetbrains/base64` 文件下创建一个文件——`Base64.kt`，委托给 `java.util.Base64` 来提供一个简单的实现：

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
actual object Base64Factory {
    actual fun createEncoder(): Base64Encoder = JvmBase64Encoder
}

object JvmBase64Encoder : Base64Encoder {
    override fun encode(src: ByteArray): ByteArray = Base64.getEncoder().encode(src)
}

```

</div>

非常简单不是吗？我们已经提供了一个平台指定的实现，但是使用了一个简单的委托给其他人编写的实现！


## JS

我们在 JS 上的实现将与 JVM 上的非常相似。我们在 `jsMain/kotlin/jetbrains/base64` 下创建了一个文件 `Base64.kt`，并将实现委托给了
NodeJS 的 `Buffer` API：

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
actual object Base64Factory {
    actual fun createEncoder(): Base64Encoder = JsBase64Encoder
}

object JsBase64Encoder : Base64Encoder {
    override fun encode(src: ByteArray): ByteArray {
        val buffer = js("Buffer").from(src)
        val string = buffer.toString("base64") as String
        return ByteArray(string.length) { string[it].toByte() }
    }
}
```

</div>

## Native

在通用的 Native 平台上，我们没有使用别人的实现，所以我们必须自己写一个。 我不会在这里解释实现细节，
但它非常简单，遵循 Base64 格式描述，且没有任何优化：

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
private val BASE64_ALPHABET: String = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"
private val BASE64_MASK: Byte = 0x3f
private val BASE64_PAD: Char = '='
private val BASE64_INVERSE_ALPHABET = IntArray(256) {
    BASE64_ALPHABET.indexOf(it.toChar())
}

private fun Int.toBase64(): Char = BASE64_ALPHABET[this]

actual object Base64Factory {
    actual fun createEncoder(): Base64Encoder = NativeBase64Encoder
}

object NativeBase64Encoder : Base64Encoder {
    override fun encode(src: ByteArray): ByteArray {
            fun ByteArray.getOrZero(index: Int): Int = if (index >= size) 0 else get(index).toInt()
            // 4n / 3 is expected Base64 payload
            val result = ArrayList<Byte>(4 * src.size / 3) 
            var index = 0
            while (index < src.size) {
                val symbolsLeft = src.size - index
                val padSize = if (symbolsLeft >= 3) 0 else (3 - symbolsLeft) * 8 / 6
                val chunk = (src.getOrZero(index) shl 16) or (src.getOrZero(index + 1) shl 8) or src.getOrZero(index + 2)
                index += 3
        
                for (i in 3 downTo padSize) {
                val char = (chunk shr (6 * i)) and BASE64_MASK.toInt()
                    result.add(char.toBase64().toByte())
                }
                // 使用 '=' 填充
                repeat(padSize) { result.add(BASE64_PAD.toByte()) }
            }
    
            return result.toByteArray()
        }
    }
```

</div>

现在我们在所有平台上都有实现，是时候开始测试我们的库了。

# 测试

为了使库完整，我们应该编写一些测试用例，但是我们有三个独立的实现，为每个测试编写重复测试用例是浪费时间的。
通用代码的好处在于它可以通过通用的测试来覆盖，后来在*每个*平台上编译和执行。
所有用于测试的位都已由工程向导生成。

让我们在 `commonTest/kotlin/jetbrains/base64` 文件夹下创建类 `Base64Test`，并为 Base64 编写一个基本的测试。

但是，您还记得，我们的 API 将字节数组转换为不同格式的字节数组，并且测试字节数组并不容易。
所以在我们编写测试代码之前，让我们为 `Base64Encoder` 接口的 `encodeToString`添加一个默认实现：

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
interface Base64Encoder {
    fun encode(src: ByteArray): ByteArray

    fun encodeToString(src: ByteArray): String {
        val encoded = encode(src)
        return buildString(encoded.size) {
            encoded.forEach { append(it.toChar()) }
        }
    }
}
```

</div>

注意，在*每个*平台上的实现都可以将字节数组编码成字符串。如果我们想要，我们可以为此方法提供更有效的实现，
比如说，在 JVM 上：

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
object JvmBase64Encoder : Base64Encoder {
    override fun encode(src: ByteArray): ByteArray = Base64.getEncoder().encode(src)
    override fun encodeToString(src: ByteArray): String = Base64.getEncoder().encodeToString(src)
}
```

</div>

具有可选的更多指定覆盖的默认实现是多平台库的另一个优点。现在，当我们有一个基于字符串的API时，我们可以用基本测试来覆盖它：

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
class Base64Test {
    @Test
    fun testEncodeToString() {
        checkEncodeToString("Kotlin is awesome", "S290bGluIGlzIGF3ZXNvbWU=")
    }

    @Test
    fun testPaddedStrings() {
        checkEncodeToString("", "")
        checkEncodeToString("1", "MQ==")
        checkEncodeToString("22", "MjI=")
        checkEncodeToString("333", "MzMz")
        checkEncodeToString("4444", "NDQ0NA==")
    }

    private fun checkEncodeToString(input: String, expectedOutput: String) {
        assertEquals(expectedOutput, Base64Factory.createEncoder().encodeToString(input.asciiToByteArray()))
    }

    private fun String.asciiToByteArray() = ByteArray(length) {
        get(it).toByte()
    }
}
```

</div>

执行 `./gradlew check` 并且你讲看到测试代码在 JVM、JS 以及 Native 平台上共运行了三次！

如果我们想要，我们可以将测试代码添加到指定平台，接下来只执行这个平台部分的测试。
例如，我们可以在 JVM 上添加 UTF-16 测试。只需按照之前的步骤操作，但是要在 `jvmTest/kotlin/jetbrains/base64` 中创建文件：
<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
class Base64JvmTest {
    @Test
    fun testNonAsciiString() {
        val utf8String = "Gödel"
        val actual = Base64Factory.createEncoder().encodeToString(utf8String.toByteArray())
        assertEquals("R8O2ZGVs", actual)
    }
}
```

</div>
除了公共部分之外，此测试还将在 JVM 目标平台上自动执行。

## 将库发布到 Maven

我们的第一个多平台库几乎已经准备就绪。最后一步是发布它，因此其他项目可以依赖于我们的库。
要使发布机制起作用，您应该在 `settings.gradle` 中启用实验性的 Gradle 功能：

<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
enableFeaturePreview('GRADLE_METADATA')
```

</div>


现在经典的 `maven-publish` Gradle [插件](https://docs.gradle.org/current/userguide/publishing_maven.html)可以使用了。
不要忘记在 `build.gradle` 中指定库的 group 与版本以及插件：

<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
apply plugin: 'maven-publish'
group 'org.jetbrains.base64'
version '1.0.0'
```

</div>

现在使用命令 `./gradlew publishToMavenLocal` 来检查它，你会看到构建已经成功。
至此，我们的库已经成功发布并且所有的 Kotlin 工程都可以依赖它，无论它是另外一个通用库还是 JVM、JS 或者是 Native 应用程序。


# 总结

在本篇教程中我们学到了：
- 创建一个多平台库，并且包含平台指定的实现。
- 提供了公共部分的默认实现，并在 JVM 上使用它。
- 编写一个能在各个平台上运行的通用测试用例。
- 发布最终版本的库到 Maven 仓库。
