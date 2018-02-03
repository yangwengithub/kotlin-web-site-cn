---
type: doc
layout: reference
category: Basics
title: "编码规范"
---

# 编码规范

本页包含当前 Kotlin 语言的编码风格

> 注意：如需根据本风格指南配置 IntelliJ 格式化程序，请安装 Kotlin 插件
> 1.2.20 或更高版本，转到“Settings | Editor | Code Style | Kotlin”，点击右<!--
> -->上角的“Set from...”链接，并从菜单中选择“Predefined style / Kotlin style guide”。

## 源代码组织

### 目录结构

在混合语言项目中，Kotlin 源文件应当与 Java 源文件位于同一源文件根目录下，
并遵循相同的目录结构（每个文件应存储在与其 package 语句对应的目录中
）。

在纯 Kotlin 项目中，推荐的目录结构遵循省略了公共根包的包结构
（例如，如果项目中的所有代码都位于“org.example.kotlin”包及其<!--
-->子包中，那么“org.example.kotlin”包的文件应该直接放在源代码根目录下，而
“org.example.kotlin.foo.bar”中的文件应该放在源代码根目录下的“foo/bar”子目录中）。

### 源文件名称

如果 Kotlin 文件包含单个类（以及可能相关的顶层声明），那么文件名应该与<!--
-->该类的名称相同，并追加 .kt 扩展名。如果文件包含多个类或只包含顶层声明，
那么选择一个描述该文件所包含内容的名称，并以此命名该文件。使用首字母大写的驼峰风格
（例如 `ProcessDeclarations.kt`）。

文件的名称应该描述文件中代码的作用。因此，应避免在文件名中使用<!--
-->诸如“Util”之类的无意义词语。

### 源文件组织

鼓励多个声明（类、顶级函数或者属性）放在同一个 Kotlin 源文件中，
只要这些声明在语义上彼此紧密关联并且文件保持合理大小
（不超过几百行）。

特别是在为类定义与类的所有客户都相关的扩展函数时，
请将它们放在与类自身定义相同的地方。而在定义仅对<!--
-->指定客户有意义的扩展函数时，请将它们放在紧挨该客户代码之后。不要只是为了保存
“Foo 的所有扩展函数”而创建文件。

### 类布局

通常，一个类的内容按以下顺序排列：

- 属性声明与初始化块
- 次构造函数
- 方法声明
- 伴生对象

不要按字母顺序或者可见性对方法声明排序，也不要将常规方<!--
-->法与扩展方法分开。而是要把相关的东西放在一起，这样从上到下<!--
-->阅读类的人就能够跟进所发生事情的逻辑。选择一个顺序（高级别优先，或者相反）
并坚持下去。

将嵌套类放在紧挨使用这些类的代码之后。如果打算在外部使用嵌套类，而且类中并没有<!--
-->引用这些类，那么把它们放到末尾，在伴生对象之后。

### 接口实现布局

在实现一个接口时，实现成员的顺序应该与该接口的成员顺序相同（如果需要，
还要插入用于实现的额外的私有方法）

### 重载布局

在类中总是将重载放在一起。

## 命名规则

Kotlin 遵循 Java 命名约定。尤其是：

包的名称总是小写且不使用下划线（`org.example.myproject`）。
通常不鼓励使用多个词的名称，但是如果确实需要使用多个词，可以将它们连接在一起<!--
-->或使用驼峰（`org.example.myProject`）。

类和对象的名称以大写字母开头并使用驼峰：

``` kotlin
open class DeclarationProcessor { …… }

object EmptyDeclarationProcessor : DeclarationProcessor() { …… }
```

### 函数名
 
函数、属性与局部变量的名称以小写字母开头、使用驼峰而不使用下划线：

``` kotlin
fun processDeclarations() { …… }
var declarationCount = ……
``` 

例外：用于创建类实例的工厂函数可以与要创建的类具有相同的名称：

``` kotlin
abstract class Foo { …… }

class FooImpl : Foo { …… }

fun Foo(): Foo { return FooImpl(……) }
```

#### 测试方法的名称

