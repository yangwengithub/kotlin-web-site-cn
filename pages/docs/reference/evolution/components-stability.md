---
type: doc
layout: reference
category: "Compatibility"
title: "不同组件的稳定性"
---

# 不同组件的稳定性

依据组件的发展速度，可以有不同的稳定性模式：
<a name="moving-fast"></a>
*   **快速流转（MF，Moving fast）**：即使在[增量版本](kotlin-evolution.html#feature-releases-and-incremental-releases)之间也不要期待任何兼容性，任何功能都可以在没有警告的情况下添加、删除或者更改。

*   **有功能添加的增量版本（AIR，Additions in Incremental Releases）**：可以在增量版本中添加内容，应避免删除与更改行为，而如果必须要删改的话，应在之前的增量版本中预告。

*   **稳定增量版本（SIR，Stable Incremental Releases）**：增量版本完全兼容，只会有优化与 bug 修复。可以在[特性版本](kotlin-evolution.html#feature-releases-and-incremental-releases)中进行任何更改。

<a name="fully-stable"></a>
*   **完全稳定（FS，Fully Stable）**：增量版本完全兼容，特性版本兼容旧版。

对于相同的组件，源代码兼容性与二进制兼容性可以有不同的模式，例如，在二进制格式稳定之前，源代码语言可以达到完全稳定，反之亦然。

[Kotlin 演进制度](kotlin-evolution.html)的条款只适用于已经达到完全稳定（FS）的组件。从那一刻起，不兼容的变更必须得到语言委员会的批注。

|**组件**|**进入该状态的版本**|**源代码兼容模式**|**二进制兼容模式**|
| --- | --- | --- | --- |
| Kotlin/JVM|1.0|FS|FS|
| kotlin-stdlib（JVM）|1.0|FS|FS
| KDoc 语法|1.0|FS|N/A
| 协程|1.3|FS|FS
| kotlin-reflect（JVM）|1.0|SIR|SIR
| Kotlin/JS|1.1|AIR|MF
| Kotlin/Native|1.3|AIR|MF
| Kotlin 脚本（*.kts）|1.2|AIR|MF
| dokka|0.1|MF|N/A
| Kotlin 脚本 API|1.2|MF|MF
| 编译器插件 API|1.0|MF|MF
| 序列化|1.3|MF|MF
| 多平台项目|1.2|MF|MF
| 内联类|1.3|MF|MF
| 无符号算术|1.3|MF|MF
| **默认情况下，所有其他实验性特性**|N/A|**MF**|**MF**