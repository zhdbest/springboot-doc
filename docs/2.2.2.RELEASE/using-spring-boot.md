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

>[!warning]
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

>[!note]
>
>你只需要为此依赖项指定 Spring Boot 版本号，如果导入其他启动器，则可以放心地省略版本号。

即使使用该设置，你也可以通过覆盖自己项目中的属性来覆盖各个依赖项。例如，要升级到另一个 Spring Data 版本，可以将以下元素添加到`pom.xml`中：

```xml
<properties>
    <spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

>[!tip]
>
>查看[`spring-boot-dependencies` pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml)以获取所支持属性的列表。



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

如上所述，前面的示例设置不允许你使用属性来覆盖个别依赖项。为了获得相同的结果，需要在你项目的`dependencyManagement`中`spring-boot-dependencies`条目之前添加一个条目。例如，要升级到另一个 Spring Data 版本，可以将以下元素添加到`pom.xml`中：

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



### 1.2.3 使用 Spring Boot Maven 插件

Spring Boot 包含一个[Maven插件](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/build-tool-plugins.html#build-tool-plugins-maven-plugin)，可以将项目打包为可执行 jar。 如果要使用插件，请将其添加到你的`<plugins>`部分，如以下示例所示：

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

如果你使用了 Spring Boot 启动器作为父 pom，则只需添加插件。 除非你要更改父 pom 中定义的设置，否则无需对其进行配置。



## 1.3 Gradle

要了解有关将 Spring Boot 与 Gradle 结合使用的信息，请参阅 Spring Boot 的 Gradle 插件的文档：

* 参考（[HTML](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/gradle-plugin/reference/html/) 或 [PDF](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/gradle-plugin/reference/pdf/spring-boot-gradle-plugin-reference.pdf)）
* [API](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/gradle-plugin/reference/api/)



## 1.4 ANT

可以使用 Apache Ant + Ivy 构建 Spring Boot 项目。`spring-boot-antlib`“ AntLib”模块也可用于帮助 Ant 创建可执行 jar。

为了声明依赖关系，典型的`ivy.xml`文件看起来类似于以下示例：

```xml
<ivy-module version="2.0">
    <info organisation="org.springframework.boot" module="spring-boot-sample-ant" />
    <configurations>
        <conf name="compile" description="everything needed to compile this module" />
        <conf name="runtime" extends="compile" description="everything needed to run this module" />
    </configurations>
    <dependencies>
        <dependency org="org.springframework.boot" name="spring-boot-starter"
            rev="${spring-boot.version}" conf="compile" />
    </dependencies>
</ivy-module>
```

典型的`build.xml`看起来类似于一下示例：

```xml
<project
    xmlns:ivy="antlib:org.apache.ivy.ant"
    xmlns:spring-boot="antlib:org.springframework.boot.ant"
    name="myapp" default="build">

    <property name="spring-boot.version" value="2.2.2.RELEASE" />

    <target name="resolve" description="--> retrieve dependencies with ivy">
        <ivy:retrieve pattern="lib/[conf]/[artifact]-[type]-[revision].[ext]" />
    </target>

    <target name="classpaths" depends="resolve">
        <path id="compile.classpath">
            <fileset dir="lib/compile" includes="*.jar" />
        </path>
    </target>

    <target name="init" depends="classpaths">
        <mkdir dir="build/classes" />
    </target>

    <target name="compile" depends="init" description="compile">
        <javac srcdir="src/main/java" destdir="build/classes" classpathref="compile.classpath" />
    </target>

    <target name="build" depends="compile">
        <spring-boot:exejar destfile="build/myapp.jar" classes="build/classes">
            <spring-boot:lib>
                <fileset dir="lib/runtime" />
            </spring-boot:lib>
        </spring-boot:exejar>
    </target>