当且仅当在测试中，可以使用反引号括起来的带空格的方法名。
（请注意，Android 运行时目前不支持这样的方法名。）测试代码中<!--
-->也允许方法名使用下划线。

``` kotlin
class MyTestCase {
     @Test fun `ensure everything works`() {
     }
     
     @Test fun ensureEverythingWorks_onAndroid() {
     }
}
```

### 属性名

常量名称（标有 `const` 的属性，或者保存不可变数据的没有自定义 `get` 函数<!--
-->的顶层/对象 `val` 属性）应该使用大写、下划线分隔的名称：

``` kotlin
const val MAX_COUNT = 8
val USER_NAME_FIELD = "UserName"
``` 

保存带有行为的对象或者可变数据的顶层/对象属性的名称应该使用常规驼峰名称：

``` kotlin
val mutableCollection: MutableSet<String> = HashSet()
```

保存单例对象引用的属性的名称可以使用与 `object` 声明相同的命名风格：

``` kotlin
val PersonComparator: Comparator<Person> = ...
```
 
对于枚举常量，可以使用大写、下划线分隔的名称
（`enum class Color { RED, GREEN }`）也可使用以大写字母开头的常规驼峰名称，具体取决于用途。
   
#### 幕后属性的名称

如果一个类有两个概念上相同的属性，一个是公共 API 的一部分，另一个是实现<!--
-->细节，那么使用下划线作为私有属性名称的前缀：

``` kotlin
class C {
    private val _elementList = mutableListOf<Element>()

    val elementList: List<Element>
         get() = _elementList
}
```

### 选择好名称

类的名称通常是用来解释类*是*什么的名词或者名词短语：`List`、 `PersonReader`。

方法的名称通常是动词或动词短语，说明该方法*做*什么：`close`、 `readPersons`。
修改对象或者返回一个新对象的名称也应遵循建议。例如 `sort` 是<!--
-->对一个集合就地排序，而 `sorted` 是返回一个排序后的集合副本。

名称应该表明实体的目的是什么，所以最好避免在名称中使用无意义的单词
（`Manager`、 `Wrapper` 等）。

当使用首字母缩写作为名称的一部分时，如果缩写由两个字母组成，就将其大写（`IOStream`）；
而如果缩写更长一些，就只大写首首字母（`XmlFormatter`、 `HttpInputStream`）。


## Formatting

In most cases, Kotlin follows the Java coding conventions.

Use 4 spaces for indentation. Do not use tabs.

For curly braces, put the opening brace in the end of the line where the construct begins, and the closing brace
on a separate line aligned vertically with the opening construct.

``` kotlin
if (elements != null) {
    for (element in elements) {
        // ...
    }
}
```

(Note: In Kotlin, semicolons are optional, and therefore line breaks are significant. The language design assumes 
Java-style braces, and you may encounter surprising behavior if you try to use a different formatting style.)

### Horizontal whitespace

Put spaces around binary operators (`a + b`). Exception: don't put spaces around the "range to" operator (`0..i`).

Do not put spaces around unary operators (`a++`)

Put spaces between control flow keywords (`if`, `when`, `for` and `while`) and the corresponding opening parenthesis.

Do not put a space before an opening parenthesis in a primary constructor declaration, method declaration or method call.

```kotlin
class A(val x: Int)

fun foo(x: Int) { }

fun bar() {
    foo(1)
}
```

Never put a space after `(`, `[`, or before `]`, `)`.

Never put a space around `.` or `?.`: `foo.bar().filter { it > 2 }.joinToString()`, `foo?.bar()`

Put a space after `//`: `// This is a comment`

Do not put spaces around angle brackets used to specify type parameters: `class Map<K, V> { ... }`

Do not put spaces around `::`: `Foo::class`, `String::length`

Do not put a space before `?` used to mark a nullable type: `String?`

As a general rule, avoid horizontal alignment of any kind. Renaming an identifier to a name with a different length
should not affect the formatting of either the declaration or any of the usages.

### Colon

