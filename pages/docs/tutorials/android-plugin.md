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

## View binding

### 背景

相信每一位安卓开发人员对 `findViewById()` 这个方法再熟悉不过了，毫无疑问，潜在的 bug 以及脏乱的代码令后续开发无从下手的。尽管存在一系列的开源库能够为这个问题带来解决方案，这些库需要为每个公开的 'view' 添加注解字段。

现在 Kotlin 安卓扩展插件能够提供与这些开源库功能相同的体验，不需要添加任何额外代码。

本质上，它允许代码以下面这种方式实现：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
// 从 "main" 源集中使用 R.layout.activity_main 
import kotlinx.android.synthetic.main.activity_main.*

class MyActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // 代替 findViewById<TextView>(R.id.textView)
        textView.setText("Hello, world!")
    }
}
```
</div>

`textView` 是对 `Activity` 的一项扩展属性，与在 `activity_main.xml` 中的声明具有同样类型 (它是一个 `TextView`)。


## 使用 Kotlin 安卓扩展

### 依赖配置

{{ site.text_using_gradle }}

安卓扩展是 IntelliJ IDEA 与 Android Studio 的 Kotlin 插件的组成之一，因此不需要再单独安装额外插件。

开发者仅需要在模块的 `build.gradle` 文件中启用 Gradle 安卓扩展插件即可：

<div class="sample" markdown="1" theme="idea" mode="groovy">

```groovy
apply plugin: 'kotlin-android-extensions'
```
</div>

### 导入合成的属性

仅需要一行即可非常方便导入指定布局文件中所有控件属性：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
import kotlinx.android.synthetic.main.＜布局＞.*
```
</div>

因此，如果布局文件名是 `activity_main.xml`，我们将导入 `kotlinx.android.synthetic.main.activity_main.*`。

若需要调用 `View` 的合成属性，同时还应该导入 `kotlinx.android.synthetic.main.activity_main.view.*`。

导入完成后即可调用在xml文件中以视图控件具名属性的对应扩展，比如下例:

<div class="sample" markdown="1" theme="idea" mode="xml">

```xml
<TextView
    android:id="@+id/hello"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"/>
```
</div>

将有一个名为 `hello` 的属性：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
activity.hello.text = "Hello World!"
```
</div>

### 视图缓存

调用 `findViewById()`可能会很慢，尤其是在视图层次结构庞大的情况下，因此 Android 扩展程序试图通过在容器中缓存视图来使 `findViewById()` 的调用次数达到最少。

默认情况下，Android 扩展给每个用 kotlin 编写的容器（[`Activity`](https://developer.android.com/reference/android/app/Activity.html)、[`Fragment`](https://developer.android.com/reference/android/support/v4/app/Fragment.html)、[`View`](https://developer.android.com/reference/android/view/View.html) 或者 `LayoutContainer` 的实现）添加了一个隐藏的缓存函数以及一个存储字段。该方法很小，因此不会增加APK的大小。

在下面的示例中，`findViewById()` 仅被调用一次：

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

我们不知道这个函数会仅在用 kotlin 编写的 activity 中被调用还是会在普通的 Java activity 中被调用。 因此，即使将上一个示例中的 `MyActivity` 实例作为接收者进行传递，我们也不会在此处使用缓存。


#### 更改视图缓存策略

你可以全局或按容器更改缓存策略。这也需要打开[实验模式](#启用实验特性)。

项目全局缓存策略在 `build.gradle` 文件中设置：

<div class="sample" markdown="1" theme="idea" mode="groovy">

```groovy
androidExtensions {
    defaultCacheImplementation = "HASH_MAP" // 也可以是 SPARSE_ARRAY、NONE
}
```
</div>

默认情况下，Android扩展插件使用 `HashMap` 作为幕后存储集合，但是你可以切换到 `SparseArray`，也可以关闭缓存。当仅使用 Android 扩展的[Parcelable](#parcelable-实现生成器)部分时，关闭缓存特别有用。

另外，你可以通过给一个容器添加注解 `@ContainerOptions` 来更改它的缓存策略：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

``` kotlin
import kotlinx.android.extensions.ContainerOptions

@ContainerOptions(cache = CacheImplementation.NO_CACHE)
class MyActivity : Activity()

