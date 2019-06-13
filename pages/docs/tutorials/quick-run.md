---
type: tutorial
layout: tutorial
title: "运行代码片段"
description: "本教程展示了在不创建或修改整个项目的情况下以轻量级方式运行 Kotlin 代码片段的方法。"
authors: Pavel Semyonov johnpoint(翻译)
date: 2018-12-24
showAuthorInfo: false
related:
    - command-line.md
---

有时，您可能需要在项目或应用程序之外快速编写和执行一些代码。例如，在学习 Kotlin 或计算表达式时，这可能很有用。让我们看看快速运行Kotlin代码的两种简便方法:
* [代码草稿](#代码草稿) 让您在 IDE 中的项目外部的临时文件中编写和运行代码。
* [REPL](#repl) (_Read-Eval-Print-Loop_) 在控制台中以交互方式运行代码。     


## 代码草稿

> 目前，代码草稿仅支持 Kotlin/JVM 项目。
{:.note}

IntelliJ IDEA 的 Kotlin 插件支持 [代码草稿](https://www.jetbrains.com/help/idea/scratches.html)。 代码草稿允许您在项目的同一 IDE 窗口中创建代码并即时运行它们。草稿与项目无关；您可以从操作系统上的任何 IntelliJ IDEA 窗口访问并运行任意代码草稿。

要创建 Kotlin 草稿，请单击 __文件 \| 新的 \| 草稿文件__ 并选择 __Kotlin__ 类型。

在您的草稿中，您可以编写任何有效的 Kotlin 代码，包括新的函数和类。IntelliJ IDEA 支持草稿的语法高亮、自动补全和其他代码编辑功能。

开始编写代码，然后单击 __运行__ 。执行结果将出现在代码行的对面。

<div style="display: flex; align-items: center; margin-bottom: 10px;">
    <img
    src="{{ url_for('asset', path='images/tutorials/quick-run/scratch-run.png') }}"
    data-gif-src="{{ url_for('asset', path='images/tutorials/quick-run/scratch-run.gif') }}"
    class="gif-image">
</div>

### 交互模式

IntelliJ IDEA 可以自动运行您的草稿代码。要在短时间停止键入后获得执行结果，请打开 __交互模式__ 。

<div style="display: flex; align-items: center; margin-bottom: 10px;">
    <img
    src="{{ url_for('asset', path='images/tutorials/quick-run/scratch-interactive.png') }}"
    data-gif-src="{{ url_for('asset', path='images/tutorials/quick-run/scratch-interactive.gif') }}"
    class="gif-image">
</div>

### 调用模块

要在草稿中使用项目中的类或函数，请像往常一样使用 `import` 语句将它们导入到临时文件中。然后编写代码并在 __Use classpath of module__ 中选择的相应模块。要在运行草稿之前自动重建模块，请选择 __Make before Run__。

![Scratch select module]({{ url_for('tutorial_img', filename='quick-run/scratch-select-module.png') }})

### 使用 REPL 运行

若要计算草稿中的每个特定表达式，请在选择 __Use REPL__ 的情况下运行它。草稿的执行方式与 [REPL](#repl) 相同：代码行将随后运行，提供每次调用的结果。稍后，您可以通过相应行中显示的名称 `res*` 来引用结果。

![Scratch REPL]({{ url_for('tutorial_img', filename='quick-run/scratch-repl.png') }})

## REPL

_REPL_ (_读取-求值-输出-循环_) 是一个交互式运行 Kotlin 代码的工具。REPL 允许您运行表达式和代码块，而无需创建项目或函数(如果您不需要的话)。

如需在 IntelliJ IDEA 中运行 REPL，请打开 __工具 \| Kotlin \| Kotlin REPL__ 。

如需在操作系统命令行中运行REPL，请从 Kotlin 独立编译器的目录中打开 __/bin/Kotlic-JVM __ 。

打开 REPL 命令行界面后。您可以输入任何有效的 Kotlin 代码并查看运行结果。结果以变量的形式输出，自动生成的变量名称如 `res*`。您可以稍后在REPL 运行的代码中使用这些变量。

<div style="display: flex; align-items: center; margin-bottom: 10px;">
    <img
    src="{{ url_for('asset', path='images/tutorials/quick-run/repl-run.png') }}"
    data-gif-src="{{ url_for('asset', path='images/tutorials/quick-run/repl-run.gif') }}"
    class="gif-image">
</div>

REPL 也支持多行输入。多行输入的结果是其最后一个表达式的值。

<div style="display: flex; align-items: center; margin-bottom: 10px;">
    <img
    src="{{ url_for('asset', path='images/tutorials/quick-run/repl-multi-line.png') }}"
    data-gif-src="{{ url_for('asset', path='images/tutorials/quick-run/repl-multi-line.gif') }}"
    class="gif-image">
</div>
