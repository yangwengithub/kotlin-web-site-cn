---
type: tutorial
layout: tutorial
title:  "用 Spring Boot 创建 RESTful Web 服务"
description: "本教程将引导完成使用 Spring Boot 创建简单的 REST 控制器的过程"
authors: Hadi Hariri, Edoardo Vacchi, Sébastien Deleuze，Yue_plus（翻译）
showAuthorInfo: true
source: spring-boot-restful
---
Kotlin 使用 Spring Boot 可以非常顺畅地工作，
并且 Kotlin 可以完全遵循 [Spring 指南](https://spring.io/guides)中创建 RESTful 服务的许多步骤。
但是，在定义 Gradle 配置和项目布局结构以及初始化代码方面存在一些细微的差异。

在本教程中，将逐步完成所需的步骤。
有关 Spring Boot 与 Kotlin 的更详尽说明，请参见[使用 Spring Boot 与 Kotlin 构建 Web 应用程序](https://spring.io/guides/tutorials/spring-boot-kotlin/).

注意，本教程中的所有类都在 `org.jetbrains.kotlin.demo` 包中。

### 定义项目与依赖项
{{ site.text_using_gradle }}

Gradle 文件几乎是标准的 Spring Boot。唯一的区别是 Kotlin 的源文件夹的结构布局、所需的 Kotlin 依赖项与 Gradle 插件：[*kotlin-spring*](https://www.kotlincn.net/docs/reference/compiler-plugins.html#kotlin-spring-compiler-plugi)（以使用 CGLIB 代理为例，`@Configuration` 与 `@Bean` 处理需要 `open` 类）。

<div class="sample" markdown="1" theme="idea" mode="groovy">
``` groovy
buildscript {
    ext.kotlin_version = '{{ site.data.releases.latest.version }}' // Kotlin 集成所需
    ext.spring_boot_version = '2.1.0.RELEASE'
    repositories {
        jcenter()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version" // Kotlin 集成所需
        classpath "org.jetbrains.kotlin:kotlin-allopen:$kotlin_version" // 参见 https://www.kotlincn.net/docs/reference/compiler-plugins.html#spring-support
        classpath "org.springframework.boot:spring-boot-gradle-plugin:$spring_boot_version"
	classpath "io.spring.gradle:dependency-management-plugin:1.0.6.RELEASE"
    }
}

apply plugin: 'kotlin' // Kotlin 集成所需
apply plugin: "kotlin-spring" // https://www.kotlincn.net/docs/reference/compiler-plugins.html#spring-support
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

jar {
    baseName = 'gs-rest-service'
    version = '0.1.0'
}

repositories {
    jcenter()
}

dependencies {
    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version" // Kotlin 集成所需
    compile "org.springframework.boot:spring-boot-starter-web"
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```
</div>

### 创建 Greeting 数据类与控制器
下一步是创建具有两个属性（*id* 与 *content*）的 Greeting 数据类

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
data class Greeting(val id: Long, val content: String)
```
</div>

现在，定义 *GreetingController* ，以 */greeting?name={value}* 的形式接受请求，
并返回表示 *Greeting* 实例的 JSON 对象。

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
@RestController
class GreetingController {

    val counter = AtomicLong()

    @GetMapping("/greeting")
    fun greeting(@RequestParam(value = "name", defaultValue = "World") name: String) =
            Greeting(counter.incrementAndGet(), "Hello, $name")

}
```
</div>

可以看出，这几乎是 Java 到 Kotlin 的一对一转换，对 Kotlin 没有任何特殊要求。

### 创建 Application 类
最后，需要定义一个 Application 类。使用 Kotlin 来定义一个 Spring Boot 所需的公共静态 main 方法。
可以使用 *@JvmStatic* 注解与一个伴生对象来完成，但在这里更推荐使用 Application 类外部定义的[顶级函数]({{ url_for('page', page_path="docs/reference/functions") }})，因为这可以使代码更简洁明了。

无需将 Application 类标记为 *open*，因为 Gradle 插件 *kotlin-spring* 会自动完成。

<div class="sample" markdown="1" theme="idea" data-highlight-only>
``` kotlin
@SpringBootApplication
class Application

fun main(args: Array<String>) {
    SpringApplication.run(Application::class.java, *args)
}
```
</div>

### 运行应用程序
现在可以使用一个 Gradle 标准任务来运行 Spring Boot 应用：

    ./gradlew bootRun

当应用完成编译，资源捆绑并启动，就可以通过浏览器访问了（默认端口为 8080）

![Running App]({{ url_for('tutorial_img', filename='spring-boot-restful/running-app.png')}})

