---
type: tutorial
layout: tutorial
title:  "Kotlin Android 扩展"
description: "本教程介绍如何使用 Kotlin Android 扩展来改进对 Android 开发的支持。"
authors: Yan Zhulanow
showAuthorInfo: true
source:
---
在本章教程中，我们将逐步介绍如何使用 Kotlin 安卓扩展插件提升安卓的开发体验。

## View Binding

### 背景

相信每一位安卓开发人员对 `findViewById()` 这个方法再熟悉不过了，毫无疑问，潜在的 bug 和脏乱的代码令后续开发无从下手的。尽管存在一系列的开源库能够为这个问题带来解决方案，those libraries require annotating fields for each exposed `View`.

现在 Kotlin 安卓扩展插件能够提供与这些开源库功能相同的体验，不需要添加任何额外代码。

In essence, this allows for the following code:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
// Using R.layout.activity_main from the 'main' source set
import kotlinx.android.synthetic.main.activity_main.*

class MyActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // Instead of findViewById<TextView>(R.id.textView)
        textView.setText("Hello, world!")
    }
}
```
</div>

`textView` 是对 `Activity` 的一项扩展属性，与在 `activity_main.xml` 中的声明具有同样类型 (so it is a `TextView`)。


### 使用 Kotlin 安卓扩展

#### 依赖配置

{{ site.text_using_gradle }}

安卓扩展是 IntelliJ IDEA 与 Android Studio 的 Kotlin 插件的组成之一，因此不需要再单独安装额外插件。

开发者仅需要在模块的 `build.gradle` 文件中启用 Gradle 安卓扩展插件即可：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```groovy
apply plugin: 'kotlin-android-extensions'
```
</div>

#### 导入合成属性

仅需要一行即可非常方便导入指定布局文件中所有控件属性：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
import kotlinx.android.synthetic.main.＜布局＞.*
```
</div>

假设当前布局文件是 `activity_main.xml`，我们只需要引入 `kotlinx.android.synthetic.main.activity_main.*`。

若需要调用 `View` 的合成属性，同时还应该导入 `kotlinx.android.synthetic.main.activity_main.view.*`。

导入完成后即可调用在xml文件中以视图控件命名属性的对应扩展，比如下例:

```xml
<TextView
    android:id="@+id/hello"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"/>
```

将有一个名为 `hello` 的属性：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
activity.hello.text = "Hello World!"
```
</div>


### Experimental Mode

Android Extensions plugin includes several experimental features such as `LayoutContainer` support and a `Parcelable` implementation generator. These features are not considered production ready yet, so you need to turn on the experimental mode in `build.gradle` in order to use them:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```groovy
androidExtensions {
    experimental = true
}
```
</div>


### `LayoutContainer` Support

Android Extensions plugin supports different kinds of containers. The most basic ones are [`Activity`](https://developer.android.com/reference/android/app/Activity.html), [`Fragment`](https://developer.android.com/reference/android/support/v4/app/Fragment.html) and [`View`](https://developer.android.com/reference/android/view/View.html), but you can turn (virtually) any class to an Android Extensions container by implementing the `LayoutContainer` interface, e.g.:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
import kotlinx.android.extensions.LayoutContainer

class ViewHolder(override val containerView: View) : ViewHolder(containerView), LayoutContainer {
    fun setup(title: String) {
        itemTitle.text = "Hello World!"
    }
}
```
</div>

