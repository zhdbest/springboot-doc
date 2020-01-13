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



# 3. 配置类

Spring Boot 支持基于 Java 的配置。尽管可以采用 XML 代码来使用`SpringApplication `，但是我们通常建议您直接使用单个`@Configuration`类。 通常，定义 main 方法的类是首选的`@Configuration`类。

>[!tip]
>
>在网上有许多使用 XML 配置的 Spring 配置示例。如果可以的话，请始终尝试使用等效的基于 Java 的配置。 搜索`Enable*`注解是一个不错的起点。



## 3.1 导入其他配置类

您无需将所有`@Configuration`放在单个类中。`@Import`注解可用于导入其他配置类。或者，您可以使用`@ComponentScan`自动获取所有 Spring 组件，包括`@Configuration`类。



## 3.2 导入 XML 配置

如果必须要使用基于 XML 的配置，我们建议您仍然从`@Configuration`类开始。然后，您可以使用`@ImportResource`注解来加载 XML 配置文件。



# 4. 自动配置

Spring Boot 自动配置会尝试根据你添加的 jar 包依赖项自动配置 Spring 应用程序。例如，如果`HSQLDB`在类路径上，并且您尚未手动配置任何数据库连接 bean，则 Spring Boot 会自动配置内存数据库。

您需要通过将`@EnableAutoConfiguration`或`@SpringBootApplication`注解添加到您的一个`@Configuration`类以加入自动配置。

>[!tip]
>
>您应该只添加一个`@SpringBootApplication`或`@EnableAutoConfiguration`批注。 我们通常建议您仅将一个或另一个添加到您的主`@Configuration`类中。



## 4.1 逐渐取代自动配置

自动配置是非侵入性的。在任何时候，您都可以开始定义自己的配置，以替换自动配置的特定部分。 例如，如果您添加自己的 DataSource bean，则默认的嵌入式数据库支持将被放弃。

如果您需要了解当前正在应用哪些自动配置以及原因，请使用`--debug`开关启动您的应用程序。这样做可以启用调试日志以供选择核心记录器，并将条件报告记录到控制台。



## 4.2 禁用特定的自动配置类

如果发现正在应用不需要的特定自动配置类，则可以使用`@EnableAutoConfiguration`的 exclude 属性禁用它们，如以下示例所示：

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration(proxyBeanMethods = false)
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

如果该类不在类路径中，则可以使用注解的`excludeName`属性，并指定完全限定的名称。最后，您还可以使用`spring.autoconfigure.exclude`属性控制要排除自动配置类的列表。

>[!tip]
>
>您可以在注解级别和使用属性来定义排除项。

<p></p>

>[!note]
>
>尽管自动配置类是公共的，该类的唯一被认为是公共 API 的方面是可用于禁用自动配置的类的名称。这些类的实际内容（例如嵌套配置类或 Bean 方法）仅供内部使用，我们不建议直接使用它们。



# 5. Spring Bean 和依赖注入

您可以自由使用任何标准 Spring Framework 的技术来定义 bean 及其注入的依赖关系。为简单起见，我们发现使用`@ComponentScan`（查找您的bean）和使用`@Autowired`（进行构造函数注入）效果很好。

如果按照上面的建议组织代码（将应用程序类放在根包下），则可以添加`@ComponentScan`，而不添加任何参数。 您的所有应用程序组件（`@Component`，`@Service`，`@Repository`，`@Controller`等）都将自动注册为 Spring Bean。

以下示例展示了一个`@Service` Bean，它使用构造函数注入来获取所需的`RiskAssessor` Bean：

```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    @Autowired
    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

如果 bean 只具有一个构造函数，则可以省略`@Autowired`，如以下示例所示：

```java
@Service
public class DatabaseAccountService implements AccountService {

    private final RiskAssessor riskAssessor;

    public DatabaseAccountService(RiskAssessor riskAssessor) {
        this.riskAssessor = riskAssessor;
    }

