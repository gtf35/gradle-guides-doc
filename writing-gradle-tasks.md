# 编写自定义 Gradle Tasks

译：基于 Gradle 6.5.1 的文档翻译，翻译于 2020 年 7 月

## 目录

-   [您将要创建什么](#您将要创建什么)
-   [您需要做的准备](#您需要做的准备)
-   [创建一个临时 task](#创建一个临时-task)
-   [添加一个 task 描述](#添加一个-task-描述)
-   [使输出可配置](#使输出可配置)
-   [总结](#总结)
-   [接下来的步骤](#接下来的步骤)
-   [帮助改进这个指南](#帮助改进这个指南)



Task（任务）是 Gradle 完成任务的基石。它们代表构建中的单个原子操作，例如创建一个 JAR 或者链接一个可执行程序。本指南将引导您使用小而精简的 Task 来完成您自定义的构建过程。

## 您将要创建什么

在开始您将会创建一个在控制台打印 *Hello, World* 的临时 Gradle task。然后，您将其配置为可打印任何消息。在此过程中，您将了解临时 task 和自定义 task 类型。

## 您需要做的准备

-   大概 8 分钟
-   一个文本编辑器或 IDE
-   一份 Java Development Kit (JDK)
    -   如果使用 Gradle Groovy DSL 的话需要 JDK 7 或更高
    -   如果使用 Gradle Kotlin DSL 的话需要 JDK 8 或更高
-   Gradle V5.0 或更高版本

## 创建一个临时 task

在一个新的文件夹创建一个 ```build.gradle``` 文件（如果您更喜欢使用Groovy DSL），或者一个 ```build.gradle.kts``` 文件（如果您更喜欢使用Kotlin DSL）然后输入下面的代码

```groovy
// build.gradle
tasks.register("hello") { /**①**/
    doLast { /**②**/
        println 'Hello, World!'
    }
}
```

```kotlin
// build.gradle.kts
tasks.register("hello") { /**①**/
    doLast { /**②**/
        println("Hello, World!")
    }
}
```

① 注册一个叫 `hello` 的临时 task

② 添加一个在控制台打印的 task action 

保存这个文件并且在命令行输入 ```gradle tasks --all``` 。您的新 task 将会在```Other tasks``` 之后出现。

**验证您的任务已创建**

```
$ gradle tasks --all

Other tasks
-----------
hello
```

>   如果您使用的是 Gradle 4.0 或之后的版本，您从控制台看到的输出可能会比本指南中看到的少。指南里的日志是 Gradle 在 ```--console-plain``` 也通过命令行传递给他的情况下输出的。这样做是为了显示 Gradle 正在执行的任务。

运行你的 task

**执行您临时 task 的输出**

```
$ gradle hello

:hello ①
Hello, World ②
```

① 这表示您的 `hello` task 已执行。

② 这是您的临时 task 的输出。

恭喜你！您已经添加了第一个临时 task。

## 添加一个 task 描述

尽管您已经测试了新建的临时 task 并且知道了他是如何工作的，他是一个告诉将要使用我们构建脚本的人我们的 task 是做什么的不错的做法。这对任务进行分类也很有用。

当您运行 ```gradle tasks --all``` 时，早些时候您会在清单中看到其他 task，这些 task 包含描述并根据功能分组。同样（从Gradle 3.3开始），如果一个 task 没有一个组，除非指定了 `--all` ，否则它不会被列出。

**有据可查的任务**

```
Build Setup tasks
-----------------
init - Initializes a new Gradle build.
wrapper - Generates Gradle wrapper files.
```

这是通过在任务上设置组和描述属性来实现的。编辑您的临时 task 并添加以下内容：

```groovy
// build.gradle
tasks.register("hello") {
    group = 'Welcome'
    description = 'Produces a greeting'

    doLast {
        println 'Hello, World'
    }
}
```

```kotlin
// build.gradle.kts
tasks.register("hello") {
    group = "Welcome"
    description = "Produces a greeting"

    doLast {
        println("Hello, World")
    }
}
```

再次运行 ```gradle tasks```。

**'tasks' task 的输出**

```
$ gradle tasks

Welcome tasks
-------------
hello - Produces a greeting
```

干得漂亮!

## 使输出可配置

这是您需要将临时 task 转换为自定义 task 类型的地方，这可以通过在构建脚本中创建一个类来实现。

回到您的构建脚本中并添加以下类并重构您的 `hello` task。

```groovy
// build.gradle
class Greeting extends DefaultTask {  /**①②**/
    String message /**③**/
    String recipient

    @TaskAction /**④**/
    void sayGreeting() {
        println "${message}, ${recipient}!" /**⑤**/
    }
}

tasks.register("hello", Greeting) { /**⑥**/
    group = 'Welcome'
    description = 'Produces a world greeting'
    message = 'Hello' /**⑦**/
    recipient = 'World'
}
```

```kotlin
// build.gradle.kts
open class Greeting: DefaultTask() {  /**①②**/
    lateinit var message: String   /**③**/
    lateinit var recipient: String

    @TaskAction /**④**/
    fun sayGreeting() {
        println("$message, $recipient!") /**⑤**/
    }
}

tasks.register<Greeting>("hello") { /**⑥**/
    group = "Welcome"
    description = "Produces a world greeting"
    message = "Hello" /**⑦**/
    recipient = "World"
}
```

①：`build.gradle` (resp. `build.gradle.kts`) 的构建 DSL 是 Groovy-based DSL (resp. Kotlin-based) ，这个类是 Groovy  (resp. Kotlin) 类

②：尽管可以在特定情况下使用 Gradle API 中的其他 task 类，但是继承 `DefaultTask` 是最常见的情况。

③：添加 `message` 和 `recipient`，可以配置此自定义 task 类型的实例。

④：给 task 的默认 action 添加注解。

⑤：使用标准的 Groovy / Kotlin 输出打印消息。

⑥：使用泛型 `Greeting` 来指定 task 类型。

⑦： 配置 message 和 recipient。

测试您的修改。您应该看到相同的输出。

**转换为自定义任务类型后的输出**

```
$ gradle hello

:hello
Hello, World!
```

现在您完成了自定义 task 类型，您可以添加其他 task。另外创建一个 task 来添加问候语的德语版本。

```groovy
// build.gradle
tasks.register("gutenTag", Greeting) {
    group = 'Welcome'
    description = 'Produces a German greeting'
    message = 'Guten Tag'
    recipient = 'Welt'
}
```

```kotlin
// build.gradle.kts
tasks.register<Greeting>("gutenTag") {
    group = "Welcome"
    description = "Produces a German greeting"
    message = "Guten Tag"
    recipient = "Welt"
}
```

再次运行 `gradle tasks` 以验证是否已添加新任务。

**添加第二个 task 后 'gradle tasks' 的输出。**

```
$ gradle tasks

Welcome tasks
-------------
gutenTag - Produces a German greeting
hello - Produces a world greeting
```

最后，通过执行 `gradle gutenTag` 运行新任务

**第二项任务的输出**。

```
$ gradle gutenTag

:gutenTag
Guten Tag, Welt!
```

## 总结

搞定！您已经通过这些必要的步骤创建了一个自定义的 Gradle Task。你现在应该已经学会了如何

-   注册一个临时的 task 并且通过 `doLast` 添加一个 action。

-   添加一个 task 描述。

-   将临时 task 转换为自定义 Gradle 任务类型并注册 task实例。

-   使用 @TaskAction 设置 task 的默认 action

## 接下来的步骤

-   在构建脚本里面添加类将很快导致混乱且可能无法维护构建脚本。了解如何[组织构建逻辑](https://docs.gradle.org/5.0/userguide/organizing_build_logic.html)。
-   阅读有关[使用 task](https://docs.gradle.org/5.0/userguide/tutorial_using_tasks.html) 以及[预定义 task 和 task 类型的更多信息](https://docs.gradle.org/5.0/dsl/org.gradle.api.Task.html)。

## 帮助改进这个指南

有意见或问题吗？找到错字了？像所有 Gradle 指南一样，寻求帮助只有 GitHub issue 这一种方法。请给  [gradle/guides](https://github.com/gradle/guides/tree/master/subprojects/writing-gradle-tasks) 提 [issue](https://github.com/gradle/guides/issues/new?labels=in:writing-gradle-tasks) 或 pr，我们将尽快与您联系。