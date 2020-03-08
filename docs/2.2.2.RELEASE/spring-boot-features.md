# Spring Boot 特性

本节将深入介绍 Spring Boot。在这里，您可以了解可能要使用和自定义的关键功能。如果尚未准备好，则可能需要阅读“[ getting-started.html](getting-started.md)”和“ [using-spring-boot.html](using-spring-boot.md)”部分，以便您有足够的基础知识。



# 1. SpringApplication

`SpringApplication`类提供了一种便捷的方式来从`main()`方法启动 Spring 应用程序。在许多情况下，您可以委托给静态`SpringApplication.run`方法，如以下示例所示：

```java
public static void main(String[] args) {
    SpringApplication.run(MySpringConfiguration.class, args);
}
```

当你的应用启动时，你将看到类似如下的输出：

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::   v2.2.2.RELEASE

2019-04-31 13:09:54.117  INFO 56603 --- [           main] o.s.b.s.app.SampleApplication            : Starting SampleApplication v0.1.0 on mycomputer with PID 56603 (/apps/myapp.jar started by pwebb)
2019-04-31 13:09:54.166  INFO 56603 --- [           main] ationConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@6e5a8246: startup date [Wed Jul 31 00:08:16 PDT 2013]; root of context hierarchy
2019-04-01 13:09:56.912  INFO 41370 --- [           main] .t.TomcatServletWebServerFactory : Server initialized with port: 8080
2019-04-01 13:09:57.501  INFO 41370 --- [           main] o.s.b.s.app.SampleApplication            : Started SampleApplication in 2.992 seconds (JVM running for 3.658)
```

默认情况下，显示`INFO`级别的日志消息，包括一些相关的启动详细信息，例如启动应用程序的用户。如果您需要除`INFO`以外的其他日志级别，则可以按照“[日志级别](spring-boot-features.md#44-日志级别)”中的说明进行设置。应用程序版本是通过主应用程序类包中的实现版本来确定的。可以将`spring.main.log-startup-info`设置为`false`来关闭启动信息记录。这还将关闭对应用程序活动配置文件的日志记录。

>[!tip]
>
>要在启动期间添加其他日志记录，可以在`SpringApplication`的子类中重写`logStartupInfo(boolean)`。



## 1.1 启动失败

如果您的应用程序无法启动，则已注册的`FailureAnalyzers`将可以提供专门的错误消息和解决该问题的具体措施。例如，如果您在端口`8080`上启动 Web 应用程序，并且该端口已在使用中，则应该看到类似于以下消息的内容：

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that's listening on port 8080 or configure this application to listen on another port.
```

>[!note]
>
>Spring Boot 提供了众多`FailureAnalyzer`实现，并且您也可以[添加自己的实现](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto-failure-analyzer)。

如果没有故障分析器能够处理该异常，您仍然可以显示完整情况报告，以更好地了解出了什么问题。为此，您需要为`org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener`启用`debug`属性或启用`DEBUG`日志记录。

例如，如果您使用`java -jar`运行应用程序，则可以按照如下所示启用`debug`属性：

```bash
$ java -jar myproject-0.0.1-SNAPSHOT.jar --debug
```



## 1.2 延迟初始化

`SpringApplication`允许延迟初始化应用程序。启用延迟初始化后，将根据需要创建 bean，而不是在应用程序启动期间创建 bean。因此，启用延迟初始化可以减少应用程序启动所花费的时间。在 Web 应用程序中，启用延迟初始化将导致许多与 Web 相关的 Bean 在收到HTTP请求后才被初始化。

延迟初始化的缺点是，它可能会延迟发现应用程序的问题。如果错误配置的 Bean 延迟初始化，则启动期间将不再报错，并且只有在初始化 Bean 时问题才会表现出来。还必须注意确保 JVM 有足够的内存来容纳所有应用程序的bean，而不仅仅是启动期间初始化的 bean。 由于这些原因，默认情况下不会启用延迟初始化，因此建议在启用延迟初始化之前先对 JVM 的堆大小进行微调。

可以使用`SpringApplicationBuilder`中的`lazyInitialization`方法或`SpringApplication`中的`setLazyInitialization`方法以编程的方式启用延迟初始化。或者，可以使用`spring.main.lazy-initialization`属性启用它，如以下示例所示：

```properties
spring.main.lazy-initialization=true
```

>[!tip]
>
>如果要在对应用程序其余部分使用延迟初始化时禁用某些 bean 的延迟初始化，则可以使用`@Lazy(false)`注解将它们的延迟属性显式设置为false。



## 1.3 自定义横幅

可以通过将`banner.txt`文件添加到类路径或将`spring.banner.location`属性设置为此类文件的位置来更改启动时打印的横幅。如果文件的编码不是UTF-8，则可以设置`spring.banner.charset`。 除了文本文件之外，您还可以将`banner.gif`，`banner.jpg`或`banner.png`图像文件添加到类路径中，或设置`spring.banner.image.location`属性。图像将转换为 ASCII 艺术形式并打印在任何文字横幅上方。

在`banner.txt`文件中，您可以使用以下任意占位符：

**表1. 横幅变量**