</project>
```

>[!tip]
>
>如果你不想使用`spring-boot-antlib`模块，请参阅*[howto.html](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto-build-an-executable-archive-with-ant)* “How-to”。



## 1.5 启动器

启动器（“Starters”）是一组可以很方便在应用中使用的依赖描述符。你可以一站式的获取 Spring 及其相关的技术依赖，而不必再去寻找示例代码然后复制粘贴大量依赖描述符。例如，如果要开始使用 Spring 和 JPA 进行数据库访问，可在项目中引入`spring-boot-starter-data-jpa`依赖项。

启动器包含许多你要启动并快速运行一个项目的依赖项，并且依赖之间是相互兼容并可以传递的。

>**如何命名？**
>
>所有官方入门者都遵循类似的命名方式：`spring-boot-starter-*`，其中`*`是应用的具体类型。这种命名结构旨在在你需要寻找启动器时提供帮助。许多 IDE 中的 Maven 集成使你可以按名称搜索依赖项。例如，在合适的 Eclipse 或安装了 STS 插件的情况下，您可以在 POM 编辑页面中按`ctrl-space`并输入“ spring-boot-starter”以获取完整列表。
>
>如“[创建你自己的启动器](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/spring-boot-features.html#boot-features-custom-starter)”部分中所述，第三方启动器命名不应以`spring-boot`开头，因为它是为Spring Boot官方保留的。

Spring Boot 在`org.springframework.boot`组下提供了以下应用程序启动器：

**表1. Spring Boot 应用启动器**

| 名称                                          | 介绍                                                         | Pom                                                          |
| :-------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter`                         | 核心启动器，包括自动配置支持、日志记录和YAML                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter/pom.xml) |
| `spring-boot-starter-activemq`                | 使用 Apache ActiveMQ 的 JMS 消息传递的启动器                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-activemq/pom.xml) |
| `spring-boot-starter-amqp`                    | 使用 Spring AMQP 和 Rabbit MQ 的启动器                       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-amqp/pom.xml) |
| `spring-boot-starter-aop`                     | 使用 Spring AOP 和 AspectJ 进行面向切面编程的启动器          | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-aop/pom.xml) |
| `spring-boot-starter-artemis`                 | 使用 Apache Artemis 的 JMS 消息传递的启动器                  | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-artemis/pom.xml) |
| `spring-boot-starter-batch`                   | 使用 Spring Batch 的启动器                                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-batch/pom.xml) |
| `spring-boot-starter-cache`                   | 使用 Spring Framework 缓存支持的启动器                       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cache/pom.xml) |
| `spring-boot-starter-cloud-connectors`        | 使用 Spring Cloud Connectors 的启动器，可简化与 Cloud Foundry 和 Heroku 等云平台中服务的连接，不推荐使用 Java CFEnv | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cloud-connectors/pom.xml) |
| `spring-boot-starter-data-cassandra`          | 使用 Cassandra 分布式数据库和 Spring Data Cassandra 的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra/pom.xml) |
| `spring-boot-starter-data-cassandra-reactive` | 使用 Cassandra 分布式数据库和 Spring Data Cassandra Reactive 的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra-reactive/pom.xml) |
| `spring-boot-starter-data-couchbase`          | 使用 Couchbase 面向文档的数据库和 Spring Data Couchbase 的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase/pom.xml) |
| `spring-boot-starter-data-couchbase-reactive` | 使用 Couchbase 面向文档的数据库和 Spring Data Couchbase Reactive 的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase-reactive/pom.xml) |
| `spring-boot-starter-data-elasticsearch`      | 使用 Elasticsearch 搜索和分析引擎以及 Spring Data Elasticsearch 的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-elasticsearch/pom.xml) |
| `spring-boot-starter-data-jdbc`               | 使用 Spring Data JDBC 的启动器                               | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jdbc/pom.xml) |
| `spring-boot-starter-data-jpa`                | Spring Data JPA 与 Hibernate 结合使用的启动器                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jpa/pom.xml) |
| `spring-boot-starter-data-ldap`               | 使用 Spring Data LDAP 的启动器                               | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-ldap/pom.xml) |
| `spring-boot-starter-data-mongodb`            | 使用 MongoDB 面向文档的数据库和 Spring Data MongoDB 的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb/pom.xml) |
| `spring-boot-starter-data-mongodb-reactive`   | 使用 MongoDB 面向文档的数据库和 Spring Data MongoDB Reactive 的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb-reactive/pom.xml) |
| `spring-boot-starter-data-neo4j`              | 使用 Neo4j 图形数据库和 Spring Data Neo4j 的启动器           | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-neo4j/pom.xml) |
| `spring-boot-starter-data-redis`              | 使用 Redis 键值数据存储与 Spring Data Redis 和 Lettuce 客户端的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis/pom.xml) |
| `spring-boot-starter-data-redis-reactive`     | 使用 Redis 键值数据存储与 Spring Data Redis Reactive 和 Lettuce 客户端的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis-reactive/pom.xml) |
| `spring-boot-starter-data-rest`               | 使用 Spring Data REST 在 REST 上暴露 Spring Data 存储库的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-rest/pom.xml) |
| `spring-boot-starter-data-solr`               | 使用 Apache Solr 搜索平台和 Spring Data Solr 的启动器        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-solr/pom.xml) |
| `spring-boot-starter-freemarker`              | 使用 FreeMarker 视图构建 MVC web 应用的启动器                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-freemarker/pom.xml) |
| `spring-boot-starter-groovy-templates`        | 使用 Groovy Templates 视图构建MVC web 应用的启动器           | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-groovy-templates/pom.xml) |
| `spring-boot-starter-hateoas`                 | 使用 Spring MVC 和 Spring HATEOAS 构建基于超媒体的 RESTful Web应用程序的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-hateoas/pom.xml) |
| `spring-boot-starter-integration`             | 使用 Spring Integration 的启动器                             | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-integration/pom.xml) |
| `spring-boot-starter-jdbc`                    | 使用 JDBC 和 HikariCP 连接池的启动器                         | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jdbc/pom.xml) |
| `spring-boot-starter-jersey`                  | 使用 JAX-RS 和  Jersey 构建 RESTful web 应用的启动器，[`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-web)的替代方案 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jersey/pom.xml) |
| `spring-boot-starter-jooq`                    | 使用 jOOQ 访问 SQL 数据库的启动器，[`spring-boot-starter-data-jpa`](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-data-jpa) or [`spring-boot-starter-jdbc`](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-jdbc)的替代方案 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jooq/pom.xml) |
| `spring-boot-starter-json`                    | 关于 json 读写的启动器                                       | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-json/pom.xml) |
| `spring-boot-starter-jta-atomikos`            | 使用 Atomikos 的 JTA 事务的启动器                            | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-atomikos/pom.xml) |
| `spring-boot-starter-jta-bitronix`            | 使用 Bitronix 的 JTA 事务的启动器                            | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-bitronix/pom.xml) |
| `spring-boot-starter-mail`                    | 使用 Java Mail 和 Spring Framework 的电子邮件发送支持的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mail/pom.xml) |
| `spring-boot-starter-mustache`                | 使用 Mustache 视图构建 web 应用程序的启动器                  | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mustache/pom.xml) |
| `spring-boot-starter-oauth2-client`           | 使用 Spring Security的 OAuth2/OpenID Connect 客户端功能的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-oauth2-client/pom.xml) |
| `spring-boot-starter-oauth2-resource-server`  | 使用 Spring Security 的 OAuth2 资源服务器功能的启动器        | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-oauth2-resource-server/pom.xml) |
| `spring-boot-starter-quartz`                  | 使用 Quartz 调度器的启动器                                   | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-quartz/pom.xml) |
| `spring-boot-starter-rsocket`                 | 构建 RSocket 客户端和服务端的启动器                          | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-rsocket/pom.xml) |
| `spring-boot-starter-security`                | 使用 Spring Security 的启动器                                | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-security/pom.xml) |
| `spring-boot-starter-test`                    | 测试 Spring Boot 应用的启动器，包括 JUnit、Hamcrest、Mockito | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-test/pom.xml) |
| `spring-boot-starter-thymeleaf`               | 使用 Thymeleaf 视图构建 MVC web 应用的启动器                 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-thymeleaf/pom.xml) |
| `spring-boot-starter-validation`              | 使用 Java Bean Validation 和 Hibernate Validator 的启动器    | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-validation/pom.xml) |
| `spring-boot-starter-web`                     | 使用 Spring MVC 构建 Web（包括RESTful）应用程序的启动器，使用 Tomcat 作为默认的嵌入式容器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web/pom.xml) |
| `spring-boot-starter-web-services`            | 使用 Spring Web Services 的启动器                            | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web-services/pom.xml) |
| `spring-boot-starter-webflux`                 | 使用 Spring Framework 的响应式 Web 支持构建 WebFlux 应用程序的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-webflux/pom.xml) |
| `spring-boot-starter-websocket`               | 使用 Spring Framework 的 WebSocket支持构建 WebSocket 应用程序的启动器 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-websocket/pom.xml) |



除了应用程序启动器，以下启动器可用于添加*[production ready](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/production-ready-features.html#production-ready)*功能：

**表2. Spring Boot 生产启动器**

| 名称                           | 介绍                                                         | Pom                                                          |
| :----------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter-actuator` | 使用 Spring Boot 的 Actuator 的启动器，它提供了production ready 功能，可帮助你监控和管理应用 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-actuator/pom.xml) |



