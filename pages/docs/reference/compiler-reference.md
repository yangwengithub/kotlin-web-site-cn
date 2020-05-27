---
type: doc
layout: reference
title: "Kotlin 编译器选项"
---

# Kotlin 编译器选项

每个 Kotlin 版本都包含支持目标的编译器：[目标平台](native-overview.html#目标平台)的 JVM、JavaScript 与 native 二进制文件。

当单击 Kotlin 项目的 __Compile__ 或 __Run__ 按钮时，IDE 会使用这些编译器。

还可以按照[使用命令行编译器](/docs/tutorials/command-line.html)
教程中所述从命令行手动运行 Kotlin 编译器。例如：

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
$ kotlinc hello.kt -include-runtime -d hello.jar
```

</div>
 
## 编译器选项

Kotlin 编译器具有许多用于定制编译过程的选项。
此页面列出了针对不同目标的编译器选项，并提供了每个选项的描述。

有几种方法可以设置编译器选项及其值（_编译器参数_）：
- 在 IntelliJ IDEA 的 __Settings | Build, Execution, Deployment | Compilers | Kotlin Compiler__ 窗口中，
在 __Additional command-line parameters__ 文本框中输入编译器参数。
- 如果使用 Gradle，请在 Kotlin 编译任务的 `kotlinOptions` 属性中指定编译器参数。
有关详细信息，请参见[使用 Gradle](using-gradle.html#编译器选项)。
- 如果使用 Maven，请在 Maven 插件节点的 `<configuration>` 元素中指定编译器参数。
有关详细信息，请参见[使用 Maven](using-maven.html#指定编译器选项)。
- 如果运行命令行编译器，则将编译器参数直接添加到效用函数调用中，或将其写入 [argfile](#argfile)。

##  公共选项

以下选项对于所有 Kotlin 编译器都是通用的。

### `-version` 

显示编译器版本。
{:.details-group}

### `-nowarn`

禁止编译器在编译期间显示警告。
{:.details-group}

### `-Werror`

将所有警告转换为编译错误。
{:.details-group}

### `-verbose`

启用详细的日志记录输出，其中包括编译过程的详细信息。
{:.details-group}

### `-script`

检查 Kotlin 脚本文件。
使用此选项调用时，编译器将执行给定参数中的第一个 Kotlin 脚本文件（`*.kts`）。
{:.details-group}

### `-help` (`-h`)

显示用法信息并退出。仅显示标准选项。
要显示高级选项，请使用 `-X`。
{:.details-group}

### `-X`

显示有关高级选项的信息并退出。这些选项当前不稳定：
它们的名称与行为可能会更改，恕不另行通知。
{:.details-group}

### `-kotlin-home <path>`

指定用于发现运行时库的 Kotlin 编译器的自定义路径。
{:.details-group}
  
### `-P plugin:<pluginId>:<optionName>=<value>`

将选项传递给 Kotlin 编译器插件。
可用插件及其选项列在[编译器插件](compiler-plugins.html)中。
{:.details-group}
  
### `-language-version <version>`

提供与指定版本的 Kotlin 的源兼容性。
{:.details-group}

### `-api-version <version>`

仅允许使用 Kotlin 捆绑库指定版本中的声明。
{:.details-group}

### `-progressive`

为编译器启用[渐进模式](whatsnew13.html#渐进模式)。
{:.details-group}
在渐进模式下，针对不稳定代码的弃用与错误修复将立即生效，
而无需经历正常的迁移周期。
以渐进模式编写的代码是向后兼容的。
但是，以非渐进模式编写的代码可能会在渐进模式下导致编译错误。
{:.details-group}

{:#argfile}

### `@<argfile>`

从给定文件中读取编译器选项。这样的文件可以包含带有值与源文件路径的编译器选项。
选项与路径应由空格分隔。例如：
{:.details-group}

<div class="sample" markdown="1" mode="shell" theme="idea">

```
-include-runtime -d hello.jar
hello.kt
```

</div>

要传递包含空格的值，请用单引号（**'**）或双引号（**"**）引起来。
如果值中包含引号，请使用反斜杠（**\\**）对其进行转义。
{:.details-group}

<div class="sample" markdown="1" mode="shell" theme="idea">
     
```
-include-runtime -d 'My folder'
```
     
</div>

还可以传递多个参数文件，例如，将编译器选项与源文件分开。
{:.details-group}

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
$ kotlinc @compiler.options @classes
```

</div>

如果文件位于与当前目录不同的位置，请使用相对路径。
{:.details-group}

<div class="sample" markdown="1" mode="shell" theme="idea">

```bash
$ kotlinc @options/compiler.options hello.kt
```

</div>
    
## Kotlin/JVM 编译器选项

The Kotlin compiler for JVM compiles Kotlin source files into Java class files. 
The command-line tools for Kotlin to JVM compilation are `kotlinc` and `kotlinc-jvm`.
You can also use them for executing Kotlin script files.

In addition to the [common options](#公共选项), Kotlin/JVM compiler has the options listed below.

### `-classpath <path>` (`-cp <path>`)

Search for class files in the specified paths. Separate elements of the classpath with system path separators (**;** on Windows, **:** on macOS/Linux).
The classpath can contain file and directory paths, ZIP, or JAR files.
{:.details-group}

### `-d <path>`

Place the generated class files into the specified location. The location can be a directory, a ZIP, or a JAR file. 
{:.details-group}

### `-include-runtime`

Include the Kotlin runtime into the resulting JAR file. Makes the resulting archive runnable on any Java-enabled 
environment.
{:.details-group}

### `-jdk-home <path>`

Use a custom JDK home directory to include into the classpath if it differs from the default `JAVA_HOME`.
{:.details-group}

### `-jvm-target <version>`

Specify the target version of the generated JVM bytecode. Possible values are `1.6`, `1.8`, `9`, `10`, `11`, `12`, and `13`.
The default value is `1.6`.
{:.details-group}

### `-java-parameters`

Generate metadata for Java 1.8 reflection on method parameters.
{:.details-group}

### `-module-name <name>`

Set a custom name for the generated `.kotlin_module` file.
{:.details-group}
  
### `-no-jdk`

Don't automatically include the Java runtime into the classpath.
{:.details-group}

### `-no-reflect`

Don't automatically include the Kotlin reflection (`kotlin-reflect.jar`) into the classpath.
{:.details-group}

### `-no-stdlib`

Don't automatically include the Kotlin/JVM stdlib (`kotlin-stdlib.jar`) and Kotlin reflection (`kotlin-reflect.jar`)
into the classpath. 
{:.details-group}
  
### `-script-templates <classnames[,]>`

Script definition template classes. Use fully qualified class names and separate them with commas (**,**).
{:.details-group}


## Kotlin/JS 编译器选项

The Kotlin compiler for JS compiles Kotlin source files into JavaScript code. 
The command-line tool for Kotlin to JS compilation is `kotlinc-js`.

In addition to the [common options](#公共选项), Kotlin/JS compiler has the options listed below.

### `-libraries <path>`

Paths to Kotlin libraries with `.meta.js` and `.kjsm` files, separated by the system path separator.
{:.details-group}

### `-main {call|noCall}`

Define whether the `main` function should be called upon execution.
{:.details-group}

### `-meta-info`

Generate `.meta.js` and `.kjsm` files with metadata. Use this option when creating a JS library.
{:.details-group}

### `-module-kind {plain|amd|commonjs|umd}`

The kind of JS module generated by the compiler:
{:.details-group}
- `plain` - a plain JS module;
- `commonjs` - a [CommonJS](http://www.commonjs.org/) module;
- `amd` - an [Asynchronous Module Definition](https://en.wikipedia.org/wiki/Asynchronous_module_definition) module;
- `umd` - a [Universal Module Definition](https://github.com/umdjs/umd) module.
    
To learn more about the different kinds of JS module and the distinctions between them,
see [this](https://www.davidbcalhoun.com/2014/what-is-amd-commonjs-and-umd/) article.
{:.details-group}

### `-no-stdlib`

Don't automatically include the default Kotlin/JS stdlib into the compilation dependencies.
{:.details-group}

### `-output <filepath>`

Set the destination file for the compilation result. The value must be a path to a `.js` file including its name.
{:.details-group}

### `-output-postfix <filepath>`

Add the content of the specified file to the end of the output file.
{:.details-group}

### `-output-prefix <filepath>`

Add the content of the specified file to the beginning of the output file.
{:.details-group}

### `-source-map`

Generate the source map.
{:.details-group}

### `-source-map-base-dirs <path>`

Use the specified paths as base directories. Base directories are used for calculating relative paths in the source map.
{:.details-group}

### `-source-map-embed-sources {always|never|inlining}`

Embed source files into the source map.
{:.details-group}

### `-source-map-prefix`

Add the specified prefix to paths in the source map.
{:.details-group}


## Kotlin/Native 编译器选项

Kotlin/Native compiler compiles Kotlin source files into native binaries for the [supported platforms](native-overview.html#目标平台). 
The command-line tool for Kotlin/Native compilation is `kotlinc-native`.

In addition to the [common options](#公共选项), Kotlin/Native compiler has the options listed below.


### `-enable-assertions` (`-ea`)

Enable runtime assertions in the generated code.
{:.details-group}
    
### `-g`

Enable emitting debug information.
{:.details-group}
    
### `-generate-test-runner` (`-tr`)

Produce an application for running unit tests from the project.
{:.details-group}    
### `-generate-worker-test-runner` (`-trw`)

Produce an application for running unit tests in a [worker thread](native/concurrency.html#worker).
{:.details-group}
    
### `-generate-no-exit-test-runner` (`-trn`)

Produce an application for running unit tests without an explicit process exit.
{:.details-group}
    
### `-include-binary <path>` (`-ib <path>`)

Pack external binary within the generated klib file.
{:.details-group}
    
### `-library <path>` (`-l <path>`)

Link with the library. To learn about using libraries in Kotlin/native projects, see 
[Kotlin/Native libraries](native/libraries.html).
{:.details-group}

### `-library-version <version>` (`-lv`)

Set the library version.
{:.details-group}
    
### `-list-targets`

List the available hardware targets.
{:.details-group}

### `-manifest <path>`

Provide a manifest addend file.
{:.details-group}

### `-module-name <name>`

Specify a name for the compilation module.
This option can also be used to specify a name prefix for the declarations exported to Objective-C:
[How do I specify a custom Objective-C prefix/name for my Kotlin framework?](native/faq.html#q-how-do-i-specify-a-custom-objective-c-prefixname-for-my-kotlin-framework)
{:.details-group}

### `-native-library <path>`(`-nl <path>`)

Include the native bitcode library.
{:.details-group}

### `-no-default-libs`

Disable linking user code with the [default platform libraries](native/platform_libs.html) distributed with the compiler.
{:.details-group}
    
### `-nomain`

Assume the `main` entry point to be provided by external libraries.
{:.details-group}

### `-nopack`

Don't pack the library into a klib file.
{:.details-group}

### `-linker-option`

Pass an argument to the linker during binary building. This can be used for linking against some native library.
{:.details-group}

### `-linker-options <args>`

Pass multiple arguments to the linker during binary building. Separate arguments with whitespaces.
{:.details-group}

### `-nostdlib`

Don't link with stdlib.
{:.details-group}

### `-opt`

Enable compilation optimizations.
{:.details-group}

### `-output <name>` (`-o <name>`)

Set the name for the output file.
{:.details-group}

### `-entry <name>` (`-e <name>`)

Specify the qualified entry point name.
{:.details-group}

### `-produce <output>` (`-p`)

Specify output file kind:
{:.details-group}
- `program`
- `static`
- `dynamic`
- `framework`
- `library`
- `bitcode`

### `-repo <path>` (`-r <path>`)

Library search path. For more information, see [Library search sequence](native/libraries.html#库搜索顺序).
{:.details-group}

### `-target <target>`

Set hardware target. To see the list of available targets, use the [`-list-targets`](#-list-targets) option.
{:.details-group}

  