    // ...

}
```

>[!tip]
>
>请注意，使用构造函数注入会使得`riskAssessor`字段被标记为`final`，表示其以后无法更改。



# 6. 使用@SpringBootApplication注解

许多 Spring Boot 开发者喜欢在他们的应用程序使用自动配置，组件扫描，并能够在其“应用程序类”上定义额外的配置。单个`@SpringBootApplication`注解可用于启用这三个功能，即：

* `@EnableAutoConfiguration`：启用[Spring Boot的自动配置机制](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-auto-configuration)
* `@ComponentScan`：在应用程序所在的包上启用@Component扫描（请参阅[最佳实践](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-structuring-your-code)）
* `@Configuration`：允许在上下文中注册额外的bean或导入其他配置类

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

>[!note]
>
>`@SpringBootApplication`还提供别名以自定义`@EnableAutoConfiguration`和`@ComponentScan`的属性。

<p></p>

>[!note]
>
>这些功能都不是强制性的，您可以选择用所启用的任何功能替换此单个注解。例如，您可能不想在应用程序中使用组件扫描或配置属性扫描：
>
>```java
>package com.example.myapplication;
>
>import org.springframework.boot.SpringApplication;
>import org.springframework.context.annotation.ComponentScan
>import org.springframework.context.annotation.Configuration;
>import org.springframework.context.annotation.Import;
>
>@Configuration(proxyBeanMethods = false)
>@EnableAutoConfiguration
>@Import({ MyConfig.class, MyAnotherConfig.class })
>public class Application {
>
>    public static void main(String[] args) {
>            SpringApplication.run(Application.class, args);
>    }
>
>}
>```
>
>在此示例中，除了没有自动检测`@Component`所注解的类和`@ConfigurationProperties`所注解的类以及显式导入了用户自定义的 Bean 外，`Application`就像其他任何 Spring Boot 应用程序一样（请参阅`@Import`）。



# 7. 运行你的应用

将应用程序打包为 jar 并使用嵌入式 HTTP 服务器的最大优势之一是，您可以像运行其他应用程序一样运行你的应用。调试Spring Boot应用程序也很容易，您不需要任何特殊的 IDE 插件或扩展。

>[!note]
>
>本节仅介绍 jar 包。如果选择将应用程序打包为 war 文件，则应参考你的服务器和 IDE 文档。



## 7.1 从 IDE 运行

您可以将 IDE 中的 Spring Boot 应用程序作为简单的 Java 应用程序运行。但是，您首先需要导入您的项目。导入步骤因您的 IDE 和构建系统而异。大多数 IDE 都可以直接导入 Maven 项目。例如，Eclipse 用户可以从`File`菜单中选择`Import...`→`Existing Maven Projects`来进行导入。

如果您不能直接将项目导入IDE，则可以使用构建插件生成 IDE 元数据。Maven 包括用于[Eclipse](https://maven.apache.org/plugins/maven-eclipse-plugin/)和[IDEA](https://maven.apache.org/plugins/maven-idea-plugin/)的插件。 Gradle提供了用于[各种 IDE](https://docs.gradle.org/current/userguide/userguide.html)的插件。

>[!tip]
>
>如果不小心运行了两次 Web 应用程序，则会看到“Port already in use”的错误。STS用户可以使用`Relaunch`按钮而不是`Run`按钮来确保关闭任何现有实例。



## 7.2 作为打包的应用程序运行

如果使用 Spring Boot 的 Maven 或 Gradle 插件创建可执行jar，则可以使用`java -jar`运行应用程序，如以下示例所示：

```bash
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

也可以在启用了远程调试支持的情况下运行打包的应用程序。这样做使您可以将调试器附加到打包的应用程序，如以下示例所示：

```bash
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
       -jar target/myapplication-0.0.1-SNAPSHOT.jar
```



## 7.3 使用 Maven 插件

Spring Boot Maven 插件包含一个`run`目标，可用于快速编译和运行您的应用程序。应用程序以exploded形式运行，就像在IDE中一样。以下示例展示了运行 Spring Boot 应用程序的典型 Maven 命令：

```bash
$ mvn spring-boot:run
```

您可能还想使用`MAVEN_OPTS`操作系统环境变量，如以下示例所示：

```bash
$ export MAVEN_OPTS=-Xmx1024m
```



## 7.4 使用 Gradle 插件

Spring Boot Gradle 插件也包含一个`bootRun`任务，该任务可用于以 exploded 形式运行您的应用程序。每当您应用`org.springframework.boot`和`java`插件时，都会添加`bootRun`任务，并在以下示例中显示：

```bash
$ gradle bootRun
```

您可能还想使用`JAVA_OPTS`操作系统环境变量，如以下示例所示：

```bash
$ export JAVA_OPTS=-Xmx1024m
```



## 7.5 热部署

