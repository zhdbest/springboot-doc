# 使用 Spring Boot

本节将详细介绍如何使用 Spring Boot，它涵盖了诸如构建系统、自动配置以及如何运行应用程序之类的主题，也包括了一些 Spring Boot 的最佳实践。尽管 Spring Boot 并没有什么特别的地方（它只是另一个可以使用的库），但是有一些建议可以使你的开发过程更轻松一些。

如果你是从 Spring Boot 开始的，那么在进入本节之前，您可能应该阅读一下*[入门指南](docs/2.2.2.RELEASE/getting-started.md)*。



# 1. 构建系统

强烈建议你选择一个支持*[依赖管理](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-dependency-management)*并且可以使用 artifacts 发布到 Maven 中央仓库的构建系统，我们建议你选择 Maven 或 Gradle。也可以其他构建系统（例如 Ant），但是它们并没有得到很好的支持。



## 1.1 依赖管理

每个 Spring Boot 版本都提供了它所支持的依赖关系列表。实际上，你不需要在构建配置中为这些依赖项指定版本，因为 Spring Boot 会为你管理该版本。当你升级 Spring Boot 本身时，这些依赖项也会以一致的方式升级。

>[!note]
>
>如果有需要，你也可以自己指定版本，覆盖 Spring Boot 建议的版本。

这个列表包含可在 Spring Boot 中使用的所有 spring 模块以及完善的第三方库列表。该列表作为可作为标准的[BOM（`spring-boot-dependencies`）](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-maven-without-a-parent)在[Maven](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-maven-parent-pom)和[Gradle](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-gradle)中使用。

>![warning]
>
>Spring Boot 的每个发行版都与 Spring Framework 的基本版本相关联，**强烈**建议你不要自己指定版本。



## 1.2 Maven

Maven 用户可以继承`spring-boot-starter-parent`项目来获得合理的默认值。父项目提供了以下特性：

* Java 1.8作为默认的编译器级别
* UTF-8 编码
* 继承自 spring-boot-dependencies pom 的[依赖管理部分](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-dependency-management)管理公共依赖项的版本。当在自己的 pom 中使用这些依赖关系时，可以为这些依赖关系省去`<version>`标签。
* 使用重新打包执行id对目标进行[重新打包](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/maven-plugin//repackage-mojo.html)。
* 合理的[资源过滤](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html)
* 合理的插件配置（[exec plugin](https://www.mojohaus.org/exec-maven-plugin/), [Git commit ID](https://github.com/ktoso/maven-git-commit-id-plugin), [shade](https://maven.apache.org/plugins/maven-shade-plugin/)）
* 对`application.properties`和`application.yml`进行合理的资源过滤，包括特定的配置文件（例如：`application-dev.properties`、`application-dev.yml`）。

请注意，由于`application.properties`和`application.yml`文件采纳 Spring 风格的占位符（`$ {…}`），因此Maven 过滤已更改为使用`@..@`占位符。 （你可以通过设置一个名为`resource.delimiter`的 Maven 属性来覆盖它）



### 1.2.1 继承父启动器

为了使你的项目继承`spring-boot-starter-parent`，按如下设置`parent`：

```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.2.RELEASE</version>
</parent>
```





### 1.2.2 在不使用父 POM 的情况下使用 Spring Boot

并非每个人都喜欢继承`spring-boot-starter-parent`POM。 你可能需要使用自己公司的标准父级 POM，或者可能希望显式声明所有 Maven 配置。

如果你不想使用`spring-boot-starter-parent`，仍然可以通过使用`scope = import`的依赖项来保留依赖项管理（而不是插件管理）的好处，如下：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.2.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

如上所述，前面的示例设置不允许你使用属性来覆盖个别依赖项。为了获得相同的结果，需要在你项目的`dependencyManagement`中`spring-boot-dependencies`条目之前添加一个条目。例如，要升级到另一个 Spring Data 版本，可以将以下元素添加到pom.xml中：

```xml
<dependencyManagement>
    <dependencies>
        <!-- Override Spring Data release train provided by Spring Boot -->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-releasetrain</artifactId>
            <version>Fowler-SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>2.2.2.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

>[!note]
>
>在前面的示例中，我们指定了*BOM*，可以以相同方式覆盖任何依赖项类型。





















