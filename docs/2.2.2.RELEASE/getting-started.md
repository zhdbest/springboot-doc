# 入门指南

如果你准备开始使用 Spring Boot 或通常所说的“Spring”，可以从阅读此章节开始。本章节主要回答了“是什么？”、“怎么做？”和“为什么？”的问题。它包含了对 Spring Boot 的介绍以及安装说明。接下来，我们将引导您构建您的第一个 Spring Boot 应用，并在此过程中讨论一些核心原则。



# 1. Spring Boot 介绍

Spring Boot 可以非常简单的创建可独立运行、生产级、基于 Spring 的应用。我们对 Spring 平台和第三方库有自己的看法，这样您就可以轻松入门了。所有的 Spring Boot 应用都仅需要很少的 Spring 配置。



你可以使用 Spring Boot 创建 Java 应用，然后使用`java -jar`运行，或者采取传统的 war 包发布。我们也提供了一个运行“Spring 脚本”的命令行工具。



我们的基本目标是：

* 为所有 Spring 开发提供从根本上更快速、更被广泛理解的入门体验。
* 开箱即用，但需要在需求偏离默认值的时候尽早放弃。

* 提供对大型项目类(如嵌入式服务器、安全性、度量标准、健康检查和外部化配置)通用的一系列非功能特性。
* 完全不需要代码生成，也不需要XML配置。



# 2. 系统要求

Spring Boot 2.2.2.RELEASE 要求至少 Java8 ，并且可以兼容至 Java 13。并且还需要 [Spring Framework 5.2.2.RELEASE](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/) 及以上版本。

以下构建工具提供构建支持：

| 构建工具 | 版本                               |
| -------- | ---------------------------------- |
| Maven    | 3.3+                               |
| Gradle   | 5.x 和 6.x（4.10也支持，但已弃用） |



## 2.1 Servlet 容器

Spring Boot 支持以下嵌入式 Servlet 容器：

| 名称         | Servlet 容器 |
| ------------ | ------------ |
| Tomcat 9.0   | 4.0          |
| Jetty 9.4    | 3.1          |
| Undertow 2.0 | 4.0          |

您也可以将 Spring Boot 应用发布至任意 Servlet3.1+ 的兼容容器上。




# 3. Spring Boot 安装

Spring Boot可以与“经典” Java开发工具一起使用，也可以作为命令行工具安装。无论哪种方式，都需要有 1.8 或更高版本的 Java SDK。你可以使用以下命令检查你当前安装的 Java：

``` bash
$ java -version
```

如果您不熟悉 Java 开发，或者想尝试使用 Spring Boot，则可能要先尝试使用Spring Boot CLI（命令行界面）。 否则，请继续阅读“经典”安装说明。



## 3.1 Java 开发者安装说明

您可以像使用其他标准 Java 库一样使用 Spring Boot。你可以通过引入`spring-boot-*.jar`到你的类路径来使用它。Spring Boot 不需要任何特殊的工具集成，因此您可以使用任何 IDE 或文本编辑器进行开发。同时，Spring Boot 应用也没有什么特殊的，所以你可以像运行、调试其他 Java 程序一样调试、运行 Spring Boot 应用。

虽然你可以直接复制 Spring Boot 的 jar 包，但是我们一般推荐您使用支持依赖管理的构建工具（例如：Maven、Gradle）。



### 3.1.1 Maven 安装

