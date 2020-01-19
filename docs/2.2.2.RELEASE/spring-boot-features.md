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



# 2. 外部化配置





## 4.4 日志级别



