| 变量                                                         | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `${application.version}`                                     | 您的应用程序的版本号，在`MANIFEST.MF`中声明。例如，`Implementation-Version: 1.0`被打印为`1.0`。 |
| `${application.formatted-version}`                           | 您在`MANIFEST.MF`中声明的应用程序的版本号，其格式用于展示（用括号括起来并以`v`开头），例如`(v1.0)`。 |
| `${spring-boot.version}`                                     | 您使用的 Spring Boot 版本。 例如`2.2.2.RELEASE`。            |
| `${spring-boot.formatted-version}`                           | 您正在使用的 Spring Boot 版本，其格式用于展示（用括号括起来并以`v`开头）。 例如`(v2.2.2.RELEASE)`。 |
| `${Ansi.NAME}` (或`${AnsiColor.NAME}`, `${AnsiBackground.NAME}`, `${AnsiStyle.NAME}`) | 其中`NAME`是 ANSI 转义代码的名称。有关详细信息，请参见[`AnsiPropertySource`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/ansi/AnsiPropertySource.java)。 |
| `${application.title}`                                       | 您在`MANIFEST.MF`中声明的应用程序的标题。例如，`Implementation-Title: MyApp`被打印为MyApp。 |

>[!tip]
>
>如果要以编程方式生成横幅，可以使用`SpringApplication.setBanner(…)`方法。使用`org.springframework.boot.Banner`接口并实现自己的`printBanner()`方法。

您还可以使用`spring.main.banner-mode`属性来确定横幅是否必须在`System.out` (`console`)上打印，是否必须发送到配置的记录器（`log`）或根本不打印（`off`）。

打印的横幅使用以下名称注册为单例 bean：`springBootBanner`。



## 1.4 自定义 SpringApplication

如果`SpringApplication`的默认设置不符合您的喜好，则可以创建一个本地实例并对其进行自定义。例如，要关闭横幅，您可以编写：

```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(MySpringConfiguration.class);
    app.setBannerMode(Banner.Mode.OFF);
    app.run(args);
}
```

>[!note]
>
>传递给`SpringApplication`的构造函数参数是 Spring bean 的配置源。在大多数情况下，它们是对`@Configuration`类的引用，但它们也可以是对 XML 配置或应扫描的程序包的引用。