由于 Spring Boot 应用程序只是普通的 Java 应用程序，因此 JVM 热部署应该可以开箱即用。JVM 热部署在一定程度上受到它可以替换的字节码的限制。对于更完整的解决方案，可以使用[JRebel](https://jrebel.com/software/jrebel/)。

`spring-boot-devtools`模块还包括对应用程序快速重启的支持。有关详细信息，请参见本章后面的[开发者工具](using-spring-boot.md#8-开发者工具)部分和[热编译-操作方法](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto-hotswapping)。



# 8. 开发者工具

Spring Boot 包含一组额外的工具，这组工具可以使应用程序开发体验更加愉快。`spring-boot-devtools`模块可以包含在任何项目中，以提供其他开发时功能。要获取 devtools 支持，请将模块依赖项添加到您的构建中，如以下 Maven 和 Gradle 清单所示：

**Maven**

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

**Gradle**

```groovy
configurations {
    developmentOnly
    runtimeClasspath {
        extendsFrom developmentOnly
    }
}
dependencies {
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```

>[!note]
>
>运行完全打包的应用程序时，将自动禁用开发者工具。如果您的应用程序是使用`java -jar`启动的，或者是从特殊的类加载器启动的，则将其视为“生产应用程序”。如果这不适用于您（即，如果您从容器中运行应用程序），请考虑排除 devtools 或设置系统属性`-Dspring.devtools.restart.enabled = false`。

<p></p>

>[!tip]
>
>在 Maven 中将依赖项标记为可选或在 Gradle 中使用自定义`developmentOnly`配置（如上所示）是一种最佳实践，它可以防止将 devtools 传递应用到使用您项目的其他模块。

<p></p>

>[!tip]
>
>重新打包的存档默认情况下不包含 devtools。如果要使用[某个远程 devtools 功能](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-remote)，则需要禁用`excludeDevtools`构建属性以获取该功能。Maven 和 Gradle 插件均支持该属性。



## 8.1 属性默认值

Spring Boot 支持的一些库使用缓存来提高性能。例如，[模板引擎](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-template-engines)会缓存已编译的模板，以避免重复解析模板文件。另外，Spring MVC 可以在提供静态资源时向响应添加 HTTP 缓存的 headers 。

尽管缓存在生产中非常有益，但在开发过程中可能适得其反，从而使您无法看到刚刚在应用程序中所做的更改。因此，默认情况下，spring-boot-devtools 禁用缓存选项。

缓存选项通常由`application.properties`文件中的设置配置。例如，Thymeleaf 提供`spring.thymeleaf.cache`属性。`spring-boot-devtools`模块不需要手动设置这些属性，而是自动应用合理的开发时配置。

由于在开发 Spring MVC 和 Spring WebFlux 应用程序时需要有关Web请求的很多信息，因此开发者工具将为 Web 日志记录组启用 DEBUG 日志记录。这将为您提供有关传入请求、正在处理的处理程序、响应结果等的信息。如果您希望记录所有请求的详细信息（包括潜在的敏感信息），则可以打开`spring.http.log-request-details`配置属性。

>[!note]
>
>如果您不希望应用默认属性，则可以在`application.properties`中将`spring.devtools.add-properties`设置为false。

<p></p>



>[!tip]
>
>有关 devtools 所使用属性的完整列表，请参见[DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)。



## 8.2 自动重启

每当类路径下的文件更改时，使用`spring-boot-devtools`的应用程序都会自动重新启动。在 IDE 中工作时，这可能是一个有用的功能，因为它为代码更改提供了非常快速的反馈。默认情况下，将监视类路径上指向文件夹的任何条目的更改。请注意，某些资源（例如静态资产和视图模板）改动不需要重新启动应用程序。

>**触发重启**
>
>当 DevTools 监视类路径资源时，触发重启的唯一方法是更新类路径。导致类路径更新的方式取决于所使用的 IDE 。在 Eclipse 中，保存修改后的文件将导致类路径被更新并触发重新启动。在 IntelliJ IDEA 中，构建项目（`Build +→+ Build Project`）具有相同的效果。

<p></p>



>[!note]
>
>只要启用了分叉，您还可以使用受支持的构建插件（Maven 和 Gradle）启动应用程序，因为 DevTools 需要隔离的应用程序类加载器才能正常运行。默认情况下，Gradle 和 Maven 插件会分叉应用程序进程。











































