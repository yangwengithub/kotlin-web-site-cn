---
type: tutorial
layout: tutorial
title: "运行代码片段"
description: "本教程介绍了以轻量级方式编写和运行 Kotlin 代码段、
而无需创建整个应用程序的方法。"
authors: Pavel Semyonov，johnpoint(翻译)，Yue_plus(翻译)
date: 2019-11-13
showAuthorInfo: false
related:
    - command-line.md
---

有时可能需要在项目或应用程序之外快速编写并执行代码。
例如，在学习 Kotlin 或计算表达式时，这可能会很有用。
来看一下可以用来快速运行 Kotlin 代码的三种便捷方式：
* [草稿](#草稿与工单)：在 IDE 中的项目之外的临时文件中编写和运行代码。
* [工单](#草稿与工单)：就像草稿一样，但是它们位于项目中。
* [REPL](#repl)：(_<span title="Read">读取</span>-<span title="Eval">求值</span>-<span title="Print">输出</span>-<span title="Loop">循环</span>_)在交互式控制台中运行代码。

## 草稿与工单

IntelliJ IDEA 的 Kotlin 插件支持[_草稿_](https://www.jetbrains.com/help/idea/scratches.html)与 _工单_。
 
使用草稿，可以在与项目相同的 IDE 窗口中创建代码草稿，并即时运行它们。
草稿与项目无关；所以可以从系统上的任何 IntelliJ IDEA 窗口访问并运行所有草稿。

要创建 Kotlin 草稿，请单击 __File \| New \| Scratch file__ 并选择 __Kotlin__ 类型。

相反，工单是项目文件：它们存储在项目目录中并与项目模块绑定。
工单对于编写实际上不是软件模块但仍应一起存储在项目中的代码部分很有用。
例如，可以将工单用于教育或演示材料。

要在项目目录中创建 Kotlin 工单，请右键单击项目树中的目录并选择
__New \| Kotlin Worksheet__。

在草稿与工单中，可以编写任何有效的 Kotlin 代码。
语法高亮显示、自动完成与其他 IntelliJ IDEA 代码编辑功能也都受支持。
注意，无需声明 `main` 函数：编写的所有代码都将像在 `main` 主体中那样执行。

在草稿或工作表中完成代码编写后，请单击 __Run__。
执行结果将出现在代码右边的行中。

<div style="display: flex; align-items: center; margin-bottom: 10px;">
    <img
    src="{{ url_for('asset', path='images/tutorials/quick-run/scratch-run.png') }}"
    data-gif-src="{{ url_for('asset', path='images/tutorials/quick-run/scratch-run.gif') }}"
    class="gif-image">
</div>

### 交互模式

IntelliJ IDEA 可以自动运行草稿与工单中的代码。
要在停止输入时自动执行并获得结果，请勾选 __Interactive mode__。

<div style="display: flex; align-items: center; margin-bottom: 10px;">
    <img
    src="{{ url_for('asset', path='images/tutorials/quick-run/scratch-interactive.png') }}"
    data-gif-src="{{ url_for('asset', path='images/tutorials/quick-run/scratch-interactive.gif') }}"
    class="gif-image">
</div>

### 调用模块

可以在草稿与工单中使用 Kotlin 项目中的类或函数。

工单会自动从其所在的模块中访问类或函数。

要从草稿中使用项目中的类或函数，请照常使用 `import` 语句将它们导入到草稿文件中。
然后编写代码，并使用在 __Use classpath of module__ 列表中选择的适当模块运行它。

草稿与工单都使用已连接模块的已编译版本。
因此，如果修改了模块的源文件，则在重新构建模块后，更改才会将传播到草稿或工单中。
要在每次运行草稿或工单之前自动重新构建模块，请勾选 __Make before Run__。

![Scratch select module]({{ url_for('tutorial_img', filename='quick-run/scratch-select-module.png') }})

### 像 REPL 那样运行

若要计算草稿或工单中的每个特定表达式，请在勾选 __Use REPL__ 的情况下运行它。
该代码将以与 REPL 中相同的方式执行：代码行将按顺序运行，并提供每个调用的结果。
然后可以通过相应行中显示的名称 `res*` 来引用结果。

![Scratch REPL]({{ url_for('tutorial_img', filename='quick-run/scratch-repl.png') }})

## REPL

_REPL_（_<span title="Read">读取</span>-<span title="Eval">求值</span>-<span title="Print">输出</span>-<span title="Loop">循环</span>_）是一个交互式运行 Kotlin 代码的工具。
REPL 允许运行表达式与代码块，而无需创建项目或函数（如果不需要的话）。

如需在 IntelliJ IDEA 中运行 REPL，请打开 __Tools \| Kotlin \| Kotlin REPL__ 。

如需在操作系统命令行中运行 REPL，请从 Kotlin 独立编译器的目录中打开 __/bin/Kotlic-JVM __ 。

将打开 REPL 命令行界面。可以输入任何有效的 Kotlin 代码并查看结果。
结果将打印为具有自动生成的名称（如 `res*`）的变量。之后便可在 REPL 中运行的代码中使用此类变量。

<div style="display: flex; align-items: center; margin-bottom: 10px;">
    <img
    src="{{ url_for('asset', path='images/tutorials/quick-run/repl-run.png') }}"
    data-gif-src="{{ url_for('asset', path='images/tutorials/quick-run/repl-run.gif') }}"
    class="gif-image">
</div>

REPL 也支持多行输入。多行输入的结果为其最后一个表达式的值。

<div style="display: flex; align-items: center; margin-bottom: 10px;">
    <img
    src="{{ url_for('asset', path='images/tutorials/quick-run/repl-multi-line.png') }}"
    data-gif-src="{{ url_for('asset', path='images/tutorials/quick-run/repl-multi-line.gif') }}"
    class="gif-image">
</div>