fun MyActivity.a() { 
    // findViewById() 会被调用两次
    textView.text = "Hidden view"
    textView.visibility = View.INVISIBLE
}
```
</div>

### `Parcelable` 实现生成器

Android 扩展插件提供 [`Parcelable`](https://developer.android.com/reference/android/os/Parcelable) 实现生成器。

#### 如何使用

给该类添加 `@Parcelize` 注解，会自动生成一个 `Parcelable` 的实现。

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
import kotlinx.android.parcel.Parcelize

@Parcelize
class User(val firstName: String, val lastName: String, val age: Int): Parcelable
```
</div>

`@Parcelize` 要求在主构造函数中声明所有序列化的属性。 Android 扩展会针对每个属性发出警告，并在类主体中声明一个支持字段。另外，如果某些主构造函数参数不是属性，那么无法使用 `@Parcelize`。

如果你的类需要更高级的序列化逻辑，可以在伴生类中实现：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
@Parcelize
data class User(val firstName: String, val lastName: String, val age: Int) : Parcelable {
    private companion object : Parceler<User> {
        override fun User.write(parcel: Parcel, flags: Int) {
            // 自定义写实现
        }

        override fun create(parcel: Parcel): User {
            // 自定义读实现
        }
    }
}
```
</div>


#### 支持的类型

`@Parcelize` 支持多种类型：

- 基本类型（及其装箱版本）；
- objects 与 enums；
- `String`、`CharSequence`；
- `Exception`；
- `Size`、`SizeF`、`Bundle`、`IBinder`、`IInterface`、`FileDescriptor`；
- `SparseArray`、`SparseIntArray`、`SparseLongArray`、`SparseBooleanArray`；
- 所有 `Serializable` (包括 `Date`) 以及 `Parcelable` 的实现；
  - 所有受支持类型的集合：`List`（映射到 `ArrayList`）、`Set`（映射到 `LinkedHashSet`）、`Map`（映射到 `LinkedHashMap`）；
      + 还有一些具体的实现：`ArrayList`、`LinkedList`、`SortedSet`、`NavigableSet`、`HashSet`、`LinkedHashSet`、`TreeSet`、`SortedMap`、`NavigableMap`、`HashMap`、`LinkedHashMap`、`TreeMap`、`ConcurrentHashMap`；
- 所有受支持类型的数组；
- 所有受支持类型的可空版本。


#### 自定义 `Parcelers`

即使不直接支持的类型，你也可以为其编写 `Parceler` 映射对象。

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

可以通过注解 `@TypeParceler` 或者 `@WriteWith` 应用外部的 parcelers：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
// 局部类的 parceler
@Parcelize
@TypeParceler<ExternalClass, ExternalClassParceler>()
class MyClass(val external: ExternalClass)

// 局部属性的 parceler
@Parcelize
class MyClass(@TypeParceler<ExternalClass, ExternalClassParceler>() val external: ExternalClass)

// 局部类型的 parceler
@Parcelize
class MyClass(val external: @WriteWith<ExternalClassParceler>() ExternalClass)
```
</div>

{:#enabling-experimental-features}

### 实验性特性

Android 扩展插件包括几个实验特性：

- [LayoutContainer 支持](#layoutcontainer-支持)
- [多渠道支持](#多渠道支持)

这些特性尚未被考虑用于生产环境，因此你需要在 `build.gradle` 中打开 *实验模式* 才能使用它们：

<div class="sample" markdown="1" theme="idea" mode="groovy">

```groovy
androidExtensions {
    experimental = true
}
```
</div>

### LayoutContainer 支持

Android 扩展插件支持不同类型的容器。最基本的是[`Activity`](https://developer.android.com/reference/android/app/Activity.html)、[`Fragment`](https://developer.android.com/reference/android/support/v4/app/Fragment.html) 以及 [`View`](https://developer.android.com/reference/android/view/View.html)，但是你可以（实际上）通过实现 `LayoutContainer` 接口将任何类转换为 Android 扩展容器，例如：

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
import android.support.v7.widget.RecyclerView
import kotlinx.android.extensions.LayoutContainer

class ViewHolder(override val containerView: View) : RecyclerView.ViewHolder(containerView), LayoutContainer {
    fun setup(title: String) {
        itemTitle.text = "Hello World!"
    }
}
```
</div>

请注意，你需要打开[实验性标志](#启用实验特性)才能使用 `LayoutContainer`。


### 多渠道支持

安卓扩展插件现已支持安卓多渠道。假设当前在 `build.gradle` 文件中指定一个名为 `free` 的渠道：

<div class="sample" markdown="1" theme="idea" mode="groovy">

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

在[实验模式](#启用实验特性)中，你可以指定任何变体名称（不仅是渠道），例如 `freeDebug` 或者 `freeRelease` 也可以使用。



