# 使用 SpringBoot

如果你准备开始使用 SpringBoot 或通常所说的“Spring”，可以从阅读此章节开始。本章节主要回答了“是什么？”、“怎么做？”和“为什么？”的问题。它包含了对 SpringBoot 的介绍以及安装说明。接下来，我们将引导您构建您的第一个 SpringBoot 应用，并在此过程中讨论一些核心原则。



# 1. SpringBoot 介绍

SpringBoot 可以非常简单的创建可独立运行、生产级、基于 Spring 的应用。我们对 Spring 平台和第三方库有自己的看法，这样您就可以轻松入门了。所有的 SpringBoot 应用都仅需要很少的 Spring 配置。



你可以使用 SpringBoot 创建 Java 应用，然后使用`java -jar`运行，或者采取传统的 war 包发布。我们也提供了一个运行“Spring 脚本”的命令行工具。



我们的基本目标是：

* 为所有 Spring 开发提供从根本上更快速、更被广泛理解的入门体验。
* 开箱即用，但需要在需求偏离默认值的时候尽早放弃。

* 提供对大型项目类(如嵌入式服务器、安全性、度量标准、健康检查和外部化配置)通用的一系列非功能特性。
* 完全不需要代码生成，也不需要XML配置。



# 2. 系统要求

SpringBoot 2.2.1.RELEASE 要求至少 Java8 ，并且可以兼容至 Java 13。并且还需要 [Spring Framework 5.2.1.RELEASE](https://docs.spring.io/spring/docs/5.2.1.RELEASE/spring-framework-reference/) 及以上版本。

以下构建工具提供构建支持：

| 构建工具 | 版本                        |
| -------- | --------------------------- |
| Maven    | 3.3+                        |
| Gradle   | 5.x（4.10也支持，但已弃用） |



## 2.1 Servlet 容器

SpringBoot 支持以下嵌入式 Servlet 容器：

| 名称         | Servlet 容器 |
| ------------ | ------------ |
| Tomcat 9.0   | 4.0          |
| Jetty 9.4    | 3.1          |
| Undertow 2.0 | 4.0          |

您也可以将 SpringBoot 应用发布至任意 Servlet3.1+ 的兼容容器上。




# 3. SpringBoot 安装

Spring Boot可以与“经典” Java开发工具一起使用，也可以作为命令行工具安装。无论哪种方式，都需要有 1.8 或更高版本的 Java SDK。你可以使用以下命令检查你当前安装的 Java：

``` bash
$ java -version
```

如果您不熟悉 Java 开发，或者想尝试使用 Spring Boot，则可能要先尝试使用Spring Boot CLI（命令行界面）。 否则，请继续阅读“经典”安装说明。



## 3.1 Java 开发者安装说明

您可以像使用其他标准 Java 库一样使用 SpringBoot。你可以通过引入`spring-boot-*.jar`到你的类路径来使用它。SpringBoot 不需要任何特殊的工具集成，因此您可以使用任何 IDE 或文本编辑器进行开发。同时，SpringBoot 应用也没有什么特殊的，所以你可以像运行、调试其他 Java 程序一样调试、运行 SpringBoot 应用。

虽然你可以直接复制 SpringBoot 的 jar 包，但是我们一般推荐您使用支持依赖管理的构建工具（例如：Maven、Gradle）。



### 3.1.1 Maven 安装

SpringBoot 兼容 Apache Maven 3.3 及以上版本，如果你还没有安装 Maven，可以参考[maven.apache.org](https://maven.apache.org/)的安装说明进行安装。

在一些操作系统上，Maven 可以通过包管理器的方式进行安装。如果你使用的是 macOS，可以运行`brew install maven`。Ubuntu 用户可以使用`sudo apt-get install maven`，Windows 用户可以通过 Chocolatey 从高级的（管理员）提示符下运行`choco install maven`。

SpringBoot 依赖使用的`groupId`为`org.springframework.boot`。通常的，你的 Maven POM 文件继承自`spring-boot-starter-parent`工程，并且声明一个或多个“启动器“的依赖。SpringBoot 也提供了可选的[Maven 插件](https://docs.spring.io/spring-boot/docs/2.2.1.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins-maven-plugin)用于创建可执行 jar 包。

如下是一个典型的 `pom.xml`文件：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <!-- Inherit defaults from Spring Boot -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.1.RELEASE</version>
    </parent>

    <!-- Override inherited settings -->
    <description/>
    <developers>
        <developer/>
    </developers>
    <licenses>
        <license/>
    </licenses>
    <scm>
        <url/>
    </scm>
    <url/>

    <!-- Add typical dependencies for a web application -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <!-- Package as an executable jar -->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

继承`spring-boot-starter-parent`是一个非常棒的使用 SpringBoot 的方式，但可能不适合所有场景。有时，你可能需要继承其他的父 POM，或者你可能不喜欢我们默认的设置，在这种情况下，请参阅[使用 SpringBoot](docs/using-spring-boot.md)，了解使用`import`范围的 POM 替代解决方案。



### 3.1.2 Gradle 安装

SpringBoot 兼容 5.x 版本的 Gradle，虽然 4.0 版本的也支持，但已经被弃用了，并且会在未来的版本中删除。如果你还没有安装 Gradle，你可以参考[gradle.org](https://gradle.org/)的安装说明进行安装。

SpringBoot 依赖使用 `org.springframework.boot` `group`进行声明。通常，你的项目会声明一个或多个“启动器”的依赖，SpringBoot 提供了非常有用的 Gradle 插件，可以用于简化依赖的声明和创建可执行 jar 包。

Gradle Wrapper 提供了一种很棒的方式可以在你需要构建项目的时候“获取”Gradle。这是一个轻量级的脚本和库，您随代码一起提交以引导构建过程。更多细节可以查阅[docs.gradle.org/current/userguide/gradle_wrapper.html](https://docs.gradle.org/current/userguide/gradle_wrapper.html)。

更多关于 SpringBoot 和 Gradle 使用说明的细节，可以参考 Gradle 插件使用说明的[开始]()章节。 



## 3.2 安装 SpringBoot CLI

SpringBoot CLI (Command Line Interface) 是一个可用于快速使用Spring进行原型设计的命令行工具。它使你可以运行具有类似 Java 语法而没有太多样板代码[Groovy](http://groovy-lang.org/)脚本。



# 4. 开发你的第一个 SpringBoot 应用



# 5. 延伸阅读



