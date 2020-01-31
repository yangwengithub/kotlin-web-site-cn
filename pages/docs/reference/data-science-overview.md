---
type: doc
layout: reference
category: "Introduction"
title: "Kotlin 用于数据科学"
---

# Kotlin 用于数据科学

![Kotlin for data science]({{ url_for('asset', path='images/landing/data-science/data-science-overview.png')}})

数据科学在信息技术中具有特殊地位：它同时包含软件开发与<!--
-->科学研究两方面。作为一门学科，数据科学涵盖了广泛的领域：数据工程、
数据分析、机器学习、可视化等等。

为了涵盖所有这些不同领域，软件行业有许多用于数据科学的技术与工具。
其中包含框架、特定的 IDE（称为 *notebook*）、绘图工具以及<!--
-->专为数据分析与数学研究设计的编程语言。

通用语言也可以应用于数据科学领域，而 Kotlin 已经为数据科学所采用。
在此我们会介绍一些关于将 Kotlin 用于数据科学时的实用知识。

## 工具

现代软件开发人员很少会在纯文本编辑器中编写代码并在命令行运行。
相反，大家倾向于使用可以在一个工具中处理所有开发任务的集成开发环境（IDE，Integrated Development Environment）。
数据科学家也有类似的工具，称为 *notebook*。notebook 让用户可以进行研究并将其存储在单个环境中。
在一个 notebook 中，可以在代码旁编写叙述性文本、执行代码块及以任何<!--
-->所需格式（输出文本、表格、数据可视化等等）查看结果。

Kotlin 提供了与两个流行的 notebook 的集成：Jupyter 与 Apache Zeppelin，它们都支持编写及<!--
-->运行 Kotlin 代码块。

### Jupyter 内核

