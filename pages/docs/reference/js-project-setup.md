---
type: doc
layout: reference
category: "JavaScript"
title: "搭建 Kotlin/JS 项目"
---

# 搭建 Kotlin/JS 项目

Kotlin/JS projects use Gradle as a build system. To let developers easily manage their Kotlin/JS projects, we offer
the Kotlin/JS Gradle plugin that provides project configuration tools together with helper tasks for automating routines
typical for JavaScript development. For example, the plugin downloads the [Yarn](https://yarnpkg.com/) package manager
for managing NPM dependencies in background and builds a JavaScript bundle from a Kotlin project using [Webpack](https://webpack.js.org/).

To create a Kotlin/JS project in IntelliJ IDEA, go to **File | New | Project** and select **Gradle | Kotlin/JS for browser**
 or **Kotlin/JS for Node.js**. Be sure to clear the **Java** checkbox.
 
![New project wizard](/assets/images/reference/js-project-setup/wizard.png)


Alternatively, you can apply the `org.jetbrains.kotlin.js` plugin to a Gradle project manually in the `build.gradle` file.
If you use the Gradle Kotlin DSL, you can apply the plugin with `kotlin(“js”)`.


// Groovy DSL
 
```groovy
plugins {
    id 'org.jetbrains.kotlin.js' version '{{ site.data.releases.latest.version }}'
}
```
 


 

// Kotlin DSL
 
```kotlin
plugins {
     kotlin("js") version "{{ site.data.releases.latest.version }}"
}
 ```
 



The Kotlin/JS plugin lets you manage aspects of your project in the `kotlin` section of the build script.



```groovy
kotlin {
    //...
}
```


 
Inside the `kotlin` section, you can manage the following aspects:

* [Target execution environment](#选择执行环境): browser or Node.js 
* [Project dependencies](#管理依赖): Maven and NPM
* [Run configuration](#配置-run-任务)
* [Test configuration](#配置-test-任务)
* [Bundling](#配置-webpack-绑定) for browser projects.

## 选择执行环境

Kotlin/JS projects can target two different execution environments: 

* Browser for client-side scripting in browsers
* [Node.js](https://nodejs.org/) for running JavaScript code outside of a browser, for example, for server-side scripting.

To define the target execution environment for a Kotlin/JS project, add the `target` section with `browser {}` or `nodejs {}` inside.



```groovy
kotlin {
    target {
        browser {
        }       
    }
}    
```



The Kotlin/JS plugin automatically configures its tasks for working with the selected environment.
This includes downloading and installing dependencies required for running and testing the application, and therefore
lets developers  build, run, and test simple projects without additional configuration. 

## 管理依赖

Like any other Gradle projects, Kotlin/JS projects support traditional Gradle [dependency declarations](https://docs.gradle.org/current/userguide/declaring_dependencies.html)
in the `dependencies` section of the build script.


// Groovy DSL

```groovy
dependencies {
    implementation 'org.example.myproject:1.1.0'
}
```





// Kotlin DSL

```kotlin
dependencies {
    implementation("org.example.myproject", "1.1.0")
}
```




The Kotlin/JS Gradle plugin also supports dependency declarations for particular source sets in the `kotlin` section 
of the build script.


// Groovy DSL

```groovy
kotlin {
    sourceSets {
        main {
            dependencies {
                implementation 'org.example.myproject:1.1.0'
            }
        }
    }
}
```





// Kotlin DSL

```kotlin
kotlin {
  sourceSets["main"].dependencies {
    implementation("org.example.myproject", "1.1.0")
  }
}
```





### Kotlin 标准库

The dependency on the Kotlin/JS [standard library](https://kotlinlang.org/api/latest/jvm/stdlib/index.html) is mandatory
for all Kotlin/JS projects. If your project contains tests written in Kotlin, you should also add the dependency on the
[kotlin.test](https://kotlinlang.org/api/latest/kotlin.test/index.html) library.


// Groovy DSL

```groovy
dependencies {
    implementation 'org.jetbrains.kotlin:kotlin-stdlib-js'
    testImplementation 'org.jetbrains.kotlin:kotlin-test-js'
}
```





// Kotlin DSL

```kotlin
dependencies {
    implementation(kotlin("stdlib-js"))
    testImplementation("org.jetbrains.kotlin:kotlin-test-js")
}
```




### NPM 依赖

In the JavaScript world, the common way to manage dependencies is [NPM](https://www.npmjs.com/).
It offers the biggest public [repository](https://www.npmjs.com/)of JavaScript modules and a tool for downloading them.
The Kotlin/JS plugin lets you declare NPM dependencies in the Gradle build script among other dependencies and
does everything else automatically. Particularly, it installs the [Yarn](https://yarnpkg.com/lang/en/) package manager
and uses it to download the dependencies from the NPM repository to the `node_modules` directory of your project -
the common location for NPM dependencies of a JavaScript project. 

To declare an NPM dependency, use the `npm()` function inside the `dependencies` section of a source set.


// Groovy DSL

```groovy
kotlin {
    sourceSets {
        main {
            dependencies {
                implementation 'org.example.myproject:1.1.0'
            }
        }
    }
}
```





// Kotlin DSL

```kotlin
kotlin {
  sourceSets["main"].dependencies {
    implementation(npm("react", "16.8.3"))
  }
}
```




Once an NPM dependency is installed, you can use its API in your code as described in 
[Calling JS from Kotlin](js-interop.html).

## 配置 run 任务

The Kotlin/JS plugin provides a run task that lets you run projects without additional configuration.
For running Kotlin/JS projects, it uses [Webpack DevServer](https://webpack.js.org/configuration/dev-server/).
If you want to customize the DevServer configuration, for example, change its port, use the Webpack configuration file.

To run the project, execute the standard lifecycle `run` task:



```bash
./gradlew run
```



To see the source file changes in browser without restarting the DevServer, use 
the Gradle [continuous build](https://docs.gradle.org/current/userguide/command_line_interface.html#sec:continuous_build):



```bash
./gradlew run --continuous
```



or 



```bash
./gradlew run -t
```



## 配置 test 任务

The Kotin/JS Gradle plugin automatically sets up a test infrastructure for projects. For browser projects, it downloads
and installs the [Karma](https://karma-runner.github.io/) test runner with other required dependencies;
for NodeJS projects, the [Mocha](https://mochajs.org/) test framework is used. 

The plugin also provides useful testing features, for example:

* Source maps generation
* Test reports generation
* Test run results in the console

By default, the plugin uses [Headless Chrome](https://chromium.googlesource.com/chromium/src/+/lkgr/headless/README.md)
for running browser tests. You can also run them in other browsers by adding the corresponding entries inside the
`useKarma` section of the build script:



```groovy
kotlin {
    target {
        browser {
            testTask {
                useKarma {
                    useIe()
                    useSafari()
                    useFirefox()
                    useChrome()
                    useChromeCanary()
                    useChromeHeadless()
                    usePhantomJS()
                    useOpera()
                }
            }
        }
    }
}
```



If you want to skip tests, add the line `enabled = false` to the `testTask`.



```groovy
kotlin {
    target {
        browser {
            testTask {
                enabled = false
            }
        }
    }
}
```



To run tests, execute the standard lifecycle `check` task:



```bash
./gradlew check
```



## 配置 Webpack 绑定

For building executable JavaScript artifacts, the Kotlin/JS plugins contains the `webpackTask`.

To build a project artifact using Webpack, execute the `build` Gradle task:



```bash
./gradlew build
```