Put a space before `:` in the following cases:

  * when it's used to separate a type and a supertype;
  * when delegating to a superclass constructor or a different constructor of the same class;
  * after the `object` keyword.
    
Don't put a space before `:` when it separates a declaration and its type.
 
Always put a space after `:`.

``` kotlin
abstract class Foo<out T : Any> : IFoo {
    abstract fun foo(a: Int): T
}

class FooImpl : Foo() {
    constructor(x: String) : this(x) {
        //...
    }
    
    val x = object : IFoo { ... } 
} 
```

### Class header formatting

Classes with a few primary constructor parameters can be written in a single line:

```kotlin
class Person(id: Int, name: String)
```

Classes with longer headers should be formatted so that each primary constructor parameter is in a separate line with indentation.
Also, the closing parenthesis should be on a new line. If we use inheritance, then the superclass constructor call or list of implemented interfaces
should be located on the same line as the parenthesis:

```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name) {

    // ...
}
```

For multiple interfaces, the superclass constructor call should be located first and then each interface should be located in a different line:

```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name),
    KotlinMaker {
    
    // ...
}
```

For classes with a long supertype list, put a line break after the colon and align all supertype names vertically:

```kotlin
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne {

    fun foo() {}
}
```

To clearly separate the class header and body when the class header is long, either put a blank line
following the class header (as in the example above), or put the opening curly brace on a separate line:

```kotlin
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne
{
    fun foo() {}
}
```

Use regular indent (4 spaces) for constructor parameters.

> Rationale: This ensures that properties declared in the primary constructor have the same indentation as properties
> declared in the body of a class.

### Modifiers

If a declaration has multiple modifiers, always put them in the following order:

``` kotlin
public / protected / private / internal
expect / actual
final / open / abstract / sealed / const
external
override
lateinit
tailrec
vararg
suspend
inner
enum / annotation
companion
inline
infix
operator
data
```

Place all annotations before modifiers:

``` kotlin
@Named("Foo")
private val foo: Foo
```

Unless you're working on a library, omit redundant modifiers (e.g. `public`).

### Annotation formatting

Annotations are typically placed on separate lines, before the declaration to which they are attached, and with the same indentation:

``` kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude
```

Annotations without arguments may be placed on the same line:

``` kotlin
@JsonExclude @JvmField
var x: String
```

A single annotation without arguments may be placed on the same line as the corresponding declaration:

``` kotlin
@Test fun foo() { ... }
```

### File annotations

File annotations are placed after the file comment (if any), before the `package` statement, and are separated from `package` with a blank line (to emphasize the fact that they target the file and not the package).

``` kotlin
/** License, copyright and whatever */
@file:JvmName("FooBar")

package foo.bar
```

### Function formatting

If the function signature doesn't fit on a single line, use the following syntax:

``` kotlin
fun longMethodName(
    argument: ArgumentType = defaultValue,
    argument2: AnotherArgumentType
): ReturnType {
    // body
}
```

Use regular indent (4 spaces) for function parameters.

> Rationale: Consistency with constructor parameters

Prefer using an expression body for functions with the body consisting of a single expression.

``` kotlin
fun foo(): Int {     // bad
    return 1 
}

fun foo() = 1        // good
```

### Expression body formatting

If the function has an expression body that doesn't fit in the same line as the declaration, put the `=` sign on the first line.
Indent the expression body by 4 spaces.

``` kotlin
fun f(x: String) =
    x.length
```

## Property formatting

For very simple read-only properties, consider one-line formatting:

```kotlin
val isEmpty: Boolean get() = size == 0
```

For more complex properties, always put `get` and `set` keywords on separate lines:

```kotlin
val foo: String
    get() {
        // ...
    }

```

For properties with an initializer, if the initializer is long, add a line break after the equals sign
and indent the initializer by four spaces:

```kotlin
private val defaultCharset: Charset? =
    EncodingRegistry.getInstance().getDefaultCharsetForPropertiesFiles(file)
```

### Formatting control flow statements

If the condition of an `if` or `when` statement is multiline, always use curly braces around the body of the statement.
Indent each subsequent line of the condition by 4 spaces relative to statement begin. 
Put the closing parentheses of the condition together with the opening curly brace on a separate line:

