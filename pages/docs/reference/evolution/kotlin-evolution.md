---
type: doc
layout: reference
category: "Compatibility"
title: "Kotlin 演进"
---


# Kotlin 演进


## 实用主义演进原则


                        语言的设计是石头铸造的（译注：原文为“cast in stone”，意为“最终定论，板上钉钉”，此处双关），
                        但这块石头相当柔软，
                        经过一番努力，我们可以后期重塑它。
                        
                        Kotlin 设计团队

Kotlin 旨在成为程序员的实用工具。在语言演进方面，它的实用主义本质遵循以下原则：



*   一直保持语言的现代性。
*   与用户保持持续的反馈循环。
*   使版本更新对用户来说是舒适的。

由于这是理解 Kotlin 如何向前发展的关键，我们来展开来看这些原则。

**保持语言现代性**。我们承认系统随着时间的推移积累了很多遗留问题。曾经是尖端技术的东西如今可能已经无可救药地过时了。我们必须改进语言，使其与用户需求保持一致、与用户期望保持同步。这不仅包括添加新特性，还包括逐步淘汰不再推荐用于生产的旧特性，并且完全成为历史特性。

**舒适的更新**。如果没有适度谨慎地进行不兼容的变更（例如从语言中删除内容）可能会导致从一个版本到下一个版本的痛苦迁移过程。我们会始终提前公布这类变更，将相应内容标记为已弃用并 _在变更发生之前_ 提供自动化的迁移工具。当语言发生变更之时，我们希望世界上绝大多数代码都已经更新，这样迁移到新版本就没有问题了。

**反馈循环**。 通过弃用周期需要付出很大的努力，因此我们希望最大限度地减少将来不兼容变更的数量。除了使用我们的最佳判断之外，我们相信在现实生活中试用是验证设计的最佳方法。在最终定论之前，我们希望已经实战测试过。这就是为什么我们利用每个机会在语言的生产版本中提供我们早期版设计，只是带有_实验性_ 状态。实验性特性并不稳定，可以随时更改，选择使用它们的用户明确表示已准备好了应对未来的迁移问题。这些用户提供了宝贵的反馈，而我们收集这些反馈来迭代设计并使其坚如磐石。


## 不兼容的变更

如果从一个版本更新到另一个版本时，一些以前工作的代码不再工作，那么它是语言中的 _不兼容的变更_ （有时称为“破坏性变更”）。在一些场景中“不再工作”的确切含义可能会有争议，但是它肯定包含以下内容：



*   之前编译运行正常的代码现在（编译或链接）失败并报错。这包括删除语言结构以及添加新的限制。
*   之前正常执行的代码现在抛异常了。

属于“灰色区域”的不太明显的情况包括以不同方式处理极端情况，抛出与以前不同类型的异常，仅通过反射可以观察到的行为更改，未记录/未定义的行为更改，重命名二进制文件等。有时这些更改非常重要，并且会极大地影响迁移体验，有时影响微不足道。

绝对不是不兼容的变更的一些示例包括



*   添加新的警告。
*   启用新的语言结构或放宽对现有语言结构的限制。
*   更改私有/内部 API 和其他实现细节。

保持语言现代化和舒适更新的原则表明，有时需要进行不兼容的更改，但应该详细介绍这些更改。我们的目标是使用户提前了解即将发生的更改，以便他们能够轻松地迁移代码。

理想情况下，应通过有问题的代码中报告的编译期警告（通常称为弃用警告）来声明每个不兼容的更改，并提供自动迁移辅助工具。 因此，理想的迁移工作流程如下：



*   更新到版本 A（宣布更改）
    *   查看有关即将发生的更改的警告
    *   借助工具迁移代码
*   更新到版本 B（发生更改）
    *   完全没有问题

实际上，在编译期无法准确检测到某些更改，因此不会报告任何警告，但是至少会通过版本 A 的发行说明通知用户版本 B 中即将进行的更改。


### 处理编译器错误

编译器是复杂的软件，尽管开发人员尽了最大努力，但它们仍然存在 bug。导致编译器自身编译失败、或报告虚假错误、或生成明显编译失败的代码的 bug，虽然很烦人并且常常令人尴尬，但它们很容易修复，因为这些修复不构成不兼容的变更。其他 bug 可能会导致编译器生成不会编译失败的错误代码，例如：遗漏了源代码中的一些错误，或者只是生成了错误的指令。这些 bug 的修复是技术上不兼容的更改（某些代码过去可以正常编译，但现在编译失败），但是我们倾向于尽快修复它们，以防止不良代码模式在用户代码中传播。我们认为，这符合“舒适更新”的原则，因为较少的用户有机会遇到此问题。当然，这仅适用于在发行版本中出现后不久发现的 bug。


## Decision Making