Note that you need to turn on the [experimental flag](#experimental-mode) to use `LayoutContainer`.


### 多渠道支持

安卓扩展插件现已支持安卓多渠道。假设当前在 `build.gradle` 文件中指定一个名为 `free` 的渠道：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```groovy
android {
    productFlavors {
        free {
            versionName "1.0-free"
        }
    }
}
```
</div>

所以现在只需要添加一行导入语句即可从 `free/res/layout/activity_free.xml` 布局中导入所有的合成属性：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
import kotlinx.android.synthetic.free.activity_free.*
```
</div>

In the [experimental mode](#experimental-mode), you can specify any variant name (not only flavor), e.g. `freeDebug` or `freeRelease` will work as well.


### View Caching

Invoking `findViewById()` can be slow, especially in case of huge view hierarchies, so Android Extensions tries to minimize `findViewById()` calls by caching views in containers.

By default, Android Extensions adds a hidden cache function and a storage field to each container ([`Activity`](https://developer.android.com/reference/android/app/Activity.html), [`Fragment`](https://developer.android.com/reference/android/support/v4/app/Fragment.html), [`View`](https://developer.android.com/reference/android/view/View.html) or a `LayoutContainer` implementation) written in Kotlin. The method is pretty small so it does not increase the size of APK much.

In the following example, `findViewById()` is only invoked once:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
class MyActivity : Activity()

fun MyActivity.a() { 
    textView.text = "Hidden view"
    textView.visibility = View.INVISIBLE
}
```
</div>

然而在下面的例子中：

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
fun Activity.b() { 
    textView.text = "Hidden view"
    textView.visibility = View.INVISIBLE
}
```
</div>

We wouldn't know if this function would be invoked on only activities from our sources or on plain Java activities also. Because of this, we don’t use caching there, even if `MyActivity` instance from the previous example is passed as a receiver.


### Changing View Caching Strategy

You can change the caching strategy globally or per container. This also requires switching on the [experimental mode](#experimental-mode).

Project-global caching strategy is set in the `build.gradle` file:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```groovy
androidExtensions {
    defaultCacheImplementation = "HASH_MAP" // also SPARSE_ARRAY, NONE
}
```
</div>

By default, Android Extensions plugin uses `HashMap` as a backing storage, but you can switch to the `SparseArray` implementation, or just switch off caching. The latter is especially useful when you use only the [Parcelable](#parcelable) part of Android Extensions.

Also, you can annotate a container with `@ContainerOptions` to change its caching strategy:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
import kotlinx.android.extensions.ContainerOptions

@ContainerOptions(cache = CacheImplementation.NO_CACHE)
class MyActivity : Activity()

fun MyActivity.a() { 
    // findViewById() will be called twice
    textView.text = "Hidden view"
    textView.visibility = View.INVISIBLE
}
```
</div>

## Parcelable

Starting from Kotlin 1.1.4, Android Extensions plugin provides Parcelable implementation generator as an experimental feature.

### Enabling Parcelable support

Apply the `kotlin-android-extensions` Gradle plugin as described [above](#依赖配置) and [turn on](#experimental-mode) the experimental flag.

### How to use

Annotate the class with `@Parcelize`, and a `Parcelable` implementation will be generated automatically.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
import kotlinx.android.parcel.Parcelize

@Parcelize
class User(val firstName: String, val lastName: String, val age: Int): Parcelable
```
</div>

`@Parcelize` requires all serialized properties to be declared in the primary constructor. Android Extensions will issue a warning on each property with a backing field declared in the class body. Also, `@Parcelize` can't be applied if some of the primary constructor parameters are not properties.

If your class requires more advanced serialization logic, you can write it inside a companion class:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
@Parcelize
data class User(val firstName: String, val lastName: String, val age: Int) : Parcelable {
    private companion object : Parceler<User> {
        override fun User.write(parcel: Parcel, flags: Int) {
            // Custom write implementation
        }

        override fun create(parcel: Parcel): User {
            // Custom read implementation
        }
    }
}
```
</div>


### Supported Types

`@Parcelize` supports a wide range of types:

- Primitive types (and its boxed versions);
- Objects and enums;
- `String`, `CharSequence`;
- `Exception`;
- `Size`, `SizeF`, `Bundle`, `IBinder`, `IInterface`, `FileDescriptor`;
- `SparseArray`, `SparseIntArray`, `SparseLongArray`, `SparseBooleanArray`;
- All `Serializable` (yes, `Date` is supported too) and `Parcelable` implementations;
- Collections of all supported types: `List` (mapped to `ArrayList`), `Set` (mapped to `LinkedHashSet`), `Map` (mapped to `LinkedHashMap`);
    + Also a number of concrete implementations: `ArrayList`, `LinkedList`, `SortedSet`, `NavigableSet`, `HashSet`, `LinkedHashSet`, `TreeSet`, `SortedMap`, `NavigableMap`, `HashMap`, `LinkedHashMap`, `TreeMap`, `ConcurrentHashMap`;
- Arrays of all supported types;
- Nullable versions of all supported types.


### Custom `Parceler`s

Even if your type is not supported directly, you can write a `Parceler` mapping object for it.

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
class ExternalClass(val value: Int)

object ExternalClassParceler : Parceler<ExternalClass> {
    override fun create(parcel: Parcel) = ExternalClass(parcel.readInt())

    override fun ExternalClass.write(parcel: Parcel, flags: Int) {
        parcel.writeInt(value)
    }
}
```
</div>

External parcelers can be applied using `@TypeParceler` or `@WriteWith` annotations:

<div class="sample" markdown="1" theme="idea" data-highlight-only>
```kotlin
// Class-local parceler
@Parcelize
@TypeParceler<ExternalClass, ExternalClassParceler>()
class MyClass(val external: ExternalClass)

// Property-local parceler
@Parcelize
class MyClass(@TypeParceler<ExternalClass, ExternalClassParceler>() val external: ExternalClass)

// Type-local parceler
@Parcelize
class MyClass(val external: @WriteWith<ExternalClassParceler>() ExternalClass)
```
</div>
