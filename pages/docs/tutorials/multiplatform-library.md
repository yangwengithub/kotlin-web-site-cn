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
We are starting with an implementation for the JVM. Let's create a file `Base64.kt` in `jvmMain/kotlin/jetbrains/base64` folder and provide a simple implementation, which delegates to `java.util.Base64`:

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

Pretty simple, isn't it? We have provided a platform-specific implementation, but used a straightforward delegation to an implementation someone else has written!


## JS

Our JS implementation will be very similar to the JVM one. We create a file `Base64.kt` in `jsMain/kotlin/jetbrains/base64` and provide an implementation 
which delegates to NodeJS `Buffer` API:

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

On the generic Native platform we don't have the luxury to use someone else's implementation, so we will have to write one ourselves. I won't explain the implementation details here,
but it's pretty straightforward and follows Base64 format description without any optimizations:

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
                // Fill the pad with '='
                repeat(padSize) { result.add(BASE64_PAD.toByte()) }
            }
    
            return result.toByteArray()
        }
    }
```

</div>

Now we have implementations on all the platforms and it is time to move to testing of our library.

# 测试

To make the library complete we should write some tests, but we have three independent implementations and it is a waste of time to write duplicate tests for each one.
The good thing about common code is that it can be covered with common tests, which later are compiled and executed on *every* platform.
All the bits for testing are already generated by the project Wizard.

Let's create the class `Base64Test` in `commonTest/kotlin/jetbrains/base64` folder and write the basic tests for Base64.

But as you remember, our API converts byte arrays to byte arrays in a different format and it is not easy to test byte arrays.
So before we start writing a test, let's add the method `encodeToString` with a default implementation to our `Base64Encoder` interface:

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

Notice that the implementation on *every* platform can encode byte arrays to a string. If we want we can provide a more efficient implementation for this method, 
for example, let's specialize it on the JVM:

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only="1" auto-indent="false">

```kotlin
object JvmBase64Encoder : Base64Encoder {
    override fun encode(src: ByteArray): ByteArray = Base64.getEncoder().encode(src)
    override fun encodeToString(src: ByteArray): String = Base64.getEncoder().encodeToString(src)
}
```

</div>

Default implementations with optional more specialized overrides is another bonus of the multiplatform library. Now, when we have a string-based API, we can cover it with basic tests:

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

Execute `./gradlew check` and you will see that the tests are run three times, on JVM, on JS, and on Native!

If we want, we can add tests to a specific platform, then it will be executed only as part of these platform tests.
For example, we can add UTF-16 tests on JVM. Just follow the same steps as before, but create file in `jvmTest/kotlin/jetbrains/base64`:
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
This test will be automatically executed on the JVM target in addition to the common part.

## Publishing library to Maven

Our first multiplatform library is almost ready. The last step is to publish it, so other projects can then depend on our library.
To make the publishing mechanism work, you should enable the experimental Gradle feature in `settings.gradle`:

<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
enableFeaturePreview('GRADLE_METADATA')
```

</div>


Now the classic `maven-publish` Gradle [plugin](https://docs.gradle.org/current/userguide/publishing_maven.html) can be used.
Don't forget to specify the group and version of your library along with the plugin in `build.gradle`:

<div class="sample" markdown="1" mode="groovy" theme="idea" data-highlight-only="1" auto-indent="false">

```groovy
apply plugin: 'maven-publish'
group 'org.jetbrains.base64'
version '1.0.0'
```

</div>

Now check it with the command `./gradlew publishToMavenLocal` and you should see a successful build. 
That's it, our library is now successfully published and any Kotlin project can depend on it, whether it is another common library, JVM, JS, or Native application.


# 总结

In this tutorial we have:
- Created a multiplatform library with platform-specific implementations.
- Provided default implementation for common part and specialized it on JVM.
- Written common tests which are executed on every platform.
- Published the final library to Maven repository.