``` kotlin
if (!component.isSyncing &&
    !hasAnyKotlinRuntimeInScope(module)
) {
    return createKotlinNotConfiguredPanel(module)
}
```

> Rationale: Tidy alignment and clear separation of condition and statement body

Put the `else`, `catch`, `finally` keywords, as well as the `while` keyword of a do/while loop, on the same line as the 
preceding curly brace:

``` kotlin
if (condition) {
    // body
} else {
    // else part
}

try {
    // body
} finally {
    // cleanup
}
```

In a `when` statement, if a branch is more than a single line, consider separating it from adjacent case blocks with a blank line:

``` kotlin
private fun parsePropertyValue(propName: String, token: Token) {
    when (token) {
        is Token.ValueToken ->
            callback.visitValue(propName, token.value)

        Token.LBRACE -> { // ...
        }
    }
}
```

Put short branches on the same line as the condition, without braces.

``` kotlin
when (foo) {
    true -> bar() // good
    false -> { baz() } // bad
}
```


### Method call formatting

In long argument lists, put a line break after the opening parenthesis. Indent arguments by 4 spaces. 
Group multiple closely related arguments on the same line.

``` kotlin
drawSquare(
    x = 10, y = 10,
    width = 100, height = 100,
    fill = true
)
```

Put spaces around the `=` sign separating the argument name and value.

### Chained call wrapping

When wrapping chained calls, put the . character or the `?.` operator on the next line, with a single indent:

``` kotlin
val anchor = owner
    ?.firstChild!!
    .siblings(forward = true)
    .dropWhile { it is PsiComment || it is PsiWhiteSpace }
```

The first call in the chain usually should have a line break before it, but it's OK to omit it the code makes more sense that way.

### Lambda formatting

In lambda expressions, spaces should be used around the curly braces, as well as around the arrow which separates the parameters
from the body. If a call takes a single lambda, it should be passed outside of parentheses whenever possible.

``` kotlin
list.filter { it > 10 }
```

If assigning a label for a lambda, do not put a space between the label and the opening curly brace:

``` kotlin
fun foo() {
    ints.forEach lit@{
        // ...
    }
}
```

When declaring parameter names in a multiline lambda, put the names on the first line, followed by the arrow and the newline:

``` kotlin
appendCommaSeparated(properties) { prop ->
    val propertyValue = prop.get(obj)  // ...
}
```

If the parameter list is too long to fit on a line, put the arrow on a separate line:

``` kotlin
foo {
   context: Context,
   environment: Env
   ->
   context.configureEnv(environment)
}
```

## Documentation comments

For longer documentation comments, place the opening `/**` on a separate line and begin each subsequent line
with an asterisk:

``` kotlin
/**
 * This is a documentation comment
 * on multiple lines.
 */
```

Short comments can be placed on a single line:

``` kotlin
/** This is a short documentation comment. */
```

Generally, avoid using `@param` and `@return` tags. Instead, incorporate the description of parameters and return values
directly into the documentation comment, and add links to parameters wherever they are mentioned. Use `@param` and
`@return` only when a lengthy description is required which doesn't fit into the flow of the main text.

``` kotlin
// Avoid doing this:

/**
 * Returns the absolute value of the given number.
 * @param number The number to return the absolute value for.
 * @return The absolute value.
 */
fun abs(number: Int) = ...

// Do this instead:

/**
 * Returns the absolute value of the given [number].
 */
fun abs(number: Int) = ...
```

## Avoiding redundant constructs

In general, if a certain syntactic construction in Kotlin is optional and highlighted by the IDE
as redundant, you should omit it in your code. Do not leave unnecessary syntactic elements in code
just "for clarity".

### Unit

If a function returns Unit, the return type should be omitted:

``` kotlin
fun foo() { // ": Unit" is omitted here

}
```

### Semicolons

Omit semicolons whenever possible.

### String templates

Don't use curly braces when inserting a simple variable into a string template. Use curly braces only for longer expressions.