也可以通过使用`application.properties`文件配置`SpringApplication`。 有关详细信息，请参见[外部化配置](spring-boot-features.md#2-外部化配置)。

有关配置选项的完整列表，请参见[`SpringApplication` Javadoc](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/api//org/springframework/boot/SpringApplication.html)。



## 1.5 快速构建器 API

如果您需要构建`ApplicationContext`层次结构（具有父/子关系的多个上下文），或者如果您更喜欢使用“流利的”构建器API，则可以使用`SpringApplicationBuilder`。

`SpringApplicationBuilder`允许您将多个方法调用链接在一起，并允许您创建层次结构的`parent`方法和`child`方法，如以下示例所示：

```java
new SpringApplicationBuilder()
        .sources(Parent.class)
        .child(Application.class)
        .bannerMode(Banner.Mode.OFF)
        .run(args);
```

>[!note]
>
>创建`ApplicationContext`层次结构时有一些限制。例如，Web 组件**必须**包含在子上下文中，并且相同的`Environment`用于父上下文和子上下文。 有关完整的详细信息，请参见[`SpringApplicationBuilder ` Javadoc](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/api//org/springframework/boot/builder/SpringApplicationBuilder.html)。



## 1.6 应用事件和监听器

除了通常的 Spring Framework 事件（例如`ContextRefreshedEvent`）之外，`SpringApplication`还发送一些其他应用程序事件。

>[!note]
>
>一些事件实际上是在`ApplicationContext`被创建之前触发，因此您不能将这些事件注册为`@Bean`。您可以使用`SpringApplication.addListeners(…)`方法或`SpringApplicationBuilder.listeners(…)`方法注册它们。
>
>如果希望这些监听器自动注册，而不管创建应用程序的方式如何，都可以将`META-INF/spring.factories`文件添加到您的项目中，并使用`org.springframework.context.ApplicationListener`键引用您的监听器（们），如以下示例所示：
>
>```properties
>org.springframework.context.ApplicationListener=com.example.project.MyListener
>```

应用事件在您的应用程序运行时按以下顺序发送：

1. `ApplicationStartingEvent`在运行开始时但未进行任何处理之前（侦听器和初始化程序的注册除外）发送。
2. `ApplicationEnvironmentPreparedEvent`在当已知`Environment`要在上下文中使用但创建上下文之前发送。
3. `ApplicationContextInitializedEvent`在`ApplicationContext`准备好并且`ApplicationContextInitializers`被调用但未加载任何 bean 定义时发送。
4. `ApplicationPreparedEvent`在刷新开始之前但加载 bean 定义之后发送。
5. `ApplicationStartedEvent`在刷新上下文之后但在调用任何应用程序和命令行运行程序之前发送。
6. `ApplicationReadyEvent`在调用任何应用程序和命令行运行程序之后发送。它表明该应用程序已准备就绪，可以处理请求。
7. `ApplicationFailedEvent`在启动时若发生异常则发送。

上面的列表仅包含绑定到`SpringApplication`的`SpringApplicationEvent`。除这些以外，以下事件也在`ApplicationPreparedEvent`之后和`ApplicationStartedEvent`之前发布：

1. `ContextRefreshedEvent`在刷新`ApplicationContext`时发送。
2. `WebServerInitializedEvent`在`WebServer`准备就绪后发送。`ServletWebServerInitializedEvent`和`ReactiveWebServerInitializedEvent`分别是servlet和响应式变量。

>[!tip]
>
>您通常不需要使用应用程序事件，但是很容易知道它们的存在。在内部，Spring Boot 使用事件来处理各种任务。

应用事件是通过使用 Spring Framework 的事件发布机制发送的。此机制的一部分确保在子级上下文中发布给监听器的事件也可以在任何祖先上下文中发布给监听器。这导致，如果您的应用程序使用`SpringApplication`实例的层次结构，则监听器可能会收到同一类型的应用程序事件的多个实例。

为了使您的监听器能够区分其上下文的事件和后代上下文的事件，它应请求其注入的应用程序上下文，然后将注入的上下文与事件的上下文进行比较。可以通过实现`ApplicationContextAware`来注入上下文，或者，如果监听器是 bean，则可以使用`@Autowired`注入上下文。



## 1.7 Web 环境

`SpringApplication`尝试代表您创建正确的`ApplicationContext`类型。用于确定`WebApplicationType`的算法非常简单：

* 如果存在 Spring MVC，则使用`AnnotationConfigServletWebServerApplicationContext`
* 如果不存在 Spring MVC 并且存在 Spring WebFlux，则使用`AnnotationConfigReactiveWebServerApplicationContext`
* 否则，将使用`AnnotationConfigApplicationContext`

这意味着，如果您在同一应用程序中使用 Spring MVC 和 Spring WebFlux 中的新`WebClient`，则默认情况下将使用 Spring MVC。您可以通过调用`setWebApplicationType(WebApplicationType)`轻松覆盖它。

也可以完全控制用于调用`setApplicationContextClass(…)`使用的`ApplicationContext`类型。

>[!tip]
>
>在 JUnit 测试中使用`SpringApplication`时，通常希望调用`setWebApplicationType(WebApplicationType.NONE)`。



## 1.8 获取应用程序参数

如果您需要获取传递给` SpringApplication.run(…) `的应用程序参数，则可以注入`org.springframework.boot.ApplicationArguments` bean。`ApplicationArguments`接口提供对原始`String[]`参数以及已解析的` option `和` non-option `参数的访问，如以下示例所示：

```java
import org.springframework.boot.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.stereotype.*;

@Component
public class MyBean {

    @Autowired
    public MyBean(ApplicationArguments args) {
        boolean debug = args.containsOption("debug");
        List<String> files = args.getNonOptionArgs();
        // if run with "--debug logfile.txt" debug=true, files=["logfile.txt"]
    }

}
```

>[!tip]
>
>Spring Boot 还向Spring 环境注册了`CommandLinePropertySource`。这样，您还可以使用`@Value`注解注入单个应用程序参数。



## 1.9 使用ApplicationRunner或CommandLineRunner

如果在`SpringApplication`启动后需要运行某些特定的代码，则可以实现`ApplicationRunner`或`CommandLineRunner`接口。两个接口都以相同的方式工作并提供一个`run`方法，该方法在` SpringApplication.run(…) `完成之前被调用。

`CommandLineRunner`接口以简单的字符串数组提供对应用程序参数的访问，而`ApplicationRunner`使用前面讨论的`ApplicationArguments`接口。以下示例显示了带有`run`方法的`CommandLineRunner`：

```java
import org.springframework.boot.*;
import org.springframework.stereotype.*;

@Component
public class MyBean implements CommandLineRunner {

    public void run(String... args) {
        // Do something...
    }

}
```

如果定义了几个必须按特定顺序调用的`CommandLineRunner`或`ApplicationRunner` Bean，则可以另外实现`org.springframework.core.Ordered`接口或使用`org.springframework.core.annotation.Order`注解。



## 1.10 应用退出

每个`SpringApplication`会向 JVM 注册一个关闭钩子，以确保`ApplicationContext`在退出时正常关闭。所有标准的 Spring 生命周期回调（例如`DisposableBean`接口或`@PreDestroy`注解）都可以使用。

另外，如果bean希望在调用`SpringApplication.exit()`时返回特定的退出码，则可以实现`org.springframework.boot.ExitCodeGenerator`接口。然后可以将此退出码传递给`System.exit()`，以将其作为状态码返回，如以下示例所示：

```java
@SpringBootApplication
public class ExitCodeApplication {

    @Bean
    public ExitCodeGenerator exitCodeGenerator() {
        return () -> 42;
    }

    public static void main(String[] args) {
        System.exit(SpringApplication.exit(SpringApplication.run(ExitCodeApplication.class, args)));
    }

}
```

另外，`ExitCodeGenerator`接口可以由异常实现。遇到此类异常时，Spring Boot 返回实现的`getExitCode()`方法所提供的退出码。



## 1.11 管理员功能

通过指定`spring.application.admin.enabled`属性，可以为应用启用与管理员相关的功能。这将在平台`MBeanServer`上公开[`SpringApplicationAdminMXBean`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/admin/SpringApplicationAdminMXBean.java)。您可以使用此功能来远程管理 Spring Boot 应用。对于任何服务包装器实现，此功能也可能很有用。

>[!tip]
>
>如果您想知道应用程序在哪个HTTP端口上运行，请使用`local.server.port`键获取属性。



# 2. 外部化配置

Spring Boot 使您可以使配置外部化，以便可以在不同环境中使用相同的应用程序代码。您可以使用属性文件、YAML 文件、环境变量和命令行参数来外部化配置。属性值可以使用`@Value`注解直接注入到您的 bean 中，可以通过 Spring 的`Environment`抽象访问，也可以通过`@ConfigurationProperties`[绑定到结构化对象](spring-boot-features.md#28-类型安全的配置属性)。

Spring Boot 使用一个非常特殊的`PropertySource`顺序，该顺序旨在允许合理地覆盖值。顺序如下：

1. 当 devtools 处于可用状态时，`$HOME/.config/spring-boot`文件夹中的 [Devtools 全局设置属性](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-devtools-globalsettings)
2. 测试用例上的[`@TestPropertySource`](https://docs.spring.io/spring/docs/5.2.2.RELEASE/javadoc-api/org/springframework/test/context/TestPropertySource.html)注解
3. 测试用例中的`properties`属性。 在`@SpringBootTest`和测试注解上可用，用于测试应用程序的特定部分
4. 命令行参数
5. 来自`SPRING_APPLICATION_JSON`（嵌入在环境变量或系统属性中的内联JSON）的属性。
6. `ServletConfig`初始化参数
7. `ServletContext`初始化参数
8. 来自`java:comp/env`的 JNDI 属性
9. Java 系统配置（`System.getProperties()`）
10. 操作系统环境变量
11. 仅在`random.*`中具有属性的`RandomValuePropertySource`
12. 没有打进 jar 包的[Profile-specific 应用属性](spring-boot-features.md#24-profile-specific-属性)（`application-{profile}.properties`和 YAML 变量）
13. 打进 jar 包的[Profile-specific 应用属性](spring-boot-features.md#24-profile-specific-属性)（`application-{profile}.properties`和 YAML 变量）
14. 没有打进 jar 包的应用属性 （`application.properties` 和 YAML 变量）
15. 打进 jar 包的应用属性 （`application.properties` 和 YAML 变量）
16. `@Configuration`类上的`@PropertySource`注解。请注意，在刷新应用程序上下文之前，不会将此类属性源添加到`Environment`中。现在配置某些属性（如`logging.*`和`spring.main.*`）为时已晚，这些属性在刷新开始之前就已读取。
17. 默认属性（通过设置`SpringApplication.setDefaultProperties`指定）



为了提供一个具体的示例，假设您开发了一个使用`name`属性的`@Component`，如以下示例所示：

```java
import org.springframework.stereotype.*;
import org.springframework.beans.factory.annotation.*;

@Component
public class MyBean {

    @Value("${name}")
    private String name;

    // ...

}
```

在您的应用类路径上（例如，在jar内），您可以拥有一个`application.properties`文件，该文件为`name`提供合理的默认属性值。在新环境中运行时，可以在 jar 外部提供一个覆盖`name`的`application.properties`文件。对于一次性测试，可以使用特定的命令行开关启动（例如：`java -jar app.jar --name="Spring"`）。

>[!tip]
>
>可以在命令行中使用环境变量来提供`SPRING_APPLICATION_JSON`属性。例如，您可以在UN * X shell 中使用以下行：
>
>```shell
>$ SPRING_APPLICATION_JSON='{"acme":{"name":"test"}}' java -jar myapp.jar
>```
>
>在前面的示例中，您最终在Spring `Environment`中设置了`acme.name = test`。您还可以在系统属性中将JSON 作为`spring.application.json`提供，如以下示例所示：
>
>```shell
>$ java -Dspring.application.json='{"name":"test"}' -jar myapp.jar
>```
>
>你也可以通过命令行参数提供 JSON，如以下所示：
>
>```shell
>$ java -jar myapp.jar --spring.application.json='{"name":"test"}'
>```
>
>您还可以将 JSON 作为 JNDI 变量提供，如下所示： `java:comp/env/spring.application.json`



## 2.1 配置随机值

`RandomValuePropertySource`可用于注入随机值（例如，注入秘码或测试用例）。它可以产生 integers，longs，uuid 或字符串，如以下示例所示：

```properties
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

`random.int*`语法是`OPEN value (,max) CLOSE`，其中`OPEN,CLOSE`可以是任何字符，`value,max`是整数。如果提供了`max`，则`value`是最小值，而`max`是最大值（不包括`max`）。



## 2.2 访问命令行属性

默认情况下，`SpringApplication`将所有命令行选项参数（即以`--`开头的参数，例如`--server.port=9000`）转换为`property`，并将其添加到Spring `Environment`中。如前所述，命令行属性始终优先于其他属性源。

如果您不希望将命令行属性添加到`Environment`中，则可以使用`SpringApplication.setAddCommandLineProperties(false)`禁用它们。



## 2.3 应用属性文件

`SpringApplication`在以下位置从`application.properties`文件加载属性，并将它们添加到Spring `Environment`：

1. 当前目录的`/config`子目录
2. 当前目录
3. 类路径`/config`包
4. 类路径根目录

该列表按优先级排序（在列表较高位置定义的属性会覆盖在较低位置定义的属性）。

>[!note]
>
>您还可以使用YAML（.yml）文件来替代 .properties。

如果您不喜欢`application.properties`作为配置文件名，则可以通过指定`spring.config.name`环境属性来切换到另一个文件名。您还可以通过使用`spring.config.location`环境属性（这是目录位置或文件路径的逗号分隔列表）来引用显式位置。以下示例展示如何指定其他文件名：

```shell
$ java -jar myproject.jar --spring.config.name=myproject
```

下面的示例演示如何指定两个位置：

```shell
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
```

>[!warning]
>
>`spring.config.name`和`spring.config.location`很早就用于确定必须加载的文件。必须将它们定义为环境属性（通常是OS环境变量，系统属性或命令行参数）。

如果`spring.config.location`包含目录（而不是文件），则它们应以`/`结尾（并且在运行时，会在加载之前附加从`spring.config.name`生成的名称，包括特定配置文件的文件名）。`spring.config.location`中指定的文件按原样使用，不支持特定配置文件的变体，并且被任何特定配置文件的属性覆盖。

配置位置以相反的顺序检索。默认情况下，配置的位置是`classpath:/,classpath:/config /,file:./,file:./config/`。 结果检索顺序如下：

1. `file:./config/`
2. `file:./`
3. `classpath:/config/`
4. `classpath:/`

使用`spring.config.location`配置自定义配置位置后，它们将取代默认位置。例如，如果将`spring.config.location`的值配置为`classpath:/custom-config/,file:./custom-config/`，则搜索顺序如下：

1. `file:./custom-config/`
2. `classpath:custom-config/`

另外，当使用`spring.config.additional-location`配置自定义配置的位置时，除默认位置外，还会使用这些自定义配置。附加位置在默认位置之前被检索。例如，如果配置了`classpath:/custom-config/,file:./custom-config/`的附加位置，则检索顺序如下：

1. `file:./custom-config/`
2. `classpath:custom-config/`
3. `file:./config/`
4. `file:./`
5. `classpath:/config/`
6. `classpath:/`

通过此检索顺序，您可以在一个配置文件中指定默认值，然后在另一个配置文件中有选择地覆盖这些值。您可以在默认位置之一的`application.properties`（或使用`spring.config.name`选择的其他任何基本名称）中为应用程序提供默认值。然后，可以在运行时使用任一自定义位置中的其他文件覆盖这些默认值。

>[!note]
>
>如果使用环境变量而不是系统属性，则大多数操作系统不允许使用句点分隔的键名，但是可以使用下划线代替（例如，使用`SPRING_CONFIG_NAME`代替`spring.config.name`）。

<span></span>



>[!note]
>
>如果您的应用程序在容器中运行，则可以使用 JNDI 属性（在`java:comp/env`中）或 servlet 上下文初始化参数代替环境变量或系统属性，或者也可以使用环境变量或系统属性。



## 2.4 Profile-specific 属性





## 2.8 类型安全的配置属性





## 4.4 日志级别











# 10 使用 SQL 数据库

[Spring 框架](https://spring.io/projects/spring-framework)为使用 SQL 数据库提供了广泛的支持，从使用 JdbcTemplate 的直接 JDBC 访问到完整的“对象关系映射”技术（例如Hibernate）。[Spring Data](https://spring.io/projects/spring-data)提供了更高级别的功能：直接从接口创建存储库实现，并使用约定从您的方法名称生成查询语句。



## 10.1 配置数据源

Java 的`javax.sql.DataSource`接口提供了使用数据库连接的标准方法。传统上，“数据源”使用 URL 和一些凭据来建立数据库连接。

>[!tip]
>
>有关更多高级示例，请参见[“操作方法”部分](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto-configure-a-datasource)，通常可以完全控制 DataSource 的配置。



### 10.1.1 嵌入式数据库支持

通过使用内存嵌入式数据库来开发应用程序通常很方便。显然，内存数据库不提供持久存储。您需要在应用程序启动时填充数据库，并准备在应用程序结束时丢弃数据。

>[!tip]
>
>“操作方法”部分包括有关[如何初始化数据库的部分](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto-database-initialization)。

Spring Boot 可以自动配置嵌入式[H2](https://www.h2database.com/)，[HSQL](http://hsqldb.org/)和[Derby](https://db.apache.org/derby/)数据库。您无需提供任何连接URL。您只需要引入要使用的嵌入式数据库的构建依赖项即可。

>[!note]
>
>如果您在测试中使用此功能，则可能会注意到，无论你使用多少应用程序上下文，整个测试套件都会重复使用同一数据库。如果要确保每个上下文都有一个单独的嵌入式数据库，则应将`spring.datasource.generate-unique-name`设置为`true`。

例如，典型的 POM 依赖关系如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>org.hsqldb</groupId>
    <artifactId>hsqldb</artifactId>
    <scope>runtime</scope>
</dependency>
```

>[!note]
>
>您需要依赖`spring-jdbc`来自动配置嵌入式数据库。在此示例中，它通过`spring-boot-starter-data-jpa`可传递地引入。

<span></span>



>[!tip]
>
>如果出于某种原因为嵌入式数据库配置了连接 URL，请务必确保禁用了数据库的自动关闭功能。 如果使用H2，则应使用`DB_CLOSE_ON_EXIT=FALSE`进行操作。如果使用 HSQLDB，则应确保未使用`shutdown=true`。 通过禁用数据库的自动关闭功能，Spring Boot 可以控制何时关闭数据库，从而确保一旦不再需要访问数据库时就可以执行该操作。



### 10.1.2 连接到生产数据库

生产数据库连接也可以通过使用池化`DataSource`进行自动配置。 Spring Boot使用以下算法来选择特定的实现：

1. 我们更喜欢[HikariCP](https://github.com/brettwooldridge/HikariCP)的性能和并发性。 如果有 HikariCP，我们总是选择它。
2. 否则，如果 Tomcat 池化`DataSource`可用，我们将使用它。
3. 如果 HikariCP 和 Tomcat 池化数据源均不可用，并且[Commons DBCP2](https://commons.apache.org/proper/commons-dbcp/)可用，我们将使用 Commons DBCP2。

如果您使用`spring-boot-starter-jdbc`或`spring-boot-starter-data-jpa`“启动器”，则会自动获得对 HikariCP 的依赖。

>[!note]
>
>您可以通过设置`spring.datasource.type`属性来完全绕过该算法，并指定要使用的连接池。如果您在 Tomcat 容器中运行应用程序，则这一点尤其重要，因为默认情况下提供了`tomcat-jdbc`。

<span></span>



>[!tip]
>
>其他连接池始终可以手动配置。 如果定义自己的`DataSource` bean，则不会进行自动配置。



数据源配置由`spring.datasource.*`中的外部配置属性控制。例如，您可以在`application.properties`中声明以下部分：

```properties
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

>[!note]
>
>您至少应通过设置`spring.datasource.url`属性来指定 URL。否则，Spring Boot 会尝试自动配置嵌入式数据库。

<span></span>



>[!tip]
>
>您通常不需要指定`driver-class-name`，因为 Spring Boot 可以从`url`为大多数数据库推断出它。

<span></span>



>[!note]
>
>对于要创建池化数据源，我们需要能够验证有效的`Driver`类是否可用，因此我们在进行任何操作之前都要进行检查。换句话说，如果设置`spring.datasource.driver-class-name=com.mysql.jdbc.Driver`，则该类必须是可加载的。



有关更多支持的选项，请参见`DataSourceProperties`，这些是不管实际实现如何都起作用的标准选项。也可以使用它们各自的前缀（`spring.datasource.hikari.*`，`spring.datasource.tomcat.*`和`spring.datasource.dbcp2.*`）微调实现特定的设置。有关更多详细信息，请参考所用连接池实现的文档。

例如，如果使用[Tomcat连接池](https://tomcat.apache.org/tomcat-8.0-doc/jdbc-pool.html#Common_Attributes)，则可以自定义一些其他设置，如以下示例所示：

```properties
# Number of ms to wait before throwing an exception if no connection is available.
spring.datasource.tomcat.max-wait=10000

# Maximum number of active connections that can be allocated from this pool at the same time.
spring.datasource.tomcat.max-active=50

# Validate the connection before borrowing it from the pool.
spring.datasource.tomcat.test-on-borrow=true
```



### 10.1.3 连接到 JNDI 数据源

如果您将 Spring Boot 应用程序部署到应用服务器，则可能要使用应用服务器的内置功能来配置和管理数据源，并使用 JNDI 对其进行访问。

`spring.datasource.jndi-name`属性可以用作`spring.datasource.url`，`spring.datasource.username`和`spring.datasource.password`属性的替代方案，以从特定的 JNDI 位置访问`DataSource`。例如，`application.properties`中的以下部分显示了如何访问 JBoss AS 定义的数据源：

```properties
spring.datasource.jndi-name=java:jboss/datasources/customers
```



## 10.2 使用 JdbcTemplate

Spring 的`JdbcTemplate`和`NamedParameterJdbcTemplate`类是自动配置的，您可以将它们直接使用`@Autowire`注入到自己的 bean 中，如以下示例所示：

``` java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final JdbcTemplate jdbcTemplate;

    @Autowired
    public MyBean(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    // ...

}
```



您可以使用`spring.jdbc.template.*`属性来自定义模板的某些属性，如以下示例所示：

```properties
spring.jdbc.template.max-rows=500
```

>[!note]
>
>`NamedParameterJdbcTemplate`在后台重用相同的`JdbcTemplate`实例。如果定义了多个`JdbcTemplate`并且不存在主要候选对象，则不会自动配置`NamedParameterJdbcTemplate`。



## 10.3 JPA 和 Spring Data JPA

Java Persistence API 是一种标准技术，可让您将对象“映射”到关系型数据库。`spring-boot-starter-data-jpa` POM 提供了一种快速开始的方法。它提供以下关键依赖：

* Hibernate：最流行的 JPA 实现之一
* Spring Data JPA：让实现基于 JPA 的存储库变得容易
* Spring ORM：Spring Framework 提供的核心 ORM 支持

>[!tip]
>
>在这里，我们不会过多讨论 JPA 或 [Spring Data](https://spring.io/projects/spring-data)。您可以查阅[spring.io](https://spring.io/)的[“使用 JPA 访问数据”](https://spring.io/guides/gs/accessing-data-jpa/)指南，并阅读[Spring Data JPA](https://spring.io/projects/spring-data-jpa)和[Hibernate](https://hibernate.org/orm/documentation/)参考文档。



### 10.3.1 实体类

传统上的 JPA “实体”类在`persistence.xml`文件中指定。在 Spring Boot 中，此文件不是必需的，而是使用“实体扫描”。默认情况下，将搜索主配置类（使用`@EnableAutoConfiguration`或`@SpringBootApplication`注解的）下的所有包。

任何带有`@Entity`，`@Embeddable`或`@MappedSuperclass`注解的类都将被扫描。典型的实体类类似于以下示例：

```java
package com.example.myapp.domain;

import java.io.Serializable;
import javax.persistence.*;

@Entity
public class City implements Serializable {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private String state;

    // ... additional members, often include @OneToMany mappings

    protected City() {
        // no-args constructor required by JPA spec
        // this one is protected since it shouldn't be used directly
    }

    public City(String name, String state) {
        this.name = name;
        this.state = state;
    }

    public String getName() {
        return this.name;
    }

    public String getState() {
        return this.state;
    }

    // ... etc

}
```

>[!tip]
>
>您可以使用`@EntityScan`注解来自定义实体扫描位置。请参阅“ [howto.html](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto-separate-entity-definitions-from-spring-configuration)”。



### 10.3.2 Spring Data JPA 存储库

Spring Data JPA 存储库是个可以定义访问数据的接口。JPA 查询是根据您的方法名称自动创建的。例如，`CityRepository`接口可能声明了`findAllByState(String state)`方法来查找给定状态下的所有城市。

对于更复杂的查询，可以使用Spring Data 的[`Query`](https://docs.spring.io/spring-data/jpa/docs/2.2.3.RELEASE/api/org/springframework/data/jpa/repository/Query.html)注解对方法进行标注。

Spring Data 存储库通常继承`Repository`或`CrudRepository`接口。如果您使用自动配置，则会从您的主配置类（用`@EnableAutoConfiguration`或`@SpringBootApplication`注解标注的类）所在的包中搜索存储库。

以下示例显示了典型的 Spring Data 存储库接口定义：

```java
package com.example.myapp.domain;

import org.springframework.data.domain.*;
import org.springframework.data.repository.*;

public interface CityRepository extends Repository<City, Long> {

    Page<City> findAll(Pageable pageable);

    City findByNameAndStateAllIgnoringCase(String name, String state);

}
```

Spring Data JPA 存储库支持三种不同的引导模式：default，deferred 和 lazy。要启用 deferred 引导或 lazy 引导，请将`spring.data.jpa.repositories.bootstrap-mode`属性分别设置为`deferred`或`lazy`。使用 deferred 或 lazy 启动时，自动配置的`EntityManagerFactoryBuilder`将使用上下文的`AsyncTaskExecutor`作为引导执行程序。如果存在多个，则将使用一个名为`applicationTaskExecutor`的。

>[!tip]
>
>我们才仅仅触及 Spring Data JPA 的表面，有关完整的详细信息，请参见[Spring Data JPA 参考文档](https://docs.spring.io/spring-data/jdbc/docs/1.1.3.RELEASE/reference/html/)。



### 10.3.3 创建和删除 JPA 数据库

默认情况下，仅当您使用嵌入式数据库（H2，HSQL 或 Derby）时，才会自动创建 JPA 数据库。您可以使用`spring.jpa.*`属性显式配置 JPA 设置。例如，要创建和删除表，可以将以下行添加到`application.properties`：

```properties
spring.jpa.hibernate.ddl-auto=create-drop
```

>[!note]
>
>为此，Hibernate 自己的内部属性名称是`hibernate.hbm2ddl.auto`。您可以使用`spring.jpa.properties.*`（在将前缀添加到实体管理器之前，先去除前缀）来与其他 Hibernate 本地属性一起进行设置。下面的行显示了为 Hibernate 设置 JPA 属性的示例：

```properties
spring.jpa.properties.hibernate.globally_quoted_identifiers=true
```

前面示例中的行将`hibernate.globally_quoted_identifiers`属性的值`true`传递给 Hibernate 实体管理器。

默认情况下，DDL 执行（或验证）推迟到`ApplicationContext`启动之前。还有一个`spring.jpa.generate-ddl`标志，但是如果 Hibernate 自动配置处于可用状态，则不使用它，因为`ddl-auto`设置更细粒度。



### 10.3.4 在视图中打开 EntityManager

如果您正在运行 Web 应用程序，则 Spring Boot 默认情况下会注册[`OpenEntityManagerInViewInterceptor`](https://docs.spring.io/spring/docs/5.2.2.RELEASE/javadoc-api/org/springframework/orm/jpa/support/OpenEntityManagerInViewInterceptor.html)以应用“在视图中打开EntityManager”模式，以允许在 Web 视图中进行延迟加载。如果您不希望出现这种情况，则应在`application.properties`中将`spring.jpa.open-in-view`设置为`false`。



## 10.4 Spring Data JDBC

Spring Data 包括对 JDBC 的存储库支持，并将为`CrudRepository`上的方法自动生成 SQL。对于更高级的查询，提供了`@Query`注解。

当必要的依赖项位于类路径上时，Spring Boot 将自动配置 Spring Data 的 JDBC 存储库。只需引入依赖`spring-boot-starter-data-jdbc`，就可以将它们添加到您的项目中。如有必要，您可以通过在应用程序中添加`@EnableJdbcRepositories`注解或`JdbcConfiguration`子类来控制 Spring Data JDBC 的配置。

>[!tip]
>
>有关 Spring Data JDBC 的完整详细信息，请参阅[参考文档](https://docs.spring.io/spring-data/jdbc/docs/1.1.3.RELEASE/reference/html/)。



## 10.5 使用 H2 的 Web 控制台

[H2 数据库](https://www.h2database.com/)提供了一个[基于浏览器的控制台](https://www.h2database.com/html/quickstart.html#h2_console)，Spring Boot 可以为您自动配置该控制台。满足以下条件时，将自动配置控制台：

* 您正在开发基于 servlet 的 Web 应用程序。
* `com.h2database:h2`在类路径上。
* 您正在使用 [Spring Boot 的开发者工具](/using-spring-boot.md#8-开发者工具)。

>[!tip]
>
>如果您不使用 Spring Boot 的开发者工具，但仍想使用H2的控制台，则可以将`spring.h2.console.enabled`属性配置为`true`。

<span></span>



>[!note]
>
>H2 控制台仅在开发期间使用，因此应注意确保在生产中未将`spring.h2.console.enabled`设置为`true`。



### 10.5.1 更改 H2 控制台的路径

默认情况下，该控制台在`/h2-console`l路径下可用。您可以使用`spring.h2.console.path`属性来自定义控制台的路径。



## 10.6 使用 jOOQ

jOOQ 面向对象查询（[jOOQ](https://www.jooq.org/)）是[Data Geekery](https://www.datageekery.com/)的一个很流行的产品，它可以从数据库中生成 Java 代码，并允许您通过其流畅的 API 构建类型安全的 SQL 查询。商业版和开源版都可以与 Spring Boot 一起使用。



### 10.6.1 代码生成

为了使用 jOOQ 类型安全查询，您需要依据数据库的组织结构生成 Java 类。您可以按照[jOOQ 用户手册](https://www.jooq.org/doc/3.12.3/manual-single-page/#jooq-in-7-steps-step3)中的说明进行操作。如果您使用`jooq-codegen-maven`插件，并且还使用`spring-boot-starter-parent`“父POM”，则可以安全地忽略该插件的`<version>`标签。您还可以使用 Spring Boot 定义的版本变量（例如`h2.version`）来声明插件的数据库依赖关系。以下清单显示了一个示例：

```xml
<plugin>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <executions>
        ...
    </executions>
    <dependencies>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>${h2.version}</version>
        </dependency>
    </dependencies>
    <configuration>
        <jdbc>
            <driver>org.h2.Driver</driver>
            <url>jdbc:h2:~/yourdatabase</url>
        </jdbc>
        <generator>
            ...
        </generator>
    </configuration>
</plugin>
```



### 10.6.2 使用 DSLContext

jOOQ 所提供的流畅的 API 是通过`org.jooq.DSLContext`接口创建的。Spring Boot 将`DSLContext`自动配置为 Spring Bean，并将其连接到应用的`DataSource`。要使用` DSLContext `，可以使用`@Autowire`注入它，如下例所示：

```java
@Component
public class JooqExample implements CommandLineRunner {

    private final DSLContext create;

    @Autowired
    public JooqExample(DSLContext dslContext) {
        this.create = dslContext;
    }

}
```

>[!tip]
>
>jOOQ 手册上倾向于使用名为`create`的变量来保存`DSLContext`。

然后，您可以使用`DSLContext`构建查询，如以下示例所示：

```java
public List<GregorianCalendar> authorsBornAfter1980() {
    return this.create.selectFrom(AUTHOR)
        .where(AUTHOR.DATE_OF_BIRTH.greaterThan(new GregorianCalendar(1980, 0, 1)))
        .fetch(AUTHOR.DATE_OF_BIRTH);
}
```



### 10.6.3 jOOQ SQL 方言

除非已配置`spring.jooq.sql-dialect`属性，否则 Spring Boot 要确定要用于数据源的 SQL 方言。如果 Spring Boot 无法检测到方言，则使用`DEFAULT`。

>[!note]
>
>Spring Boot 只能自动配置开源版本的 jOOQ 支持的方言。



### 10.6.4 自定义 jOOQ

通过定义自己的`@Bean`定义（在创建 jOOQ `Configuration`时使用），可以实现更高级的自定义。 您可以为以下jOOQ 类型定义 bean：

* `ConnectionProvider`
* `ExecutorProvider`
* `TransactionProvider`
* `RecordMapperProvider`
* `RecordUnmapperProvider`
* `Settings`
* `RecordListenerProvider`
* `ExecuteListenerProvider`
* `VisitListenerProvider`
* `TransactionListenerProvider`

如果要完全控制 jOOQ 配置，还可以创建自己的`org.jooq.Configuration` `@Bean`。













































































