开源项目 [Jupyter](https://jupyter.org/) 提供了著名的基于 web 的开发环境 **Jupyter Notebook**。
对于代码执行，Jupyter 使用*内核*的概念——独立运行的不同组件，并且这些组件根据请求执行代码，
例如，当在一个 notebook 中点击 **Run** 时。

Jupyter 团队维护了一个内核——运行 Python 代码的 IPython。
但是，还有其他社区维护的用于不同语言的各种内核。
其中包括**用于 Jupyter notbook 的 Kotlin 内核**。有了这个内核，就可以在 Jupyter
notebook 中编写并运行 Kotlin 代码，以及使用以 Java 或 Kotlin 编写的第三方数据科学框架。

#### 设置 Kotlin 内核

Kotlin 内核需要安装 Java 8。

请使用 [Conda](https://docs.conda.io/projects/conda/en/latest/) 安装该内核：

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
conda install kotlin-jupyter-kernel -c jetbrains
```

</div>

一旦内核安装完毕，就可以运行 Jupyter notebook 并切换到 Kotlin 内核。
仅此而已，然后就可以在 notebook 中编写并运行 Kotlin 了！ 

![Kotlin in Jupyter notebook]({{ url_for('asset', path='images/landing/data-science/jupyter-kotlin.png')}})

可以在[这里](https://github.com/cheptsov/kotlin-jupyter-demo/blob/master/index.ipynb)找到关于 Jupyter 的 Kotlin 内核的更多信息。

### Zeppelin 解释器

[Apache Zeppelin](http://zeppelin.apache.org/) 是一个流行的基于 web 的交互式数据分析解决方案。
Zeppelin 为 [Apache Spark](http://zeppelin.apache.org/docs/latest/interpreter/spark.html)
集群计算系统提供了强大的支持，这对于数据工程特别有用。Spark 提供了多种语言的高级 API。
 
Zeppelin 中的语言支持由*解释器*提供，解释器是让用户能够使用指定语言或者数据处理后端的插件。
对于不同的编程语言有许多社区维护的解释器。
我们提供了增加 Kotlin 支持的**用于 Apache Zeppelin 的 Kotlin 解释器**。

#### 设置带 Kotlin 解释器的 Zeppelin

目前，最新版本的 Zeppelin（0.8.2）并未内置 Kotlin 解释器。
不过，在 Zeppelin 的 master 分支中有。
因此，为了向 Zeppelin 添加 Kotlin 支持，需要从源代码构建自己的版本。

构建 Zeppelin 的定制版需要：

* [Git](https://git-scm.com/)
* [Maven](https://maven.apache.org/install.html),
* [JDK 8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
* 列在[此处](https://zeppelin.apache.org/docs/latest/setup/basics/how_to_build.html#build-requirements)各项依赖

首先，从 Zeppelin 版本库中检出 master 分支：

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
git clone --depth=1 git@github.com:apache/zeppelin.git
```
</div>

或者

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
git clone --depth=1 https://github.com/apache/zeppelin.git
```
</div>

使用 Maven 构建 Zeppelin，请切换到 Zeppelin 目录并运行以下命令：

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
mvn clean package -DskipTests -Pspark-2.4 -Pscala-2.11
```
</div>

然后使用以下命令运行 Zeppelin：

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
./bin/zeppelin-daemon.sh start
```
</div>

现在可以在 `http://localhost:8089` 打开 Zeppelin UI 了。

如需了解如何在 Spark 集群中部署带 Kotlin 支持的 Zeppelin，请参见[这个说明](/docs/tutorials/zeppelin-spark-cluster.html)。

## 相关库

对于软件工程来说，任何领域的重要组成部分都有相关领域框架的可用性。
对于数据科学，包括诸如机器学习、数据分析、可视化等领域。
幸运的是，已经有很多用 Kotlin 编写的数据科学框架。
更完美的是还有更多用 Java 编写的框架，因为可以在 Kotlin 代码中无缝调用 Java 框架。

以下是可能对数据科学有用的库的两个简短列表。

### Kotlin 库
* [kotlin-statistics](https://github.com/thomasnield/kotlin-statistics) 是一个为<!--
-->探索性统计与生产统计中提供扩展函数的库。它支持基本的数字列表/序列/数组函数（从 `sum` 到 `skewness`）、
切片操作符（诸如 `countBy`、 `simpleRegressionBy`）、分箱（binning）操作符、离散 PDF 采样、
朴素贝叶斯分类器、聚类、线性回归等等。

* [kmath](https://github.com/mipt-npm/kmath) 是一个受 [NumPy](https://numpy.org/) 启发的库。
这个库支持代数结构与运算、类数组结构、数学表达式、直方图、
流运算、[commons-math](http://commons.apache.org/proper/commons-math/) 与
[koma](https://github.com/kyonifer/koma) 的包装等等。

* [krangl](https://github.com/holgerbrandl/krangl) 是一个受 R 语言的 [dplyr](https://dplyr.tidyverse.org/)
与 Python 的 [pandas](https://pandas.pydata.org/) 启发的库。这个库提供了采用函数式风格 API
进行数据操作的功能；它还包括过滤、转换、聚合与重塑表格数据的函数。

* [lets-plot](https://github.com/JetBrains/lets-plot) 是一个用 Kotlin 编写的统计数据绘图库。
Lets-Plot 是多平台的，不仅可以用于 JVM，还可以用于 JS 与 Python。更多信息请参见[下文](#lets-plot-for-kotlin)。

* [kravis](https://github.com/holgerbrandl/kravis) 是另一个用于表格数据可视化的库，其灵感来自于
Python 的 [ggplot](https://ggplot2.tidyverse.org/)。

### Java 库

因为 Kotlin 提供了与 Java 互操作的头等支持，所以也可以在用于数据科学的 Kotlin 代码中使用 Java 库。
以下是这些库的一些示例：

* [DeepLearning4J](https://deeplearning4j.org/)——一个 Java 深度学习库

* [ND4J](http://nd4j.org/)——用于 JVM 的高效矩阵数学库

* [Dex](https://github.com/PatMartin/Dex)——一个基于 Java 的数据可视化工具

* [Smile](https://github.com/haifengl/smile)——一个全面的机器学习、自然语言处理、线性代数、
 图、插值与可视化系统

* [Apache Commons Math](http://commons.apache.org/proper/commons-math/)——一个 Java 通用数学、统计与机器学习库

* [OptaPlanner](https://www.optaplanner.org/)——一个用于优化规划问题的求解器实用程序

* [Charts](https://github.com/HanSolo/charts)——一个正在开发中的科学 JavaFX 图表库

* [CoreNLP](https://stanfordnlp.github.io/CoreNLP/)——一个自然语言处理工具包

* [Apache Mahout](https://mahout.apache.org/)——一个回归、聚类与推荐的分布式框架

* [Weka](https://www.cs.waikato.ac.nz/ml/index.html)——一组用于数据挖掘任务的机器学习算法

如果这个列表还不能满足需求，可以在 Thomas Nield 的
[**Kotlin 数据科学资源**](https://github.com/thomasnield/kotlin-data-science-resources)摘要中找到更多选项。

### Lets-Plot for Kotlin

**Lets-Plot for Kotlin** is a Kotlin API for the [Lets-Plot](https://github.com/JetBrains/lets-plot) library - 
an open-source plotting library for statistical data written entirely in Kotlin. Lets-Plot was built on the concept of 
layered graphics first described in Leland Wilkinson's work [The Grammar of Graphics](https://www.goodreads.com/book/show/2549408.The_Grammar_of_Graphics)
and later implemented in the [ggplot2](https://ggplot2.tidyverse.org/) package for R.

Lets-Plot for Kotlin is tightly integrated with the [Kotlin kernel for Jupyter notebooks](#jupyter-内核).
Once you have the Kotlin kernel installed and enabled, add the following line to a Jupyter notebook:

<div class="sample" markdown="1" mode="shell" theme="idea">

```
%use lets-plot
```
</div>

That’s it, now you can call functions from Lets-Plot and see the results.

![Lets-Plot diagram]({{ url_for('asset', path='images/landing/data-science/lets-plot.png')}})

### NumPy 的 Kotlin 绑定

[**KNumpy**](https://github.com/kotlin/kotlin-numpy/) (**Kotlin Bindings for NumPy**) is a Kotlin library that enables calling NumPy functions from the Kotlin code.
[NumPy](https://numpy.org/) is a popular package for scientific computing with Python. It provides powerful capabilities
for multi-dimensional array processing, linear algebra, Fourier transform, random numbers, and other mathematical tasks. 

KNumpy provides statically typed wrappers for NumPy functions. Thanks to the functional capabilities of Kotlin,
the API of KNumpy is very similar to the one for NumPy. This lets developers that are experienced with NumPy easily switch to KNumpy.
Here are two equal code samples:

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```python
# Python

import numpy as np
a = np.arange(15).reshape(3, 5)

print(a.shape == (3, 5))        # True
print(a.ndim == 2)              # True
print(a.dtype.name)             # 'int64'

b = (np.arange(15) ** 2).reshape(3, 5)
```
</div>

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only>

```kotlin
// Kotlin 

import org.jetbrains.numkt.*

fun main() {
    val a = arange(15).reshape(3, 5)
    
    println(a.shape.contentEquals(intArrayOf(3, 5))) // true
    println(a.ndim == 2)                             // true
    println(a.dtype)                                 // class java.lang.Integer

    // create an array of ints, we square each element and the shape to (3, 5) 
    val b = (arange(15) `**` 2).reshape(3, 5)
}
```
</div>

Unlike Python, Kotlin is a statically typed language. This lets you avoid entire classes of runtime errors with KNumpy:
the Kotlin compiler detects them at earlier stages.

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```python
# Python

import numpy as np

# ...

a = np.ones((3, 3), dtype=int) * 3
b = np.random.random((3, 3))

b *= a # Success
a *= b # TypeError at runtime 
```
</div>

<div class="sample" markdown="1" mode="kotlin" theme="idea" data-highlight-only>

```kotlin
// Kotlin 

// ...

val a = ones<Int>(3, 3) * 3
val b = Random.random(3, 3)

b *= a // Success
a *= b // Compilation error: 
// Type mismatch: inferred type is KtNDArray<Double> but KtNDArray<Int> was expected
```
</div>