``` kotlin
println("$name has ${children.size} children")
```


## Idiomatic use of language features

### Immutability

Prefer using immutable data to mutable. Always declare local variables and properties as `val` rather than `var` if
they are not modified after initialization.

Always use immutable collection interfaces (`Collection`, `List`, `Set`, `Map`) to declare collections which are not
mutated. When using factory functions to create collection instances, always use functions that return immutable
collection types when possible:

``` kotlin
// Bad: use of mutable collection type for value which will not be mutated
fun validateValue(actualValue: String, allowedValues: HashSet<String>) { ... }

// Good: immutable collection type used instead
fun validateValue(actualValue: String, allowedValues: Set<String>) { ... }

// Bad: arrayListOf() returns ArrayList<T>, which is a mutable collection type
val allowedValues = arrayListOf("a", "b", "c")

// Good: listOf() returns List<T>
val allowedValues = listOf("a", "b", "c")
```

### Default parameter values

Prefer declaring functions with default parameter values to declaring overloaded functions.

``` kotlin
// Bad
fun foo() = foo("a")
fun foo(a: String) { ... }

// Good
fun foo(a: String = "a") { ... }
```

### Type aliases

If you have a functional type or a type with type parameters which is used multiple times in a codebase, prefer defining
a type alias for it:

```kotlin
typealias MouseClickHandler = (Any, MouseEvent) -> Unit
typealias PersonIndex = Map<String, Person>
```

### Lambda parameters

In lambdas which are short and not nested, it's recommended to use the `it` convention instead of declaring the parameter
explicitly. In nested lambdas with parameters, parameters should be always declared explicitly.


### Returns in a lambda

Avoid using multiple labeled returns in a lambda. Consider restructuring the lambda so that it will have a single exit point.
If that's not possible or not clear enough, consider converting the lambda into an anonymous function.

Do not use a labeled return for the last statement in a lambda.

### Named arguments

Use the named argument syntax when a method takes multiple parameters of the same primitive type, or for parameters of `Boolean` type,
unless the meaning of all parameters is absolutely clear from context.

``` kotlin
drawSquare(x = 10, y = 10, width = 100, height = 100, fill = true)
```

### Using conditional statements

Prefer using the expression form of `try`, `if` and `when`. Examples:

``` kotlin
return if (x) foo() else bar()

return when(x) {
    0 -> "zero"
    else -> "nonzero"
}
```

The above is preferable to:

``` kotlin
if (x)
    return foo()
else
    return bar()
    
when(x) {
    0 -> return "zero"
    else -> return "nonzero"
}    
```

### `if` versus `when`

Prefer using `if` for binary conditions instead of `when`. Instead of

``` kotlin
when (x) {
    null -> ...
    else -> ...
}
```

use `if (x == null) ... else ...`

Prefer using `when` if there are three or more options.

### Using nullable `Boolean` values in conditions

If you need to use a nullable `Boolean` in a conditional statement, use `if (value == true)` or `if (value == false)` checks.

### Using loops

Prefer using higher-order functions (`filter`, `map` etc.) to loops. Exception: `forEach` (prefer using a regular `for` loop instead,
unless the receiver of `forEach` is nullable or `forEach` is used as part of a longer call chain).

When making a choice between a complex expression using multiple higher-order functions and a loop, understand the cost
of the operations being performed in each case and keep performance considerations in mind. 

### Loops on ranges

Use the `until` function to loop over an open range:

```kotlin
for (i in 0..n - 1) { ... }  // bad
for (i in 0 until n) { ... }  // good
```

### Using strings

Prefer using string templates to string concatenation.

Prefer to use multiline strings instead of embedding `\n` escape sequences into regular string literals.

To maintain indentation in multiline strings, use `trimIndent` when the resulting string does not require any internal
indentation, or `trimMargin` when internal indentation is required:

``` kotlin
assertEquals("""Foo
                Bar""".trimIndent(), value)

val a = """if(a > 1) {
          |    return a
          |}""".trimMargin()
```

### Functions vs Properties

In some cases functions with no arguments might be interchangeable with read-only properties. 
Although the semantics are similar, there are some stylistic conventions on when to prefer one to another.