[JetBrains](https://jetbrains.com), the original creator of Kotlin, is driving its progress with the help of the community and in accord with the [Kotlin Foundation](/foundation/kotlin-foundation.html).

All changes to the Kotlin Programming Language are overseen by the [Lead Language Designer](/foundation/kotlin-foundation.html#lead-designer) (currently Andrey Breslav). The Lead Designer has the final say in all matters related to language evolution. Additionally, incompatible changes to fully stable components have to be approved to by the [Language Committee](/foundation/kotlin-foundation.html#language-committee) designated under the [Kotlin Foundation](/foundation/kotlin-foundation.html) (currently comprised of Jeffrey van Gogh, William R. Cook and Andrey Breslav).

The Language Committee makes final decisions on what incompatible changes will be made and what exact measures should be taken to make user updates comfortable. In doing so, it relies on a set of guidelines available [here](/foundation/language-committee-guidelines.html).


## Feature Releases and Incremental Releases

Stable releases with versions 1.2, 1.3, etc. are usually considered to be _feature releases_ bringing major changes in the language. Normally, we publish _incremental releases_, numbered 1.2.20, 1.2.30, etc, in between feature releases. 

Incremental releases bring updates in the tooling (often including features), performance improvements and bug fixes. We try to keep such versions compatible with each other, so changes to the compiler are mostly optimizations and warning additions/removals. Experimental features may, of course, be added, removed or changed at any time.

Feature releases often add new features and may remove or change previously deprecated ones. Feature graduation from experimental to stable also happens in feature releases.


### EAP Builds

Before releasing stable versions, we usually publish a number of preview builds dubbed EAP (for "Early Access Preview") that let us iterate faster and gather feedback from the community. EAPs of feature releases usually produce binaries that will be later rejected by the stable compiler to make sure that possible bugs in the binary format survive no longer than the preview period. Final Release Candidates normally do not bear this limitation.


### Experimental features

According to the Feedback Loop principle described above, we iterate on our designs in the open and release versions of the language where some features have the _experimental_ status and _are supposed to change_. Experimental features can be added, changed or removed at any point and without warning. We make sure that experimental features can't be used accidentally by an unsuspecting user. Such features usually require some sort of an explicit opt-in either in the code or in the project configuration.

Experimental features usually graduate to the stable status after some iterations.


### Status of different components

To check the stability status of different components of Kotlin (Kotlin/JVM, JS, Native, various libraries, etc), please consult [this link](components-stability.html).


## Libraries

A language is nothing without its ecosystem, so we pay extra attention to enabling smooth library evolution.

Ideally, a new version of a library can be used as a "drop-in replacement" for an older version. This means that upgrading a binary dependency should not break anything, even if the application is not recompiled (this is possible under dynamic linking). 

On the one hand, to achieve this, the compiler has to provide certain ABI stability guarantees under the constraints of separate compilation. This is why every change in the language is examined from the point of view of binary compatibility.

On the other hand, a lot depends on the library authors being careful about which changes are safe to make. Thus it's very important that library authors understand how source changes affect compatibility and follow certain best practices to keep both APIs and ABIs of their libraries stable. Here are some assumptions that we make when considering language changes from the library evolution standpoint:



*   Library code should always specify return types of public/protected functions and properties explicitly thus never relying on type inference for public API. Subtle changes in type inference may cause return types to change inadvertently, leading to binary compatibility issues.
*   Overloaded functions and properties provided by the same library should do essentially the same thing. Changes in type inference may result in more precise static types to be known at call sites causing changes in overload resolution.

Library authors can use the @Deprecated and @Experimental annotations to control the evolution of their API surface. Note that @Deprecated(level=HIDDEN) can be used to preserve binary compatibility even for declarations removed from the API.

Also, by convention, packages named "internal" are not considered public API. All API residing in packages named "experimental" is considered experimental and can change at any moment.

We evolve the Kotlin Standard Library (kotlin-stdlib) for stable platforms according to the principles stated above. Changes to the contracts for its API undergo the same procedures as changes in the language itself.


## Compiler Keys

Command line keys accepted by the compiler are also a kind of public API, and they are subject to the same considerations. Supported flags (those that don't have the "-X" or "-XX" prefix) can be added only in feature releases and should be properly deprecated before removing them. The "-X" and "-XX" flags are experimental and can be added and removed at any time.


## Compatibility Tools

As legacy features get removed and bugs fixed, the source language changes, and old code that has not been properly migrated may not compile any more. The normal deprecation cycle allows a comfortable period of time for migration, and even when it's over and the change ships in a stable version, there's still a way to compile unmigrated code. 


### Compatibility flags

We provide the -language-version and -api-version flags that make a new version emulate the behaviour of an old one, for compatibility purposes. Normally, at least one previous version is supported. This effectively leaves a time span of two full feature release cycles for migration (which usually amounts to about two years). Using an older kotlin-stdlib or kotlin-reflect with a newer compiler without specifying compatibility flags is not recommended, and the compiler will report a [warning](compatibility-modes.html) when this happens.

Actively maintained code bases can benefit from getting bug fixes ASAP, without waiting for a full deprecation cycle to complete. Currently such project can enable the -progressive flag and get such fixes enabled even in incremental releases.

All flags are available on the command line as well as [Gradle](../using-gradle.html#编译器选项) and [Maven](../using-maven.html#指定编译器选项).


### Evolving the binary format

Unlike sources that can be fixed by hand in the worst case, binaries are a lot harder to migrate, and this makes backwards compatibility very important in the case of binaries. Incompatible changes to binaries can make updates very uncomfortable and thus should be introduced with even more care than those in the source language syntax. 

For fully stable versions of the compiler the default binary compatibility protocol is the following:



*   All binaries are backwards compatible, i.e. a newer compiler can read older binaries (e.g. 1.3 understands 1.0 through 1.2),
*   Older compilers reject binaries that rely on new features (e.g. a 1.0 compiler rejects binaries that use coroutines).
*   Preferably (but we can't guarantee it), the binary format is mostly forwards compatible with the next feature release, but not later ones (in the cases when new features are not used, e.g. 1.3 can understand most binaries from 1.4, but not 1.5).

This protocol is designed for comfortable updates as no project can be blocked from updating its dependencies even if it's using a slightly outdated compiler.

Please note that not all target platforms have reached this level of stability (but Kotlin/JVM has).