最后，Spring Boot 还包括以下启动器，如果您想排除或替换个别技术层面的启动器，可以使用这些：

**表3. Spring Boot 专业启动器**

| 名称                                | 介绍                                                         | Pom                                                          |
| :---------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `spring-boot-starter-jetty`         | 使用 Jetty 作为嵌入式的 servlet 容器的启动器，[`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-tomcat)的替代方案 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jetty/pom.xml) |
| `spring-boot-starter-log4j2`        | 使用 Log4j2 进行日志记录的启动器，[`spring-boot-starter-logging`](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-logging)的替代方案 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-log4j2/pom.xml) |
| `spring-boot-starter-logging`       | 使用 Logback 进行日志记录的启动器，默认的日志启动器          | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-logging/pom.xml) |
| `spring-boot-starter-reactor-netty` | 使用 Reactor Netty 作为嵌入的响应式 HTTP 服务器的启动器      | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-reactor-netty/pom.xml) |
| `spring-boot-starter-tomcat`        | 使用 Tomcat 作为嵌入式 servlet 容器，[`spring-boot-starter-web`](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-web)使用的默认 servlet 容器启动程序 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-tomcat/pom.xml) |
| `spring-boot-starter-undertow`      | 使用 Undertow 作为嵌入式 servlet 容器的启动器，[`spring-boot-starter-tomcat`](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#spring-boot-starter-tomcat)的替代方案 | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-undertow/pom.xml) |

>[!tip]
>
>有关社区贡献的其他启动器的列表，请参阅 GitHub 上`spring-boot-starters`模块中的[README文件](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/README.adoc)。



# 2. 组织代码

Spring Boot 不需要任何特定的代码编排即可工作。但是，有一些最佳实践会有所帮助。



## 2.1 使用“default”包

当类不包含程序包的声明时，将其视为在“默认程序包”中。通常不建议使用“默认程序包”，应尽可能避免使用。对于使用`@ComponentScan`，`@ConfigurationPropertiesScan`，`@EntityScan`或`@SpringBootApplication`注解的 Spring Boot 应用程序，这可能会导致特定的问题，因为每个 jar 中的每个类都会被读取。

>[!tip]
>
>我们建议您遵循 Java 建议的程序包命名约定，使用反向域名（例如com.example.project）。



## 2.2 放置主应用类

我们通常建议您将主应用程序类放在其他类之上的根目录包中。`@SpringBootApplication`注解通常放在您的主类上，它隐式定义某些类目的基本“搜索包”。例如，如果您正在编写 JPA 应用，则使用`@SpringBootApplication`注解的类的包来搜索`@Entity`的类目。使用根目录包还允许组件扫描仅应用于您的项目。

>[!tip]
>
>如果您不想使用`@SpringBootApplication`，则可以通过导入的`@EnableAutoConfiguration`和`@ComponentScan`注解来定义该行为，因此也可以使用它们来代替`@SpringBootApplication`。

如下是一个典型的布局结构：

```
com
 +- example
     +- myapplication
         +- Application.java
         |
         +- customer
         |   +- Customer.java
         |   +- CustomerController.java
         |   +- CustomerService.java
         |   +- CustomerRepository.java
         |
         +- order
             +- Order.java
             +- OrderController.java
             +- OrderService.java
             +- OrderRepository.java
```

`Application.java`文件要声明`main`方法以及基本的`@SpringBootApplication`，如下所示：

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

