Prefer a property over a function when the underlying algorithm:

* does not throw
* is cheap to calculate (or caсhed on the first run)
* returns the same result over invocations if the object state hasn't changed

### Using extension functions

Use extension functions liberally. Every time you have a function that works primarily on an object, consider making it
an extension function accepting that object as a receiver. To minimize API pollution, restrict the visibility of
extension functions as much as it makes sense. As necessary, use local extension functions, member extension functions,
or top-level extension functions with private visibility.

### Using infix functions

Declare a function as infix only when it works on two objects which play a similar role. Good examples: `and`, `to`, `zip`.
Bad example: `add`.

Don't declare a method as infix if it mutates the receiver object.

### Factory functions

If you declare a factory function for a class, avoid giving it the same name as the class itself. Prefer using a distinct name
making it clear why the behavior of the factory function is special. Only if there is really no special semantics,
you can use the same name as the class.

Example:

``` kotlin
class Point(val x: Double, val y: Double) {
    companion object {
        fun fromPolar(angle: Double, radius: Double) = Point(...)
    }
}
```

If you have an object with multiple overloaded constructors that don't call different superclass constructors and
can't be reduced to a single constructor with default argument values, prefer to replace the overloaded constructors with
factory functions.

### Platform types

A public function/method returning an expression of a platform type must declare its Kotlin type explicitly:

``` kotlin
fun apiCall(): String = MyJavaApi.getProperty("name")
```

Any property (package-level or class-level) initialised with an expression of a platform type must declare its Kotlin type explicitly:

``` kotlin
class Person {
    val name: String = MyJavaApi.getProperty("name")
}
```

A local value initialised with an expression of a platform type may or may not have a type declaration:

``` kotlin
fun main(args: Array<String>) {
    val name = MyJavaApi.getProperty("name")
    println(name)
}
```

### Using scope functions apply/with/run/also/let

Kotlin provides a variety of functions to execute a block of code in the context of a given object. To choose the correct
function, consider the following:

  * Are you calling methods on multiple objects in the block, or passing the instance of the context object as an 
    argument? If you are, use one of the functions that allows you to access the context object as `it`,
    not `this` (`also` or `let`). Use `also` if the receiver is not used at all in the block.
    
``` kotlin
// Context object is 'it'
class Baz {
    var currentBar: Bar?
    val observable: Observable

    val foo = createBar().also {
        currentBar = it                    // Accessing property of Baz
        observable.registerCallback(it)    // Passing context object as argument
    }
}

// Receiver not used in the block
val foo = createBar().also {
    LOG.info("Bar created")
}

// Context object is 'this'
class Baz {
    val foo: Bar = createBar().apply {
        color = RED    // Accessing only properties of Bar
        text = "Foo"
    }
}
```    
    
  * What should the result of the call be? If the result needs to be the context object, use `apply` or `also`.
    If you need to return a value from the block, use `with`, `let` or `run`
    
``` kotlin
// Return value is context object
class Baz {
    val foo: Bar = createBar().apply {
        color = RED    // Accessing only properties of Bar
        text = "Foo"
    }
}


// Return value is block result
class Baz {
    val foo: Bar = createNetworkConnection().let {
        loadBar()
    }
}
```    
    
  * Is the context object nullable, or is it evaluated as a result of a call chain? If it is, use `apply`, `let` or `run`.
    Otherwise, use `with` or `also`.
     
``` kotlin
// Context object is nullable
person.email?.let { sendEmail(it) }

// Context object is non-null and accessible directly
with(person) {
    println("First name: $firstName, last name: $lastName")
}
```


## Coding conventions for libraries

When writing libraries, it's recommended to follow an additional set of rules to ensure API stability:

 * Always explicitly specify member visibility (to avoid accidentally exposing declarations as public API)
 * Always explicitly specify function return types and property types (to avoid accidentally changing the return type
   when the implementation changes)
 * Provide KDoc comments for all public members, with the exception of overrides that do not require any new documentation
   (to support generating documentation for the library)