Spring Boot 兼容 Apache Maven 3.3 及以上版本，如果你还没有安装 Maven，可以参考[maven.apache.org](https://maven.apache.org/)的安装说明进行安装。

>[!tip]
>在一些操作系统上，Maven 可以通过包管理器的方式进行安装。如果你使用的是 macOS，可以运行`brew install maven`。Ubuntu 用户可以使用`sudo apt-get install maven`，Windows 用户可以通过 Chocolatey 从高级的（管理员）提示符下运行`choco install maven`。

Spring Boot 依赖使用的`groupId`为`org.springframework.boot`。通常的，你的 Maven POM 文件继承自`spring-boot-starter-parent`工程，并且声明一个或多个“启动器“的依赖。Spring Boot 也提供了可选的[Maven 插件](https://docs.spring.io/spring-boot/docs/2.2.1.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins-maven-plugin)用于创建可执行 jar 包。

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
        <version>2.2.2.RELEASE</version>
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

> [!tip]
>
> 继承`spring-boot-starter-parent`是一个非常棒的使用 Spring Boot 的方式，但可能不适合所有场景。有时，你可能需要继承其他的父 POM，或者你可能不喜欢我们默认的设置，在这种情况下，请参阅[使用 Spring Boot](docs/using-spring-boot.md)，了解使用`import`范围的 POM 替代解决方案。



### 3.1.2 Gradle 安装

Spring Boot 兼容 5.x 和 6.x 版本的 Gradle，虽然 4.0 版本的也支持，但已经被弃用了，并且会在未来的版本中删除。如果你还没有安装 Gradle，你可以参考[gradle.org](https://gradle.org/)的安装说明进行安装。

Spring Boot 依赖使用 `org.springframework.boot` `group`进行声明。通常，你的项目会声明一个或多个“启动器”的依赖，Spring Boot 提供了非常有用的 Gradle 插件，可以用于简化依赖的声明和创建可执行 jar 包。

Gradle Wrapper 提供了一种很棒的方式可以在你需要构建项目的时候“获取”Gradle。这是一个轻量级的脚本和库，您随代码一起提交以引导构建过程。更多细节可以查阅[docs.gradle.org/current/userguide/gradle_wrapper.html](https://docs.gradle.org/current/userguide/gradle_wrapper.html)。

更多关于 Spring Boot 和 Gradle 使用说明的细节，可以参考 Gradle 插件使用说明的[开始]()章节。 



## 3.2 安装 Spring Boot CLI

Spring Boot CLI (Command Line Interface) 是一个可用于快速使用 Spring 进行原型设计的命令行工具。它使你可以运行具有类似 Java 语法而没有太多样板代码的[Groovy](http://groovy-lang.org/)脚本。

你无需使用 CLI 即可使用 Spring Boot，但这绝对是使 Spring 应用程序启动的最快方法。



### 3.2.1 手动安装

你可以从 Spring 软件库下载 Spring CLI 发行版：

* [spring-boot-cli-2.2.2.RELEASE-bin.zip](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.2.2.RELEASE/spring-boot-cli-2.2.2.RELEASE-bin.zip)
* [spring-boot-cli-2.2.2.RELEASE-bin.tar.gz](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.2.2.RELEASE/spring-boot-cli-2.2.2.RELEASE-bin.tar.gz)

也可以选择最新的[快照发行版](https://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/)。

下载完成后，请按照解压缩后的归档文件中的 [INSTALL.txt](https://raw.githubusercontent.com/spring-projects/spring-boot/v2.2.2.RELEASE/spring-boot-project/spring-boot-cli/src/main/content/INSTALL.txt) 说明进行操作。总而言之，`.zip`文件的`bin /`目录中有一个`spring`脚本（对于 Windows 是`spring.bat`）。 或者，你可以将`java -jar`来执行`.jar`文件（脚本可帮助你确保正确设置了类路径）。



### 3.2.2 通过 SDKMAN 安装

SDKMAN（软件开发套件管理器），可用于管理多版本的各种二进制 SDK，包括 Groovy 和 Spring Boot CLI。通过 [sdkman.io](https://sdkman.io/) 获取 SDKMAN，然后可以通过如下命令安装：

```bash
$ sdk install springboot
$ spring --version
Spring Boot v2.2.2.RELEASE
```

如果你为 CLI 开发了新特性并希望轻松访问你所构建的版本，请使用以下命令：

```bash
$ sdk install springboot dev /path/to/spring-boot/spring-boot-cli/target/spring-boot-cli-2.2.2.RELEASE-bin/spring-2.2.2.RELEASE/
$ sdk default springboot dev
$ spring --version
Spring CLI v2.2.2.RELEASE
```

前面的说明将安装 `spring`的本地实例（称为`dev`实例）。它指向你的目标构建位置，因此，每次重新构建 Spring Boot 时，`spring`都是最新的。

你可以通过运行以下命令来查看它：

```bash
$ sdk ls springboot

================================================================================
Available Springboot Versions
================================================================================
> + dev
* 2.2.2.RELEASE

================================================================================
+ - local version
* - installed
> - currently in use
================================================================================
```



### 3.2.3 OSX 通过 Homebrew 安装

在 Mac 上通过[Homebrew](https://brew.sh/)可以使用以下命令安装 Spring Boot CLI：

```bash
$ brew tap pivotal/tap
$ brew install springboot
```

Homebrew 将`spring`安装到`/usr/local/bin`。

>[!note]
>
>如果看不到该公式，则说明 brew 可能已过期。 这时请运行`brew update`然后重试。



### 3.2.4 通过 MacPorts 安装

在 Mac 上通过[MacPorts](https://www.macports.org/)可以使用以下命令安装 Spring Boot CLI：

```bash
$ sudo port install spring-boot-cli
```



### 3.2.5 命令行安装

Spring Boot CLI 提供为[BASH](https://en.wikipedia.org/wiki/Bash_(Unix_shell))和[zsh](https://en.wikipedia.org/wiki/Z_shell) shell 程序提供命令补全的脚本。你可以在任何 shell 中获取脚本（也称为spring），也可以将其放入个人或系统范围的 bash 补全初始化中。在 Debian 系统上，系统级脚本位于`/shell-completion/bash`中，并且在启动新的 Shell 时将执行该目录中的所有脚本。例如，如果你是使用 SDKMAN 安装的，要手动运行脚本，请使用以下命令：

```bash
$ . ~/.sdkman/candidates/springboot/current/shell-completion/bash/spring
$ spring <HIT TAB HERE>
  grab  help  jar  run  test  version
```

>[!note]
>
>如果你是使用 Homebrew 或 MacPorts 安装的 Spring Boot CLI，则命令行补全脚本会自动注册到你的 shell 中。



### 3.2.6 Windows 下通过 Scoop 安装

在 Windows 上通过[Scoop](https://scoop.sh/)可以使用以下命令安装 Spring Boot CLI：

```bash
> scoop bucket add extras
> scoop install springboot
```

Scoop 将`spring`安装到`~/scoop/apps/springboot/current/bin`。

>[!note]
>
>如果没有看到应用清单，则可能是因为 Scoop 已过期。 在这种情况下，请运行`scoop update`然后重试。



### 3.2.7 快速入门Spring CLI示例

你可通过下面的 web 应用检验你的安装，首先创建一个 `app.groovy`，代码如下：

```groovy
@RestController
class ThisWillActuallyRun {

    @RequestMapping("/")
    String home() {
        "Hello World!"
    }

}
```

然后在 shell 中启动它：

``` bash
$ spring run app.groovy
```

>[!note]
>
>第一次启动你的应用会有点慢，因为要下载依赖，后续会变快。

用你喜欢的浏览器打开 [localhost:8080](http://localhost:8080/)，可以看到如下输出：

```
Hello World!
```



## 3.3 从老版本的 Spring Boot 升级

如果你从`1.x`版本的 Spring Boot 升级，可以查阅[Spring Boot 迁移指南](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide)，里面有详细的升级介绍。还请检查“版本说明”以获取每个发行版的“新功能和值得注意的功能”列表。

升级到新版本时，某些属性可能已被重命名或删除。 Spring Boot 提供了一种在启动时分析应用程序环境并打印出检查结果的方法，还可以在运行时为你临时迁移属性。要启用该功能，请将以下依赖项添加到你的项目中：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-properties-migrator</artifactId>
    <scope>runtime</scope>
</dependency>
```

>[!warning]
>
>较晚添加到环境的属性（例如使用`@PropertySource`时）将不被包含在内。

<span></span>

>[!note]
>
>迁移完成后，请确保从项目的依赖项中删除此模块。

要升级现有的 CLI，请使用适当的包管理器命令（例如`brew upgrade`）。 如果你是手动安装的 CLI，请遵循[标准说明](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/getting-started.html#getting-started-manual-cli-installation)，并记住要更新`PATH`环境变量以删除所有较旧的引用。



# 4. 开发你的第一个 Spring Boot 应用

本节介绍如何开发一个简单的“ Hello World！” Web应用程序，该应用程序突出 Spring Boot 的一些关键特性。我们使用 Maven 来构建该项目，因为大多数 IDE 都支持它。

>[!tip]
>
>[spring.io](https://spring.io/) 上包含许多使用 Spring Boot 的[“入门”指南](https://spring.io/guides)。如果你需要解决特定的问题，请首先查阅这些入门指南。
>
>通过转到[start.spring.io](https://start.spring.io/)并从**Dependencies**搜索框中选择“ Web”启动器，可以简化以下步骤。这样做会生成一个新的项目结构，以便你可以立即开始[编码](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/getting-started.html#getting-started-first-application-code)。查看[Spring Initializr 文档](https://docs.spring.io/initializr/docs/current/reference/html//#user-guide)以获取更多详细信息。

在我们开始之前，先打开一个终端，然后执行以下命令以确认你安装的 Java 和 Maven 版本是合适的：

```bash
$ java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
```

```bash
$ mvn -v
Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-17T14:33:14-04:00)
Maven home: /usr/local/Cellar/maven/3.3.9/libexec
Java version: 1.8.0_102, vendor: Oracle Corporation
```

>[!note]
>
>该示例需要在自己的文件夹中创建。 随后的说明中假定你已经创建了一个合适的文件夹，并且它是当前目录。



## 4.1 创建 POM

我们需要先创建一个 Maven 的`pom.xml`文件。`pom.xml`是用于构建项目的。 打开您喜欢的文本编辑器并添加以下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myproject</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.2.RELEASE</version>
    </parent>

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

    <!-- Additional lines to be added here... -->

</project>
```

上面的清单应会为你提供有效的构建。 你可以通过运行`mvn package`对其进行测试（目前，您可以忽略“ jar will be empty - no content was marked for inclusion! ”警告）。

>[!note]
>
>此时，您可以将项目导入IDE（大多数现代 Java IDE 都提供对 Maven的内置支持）。 为简单起见，我们在此示例中继续使用纯文本编辑器。



## 4.2 添加类路径依赖

Spring Boot 提供了一些“启动器”，你可以添加它们的 jar 包到你的类路径下。我们冒烟测试的应用程序使用到了 POM 中`parent`部分的` spring-boot-starter-parent `启动器。`spring-boot-starter-parent`是一个特殊的启动器，提供实用的 Maven 默认值。它还提供了一个[依赖项管理](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-dependency-management)部分，以便你可以忽略其“下属”依赖项的`version`标签。

当你开发一个特殊类型的应用的时候你可能还需要其他启动器提供的依赖。由于我们正在开发 Web 应用程序，因此我们添加了`spring-boot-starter-web`依赖项。在此之前，我们可以通过运行以下命令来查看当前的状态：

```bash
$ mvn dependency:tree

[INFO] com.example:myproject:jar:0.0.1-SNAPSHOT
```

` mvn dependency:tree `命令打印出你的项目依赖的树状表示。你可以看到`spring-boot-starter-parent`本身不提供任何依赖关系。为了添加必需的依赖，请编辑你的`pom.xml`文件并且直接在`parent`部分下面添加` spring-boot-starter-web `的依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

如果你再次执行` mvn dependency:tree `命令，你将可以看到一些新增的依赖，包括 Tomcat web 服务器和 Spring Boot本身。



## 4.3 编写代码

为了完成我们的应用，我们需要创建一个 Java 文件。Maven 默认编译 `src/main/java`中的代码，所以你需要创建这样的文件夹结构，并且添加一个名为`src/main/java/Example.java`的文件，该文件包含以下代码：

```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Example.class, args);
    }

}
```

尽管这里没有太多代码，但是正在发生很多事情。 我们将在接下来的几节中逐步介绍重要部分。



### 4.3.1 `@RestController`和`@RequestMapping`注解

我们的`Example`类中的第一个注解就是`@RestController`，它被称为原型注解。它为人们阅读代码提供了提示，对于Spring来说，被注解的类扮演了特殊角色。在这种情况下，我们的类是一个Web `@Controller`，因此 Spring 在处理传入的 Web 请求时会考虑使用它。

`@RequestMapping`注解提供“路由信息”，它告诉 Spring 任何`/`路径的 HTTP 请求都应该映射到`home`方法上。`@RestController`注解告诉 Spring 将结果字符串直接呈现给调用方。

>[!tip]
>
>`@RestController`和`@RequestMapping`是 Spring MVC 的注解（它们对 Spring Boot 来说没有什么特殊的）。有关更多详细信息，请参见Spring参考文档中的[MVC部分](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web.html#mvc)。



### 4.3.2 `@EnableAutoConfiguration`注解

第二个类级的注解是`@EnableAutoConfiguration`，这个注释告诉 Spring Boot 根据所添加的 jar 依赖关系“猜测”你想如何配置 Spring。由于`spring-boot-starter-web`添加了 Tomcat 和 Spring MVC，因此自动配置会假定你正在开发 Web 应用程序并相应地设置 Spring。

>自动配置旨在与“启动器”配合使用，但是这两个概念并没有直接联系在一起。你可以在启动器之外自由选择jar依赖项。Spring Boot 仍会尽其所能自动配置你的应用程序。



### 4.3.3 “main”方法

我们的应用的最后一部分就是`main`方法，这只是遵循Java约定的应用程序入口的标准方法。我们的`main`方法通过调用`run`委托给 Spring Boot 的`SpringApplication`类。SpringApplication会引导我们的应用程序，并启动Spring，后者反过来又会启动自动配置的Tomcat Web服务器。我们需要将`Example.class`作为参数传递给`run`方法，以告诉`SpringApplication`哪个是主要的Spring组件。`args`数组会将任意公开的命令行参数传递给`run`方法。



## 4.4 运行`Example`

此时，你的应用程序应该可以工作了。由于您使用了`spring-boot-starter-parent` POM，因此你具有一个有用的`run`的目标，可以用来启动该应用程序。从根项目目录键入`mvn spring-boot:run`以启动应用程序。你应该可以看到类似于以下内容的输出：

```
$ mvn spring-boot:run

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.2.2.RELEASE)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.222 seconds (JVM running for 6.514)
```

如果你打开浏览器并访问[localhost:8080](http://localhost:8080/)，你将看到以下输出：

```
Hello World!
```

按下`ctrl-c`可以优雅的退出应用。



## 4.5 生成可执行 Jar

我们通过创建可以在生产环境中运行的完全独立的可执行 jar 文件来结束示例。可执行 jars（有时称为“fat jars”）是包含你已编译的类以及代码需要运行的所有jar依赖项的归档文件。

>**可执行 jars 和 Java**
>
>Java 没有提供加载嵌套 jar文件（jar 中本身包含的 jar 文件）的标准方法。如果你想发布独立的应用程序，则可能会出现问题。
>
>为了解决这个问题，许多开发人员使用“超级”jars。超级 jar 将应用程序内所有依赖项的所有类打包到一个存档中。这种方法的问题在于很难查看应用程序中包含哪些库。如果在多个 jar 中使用相同的文件名（但具有不同的内容），也可能会产生问题。
>
>Spring Boot采用了[另一种方法](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/appendix-executable-jar-format.html#executable-jar)，允许你真正的直接嵌套jar。

为了创建可执行 jar，我们需要添加`spring-boot-maven-plugin`到我们的`pom.xml`文件中，然后把下面的代码添加到`dependencies`部分的下面：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

>[!note]
>
>`spring-boot-starter-parent`的POM包含`<executions>`配置以绑定重新打包的目标。如果你未使用父POM，则需要自己声明此配置。更多详情参考[插件文档](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/maven-plugin//usage.html)。

保存你的`pom.xml`然后在命令行执行`mvn package`，如下：

```bash
$ mvn package

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myproject ---
[INFO] Building jar: /Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:2.2.2.RELEASE:repackage (default) @ myproject ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

在`target`包下，你可以发现`myproject-0.0.1-SNAPSHOT.jar`，这个文件应该在10M左右，如果你想看一下里面的内容，可以执行`jar tvf`，如下：

```bash
$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```

在`target`包下，你也可以发现一个小得多文件叫`myproject-0.0.1-SNAPSHOT.jar.original`，这是 Maven 在Spring Boot 重新打包之前创建的原始 jar 文件。

使用`java -jar`命令启动程序，如下：

```
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::  (v2.2.2.RELEASE)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.536 seconds (JVM running for 2.864)
```

和以前一样，要退出该应用程序，请按ctrl-c。



# 5. 延伸阅读

希望本节提供了一些Spring Boot基础知识，并带你逐步编写自己的应用程序。如果你是一个“任务导向”的开发者，则可能要跳到[spring.io](https://spring.io/)并查看一些入门指南，这些指南可以解决特定的“我该如何使用 Spring？”问题。我们也有特定于 Spring Boot 的“[操作方法](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto)”参考文档。

不然，下一个符合的逻辑步骤是阅读*[using-spring-boot.html](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot)*。 如果您真的不耐烦，还可以继续阅读有关*[Spring Boot功能](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/spring-boot-features.html#boot-features)*的信息。