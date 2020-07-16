---
type: tutorial
layout: tutorial
title:  "使用 Http Servlet 创建 Web 应用"
description: "本教程通过使用 HttpServlet 创建一个简单的控制器来显示 Hello World。"
authors: Hadi Hariri
showAuthorInfo: false
source: servlet-web-applications
---
Kotlin 可以使用 JavaEE 的 Http Servlet，就像使用其他的 Java 库或者框架一样。我们将看到<!--
-->如何让一个简单的控制器返回 "Hello, World!"。

### 定义项目和依赖关系
{{ site.text_using_gradle }}
The main dependency required for using HTTP servlets is the JavaEE API:

<div class="sample" markdown="1" theme="idea" mode="groovy">

``` groovy
dependencies {
    compile group: 'javax', name: 'javaee-api', version: '7.0'
    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
}
```
</div>

我们还需要 *war* 插件，帮助我们生成相应的构件运行和部署

<div class="sample" markdown="1" theme="idea" mode="groovy">

``` groovy
apply plugin: 'war'
```
</div>

可以在Github检出项目来查看完整的Gradle脚本。


### 创建一个 Home Controller

一旦我们在构建脚本中定义了正确的依赖，现在就可以创建一个控制器

<div class="sample" markdown="1" theme="idea" data-highlight-only>

``` kotlin
@WebServlet(name = "Hello", value = ["/hello"])
class HomeController : HttpServlet() {
    override fun doGet(req: HttpServletRequest, res: HttpServletResponse) {
        res.writer.write("Hello, World!")
    }
}
```
</div>

### 运行程序

使用IntelliJ IDEA，我们可以很容易地运行和调试应用程序，在任何可能的应用程序服务器定义 如Tomcat，Glassfish或WildFly。 在这种情况下我们要使用Tomcat
[曾被定义为一个应用程序服务器在IntelliJ IDEA](http://www.jetbrains.com/idea/webhelp/defining-application-servers-in-intellij-idea.html)。
Note that application server support is only available in IntelliJ IDEA Ultimate.

为了运行，我们需要相应的 WAR(s) 部署。我们可以使用 *war* 生成这些任务，可以很容易地通过在IntelliJ IDEA的Gradle工具执行。


![Gradle Tasks]({{ url_for('tutorial_img', filename='httpservlets/gradle-tasks.png') }})

或者使用命令行构建：

    gradle war

下一步是在IntelliJ IDEA中创建一个运行配置，使用 Tomcat/Local 部署并启动 WAR。

![Run Config]({{ url_for('tutorial_img', filename='httpservlets/tomcat-config.png') }})

一旦我们运行应用程序(使用之前的配置)，成功部署，应该能够使用浏览器打开URL看到如下的响应：

![Browser Run]({{ url_for('tutorial_img', filename='httpservlets/browser.png') }})

We can also run the project from the command line, without using IntelliJ IDEA Ultimate, if we apply the gretty plugin.
In order to do this, we need to make the following changes to build.gradle:

<div class="sample" markdown="1" theme="idea" mode="groovy">

``` groovy
buildscript {
    repositories {
        maven {
            url 'http://oss.jfrog.org/artifactory/oss-snapshot-local/'
        }
        jcenter()
    }
    dependencies {
        classpath 'org.gretty:gretty:3.0.1'
    }
}
...
apply plugin: 'org.gretty'  // Add this line
...

gretty {   // Add these lines
    contextPath = '/'
    servletContainer = 'jetty9'
}

```
</div>

Once we do that, we can start the app by running the following command

```bash
gradle appStart
```

