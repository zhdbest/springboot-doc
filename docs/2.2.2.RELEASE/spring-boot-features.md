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

除了`application.properties`文件之外，还可以使用以下命名约定来定义特定配置文件的属性：`application- {profile} .properties`。 如果没有设置激活的配置文件，则环境具有一组默认配置文件（默认为`[default]`）。换句话说，如果未显式激活任何概要文件，那么将加载`application-default.properties`中的属性。

特定配置文件的属性是从与标准`application.properties`相同的位置加载的，特定配置文件的文件总是会覆盖非特定文件，无论特定配置文件的文件是位于打包 jar 的内部还是外部。

如果指定了多个配置文件，则采用后赢策略。例如，添加`spring.profiles.active`属性指定的配置文件是在通过`SpringApplication` API 配置的配置文件之后，因此具有优先权。

>[!note]
>
>如果您在`spring.config.location`中指定了任何文件，则不考虑这些文件的特定配置文件的变体。如果您还想使用特定配置文件的属性，请配置`spring.config.location`时使用目录。



## 2.5 属性中的占位符

使用`application.properties`中的值时，它们会通过现有`Environment`进行过滤，因此您可以参考以前定义的值（例如，从System属性中）。

```properties
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

>[!tip]
>
>您还可以使用此技术来创建现有 Spring Boot 属性的“简短”变体。有关详细信息，请参见[howto.html](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto-use-short-command-line-arguments)操作方法。



## 2.6 加密属性

Spring Boot 不提供对加密属性值的任何内置支持，但是，它提供了修改 Spring `Environment`中包含的值所必需的挂钩点。 `EnvironmentPostProcessor`接口允许您在应用程序启动之前操纵`EnvironmentPostProcessor`。 有关详细信息，请参见[howto.html](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto-customize-the-environment-or-application-context)。

如果您正在寻找一种安全的方式来存储凭据和密码，则[Spring Cloud Vault](https://cloud.spring.io/spring-cloud-vault/)项目提供了对在[HashiCorp Vault](https://www.vaultproject.io/)中存储外部化配置的支持。



## 2.7 使用 YAML 代替 Properties

[YAML](https://yaml.org/)是 JSON 的超集，因此它是一种用于指定层次结构配置数据的便捷格式。只要在类路径上具有[SnakeYAML](https://bitbucket.org/asomov/snakeyaml)库，SpringApplication 类就会自动支持 YAML 作为 properties 的替代方法。

>[!note]
>
>如果您使用“启动器”，则`spring-boot-starter`会自动提供 SnakeYAML。



### 2.7.1 加载 YAML

Spring Framework 提供了两个合适的类，可用于加载 YAML 文档。`YamlPropertiesFactoryBean`将 YAML 作为`Properties`加载，而`YamlMapFactoryBean`将YAML作为`Map`加载。

例如，考虑以下 YAML 文档：

```yaml
environments:
    dev:
        url: https://dev.example.com
        name: Developer Setup
    prod:
        url: https://another.example.com
        name: My Cool App
```

前面的示例可以转换为以下属性：

```properties
environments.dev.url=https://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=https://another.example.com
environments.prod.name=My Cool App
```

YAML 列表用`[index]`间接引用表示为属性键。例如，考虑如下YAML：

```yaml
my:
   servers:
       - dev.example.com
       - another.example.com
```

前面的示例将转换为以下属性：

```properties
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

要使用 Spring Boot 的 `Binder`公用程序（`@ConfigurationProperties`所做的）绑定到类似的属性，您需要在类型为`java.util.List`（或`Set`）的目标 bean 中拥有一个属性，或者您需要提供一个`setter`或使用可变值对其进行初始化。例如，以下示例绑定到前面显示的属性：

```java
@ConfigurationProperties(prefix="my")
public class Config {

    private List<String> servers = new ArrayList<String>();

    public List<String> getServers() {
        return this.servers;
    }
}
```



### 2.7.2 在 Spring `Environment`中暴露 YAML 为属性

`YamlPropertySourceLoader`类可用于在 Spring `Environment`中将 YAML 暴露为`PropertySource`。这样做可以让您使用`@Value`注解和占位符语法来访问 YAML 属性。



### 2.7.3 多配置 YAML 文档

您可以使用`spring.profiles`键在一个文件中指定多个特定配置的 YAML 文档，以指示何时应用该文档，如以下示例所示：

```yaml
server:
    address: 192.168.1.100
---
spring:
    profiles: development
server:
    address: 127.0.0.1
---
spring:
    profiles: production & eu-central
server:
    address: 192.168.1.120
```

在前面的示例中，如果`development`配置文件处于生效状态，则`server.address`属性为`127.0.0.1`。同样，如果`production`和`eu-central`配置文件处于生效状态，则`server.address`属性为`192.168.1.120`。如果未启用 `development`，`production`和`eu-central` 配置文件，则该属性的值为`192.168.1.100`。

>[!note]
>
>因此`spring.profiles`可以包含一个简单的环境名称（例如`production`）或场景表达式。环境表达式允许表达更复杂的场景逻辑，例如`production & (eu-central | eu-west)`。有关更多详细信息，请参阅[参考指南](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java)。

如果在启动应用程序上下文时未显式激活任何环境的配置，则会激活默认配置文件。因此，在以下 YAML 中，我们为`spring.security.user.password`设置了一个值，该值仅在“默认”环境下可用：

```yaml
server:
  port: 8000
---
spring:
  profiles: default
  security:
    user:
      password: weak
```

而在以下示例中，能够始终设置密码是因为该密码未附加到任何环境，并且必须根据需要在所有其他环境配置中将其显式重置：

```yaml
server:
  port: 8000
spring:
  security:
    user:
      password: weak
```

使用`spring.profiles`元素指定的 Spring 配置可以很随意的使用`!`字符来否定。如果为单个文档指定了否定配置和非否定配置，则至少一个非否定配置文件必须匹配，并且否定配置文件不能匹配。



### 2.7.4 YAML 的缺点

YAML 文件无法使用`@PropertySource`注解加载。因此，在需要以这种方式加载值的情况下，需要使用属性文件。

在特定环境的 YAML 文件中使用多 YAML 文档语法可能会导致意外。例如，考虑文件中的以下配置：

**application-dev.yml**

```yaml
server:
  port: 8000
---
spring:
  profiles: "!test"
  security:
    user:
      password: "secret"
```

如果使用参数`--spring.profiles.active=dev`运行应用程序，则可能希望将`security.user.password`设置为“ secret”，但事实并非如此。

嵌套文档将被过滤，因为主文件名为`application-dev.yml`。它已经被认为是特定配置文件了，所以嵌套文档将被忽略。

>[!tip]
>
>我们建议您不要混用特定环境的 YAML 文件和多个 YAML 文档。坚持只使用其中之一。



## 2.8 类型安全的配置属性

使用`@Value("${property}")`注解来注入配置属性有时会很麻烦，尤其是当您	使用多个属性或数据本质上是分层的时。 Spring Boot 提供了一种使用属性的替代方法，该方法使强类型的 Bean 可以管理和验证应用程序的配置。

>[!tip]
>
>另请参见[`@Value`和类型安全的配置属性之间的区别](spring-boot-features.md#2810-configurationproperties和value)。



### 2.8.1 JavaBean属性绑定

可以如以下示例所示绑定一个 bean 声明标准 JavaBean 属性：

```java
package com.example;

import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("acme")
public class AcmeProperties {

    private boolean enabled;

    private InetAddress remoteAddress;

    private final Security security = new Security();

    public boolean isEnabled() { ... }

    public void setEnabled(boolean enabled) { ... }

    public InetAddress getRemoteAddress() { ... }

    public void setRemoteAddress(InetAddress remoteAddress) { ... }

    public Security getSecurity() { ... }

    public static class Security {

        private String username;

        private String password;

        private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

        public String getUsername() { ... }

        public void setUsername(String username) { ... }

        public String getPassword() { ... }

        public void setPassword(String password) { ... }

        public List<String> getRoles() { ... }

        public void setRoles(List<String> roles) { ... }

    }
}
```

前面的 POJO 定义了以下属性：

* `acme.enabled`，默认值为`false`。
* `acme.remote-address`，可以从`String`强制转换的类型。
* `acme.security.username`，嵌套的“security”对象，其名称由属性名称决定。特别是，返回类型在此根本不使用，可能是SecurityProperties。
* `acme.security.password`
* `acme.security.roles`，带有默认为`USER`的`String`集合。

>[!note]
>
>Spring Boot 自动配置大量利用`@ConfigurationProperties`来轻松配置自动配置的 bean。与自动配置类相似，Spring Boot 中可用的`@ConfigurationProperties`类仅供内部使用。通过 properties 文件、YAML 文件、环境变量等配置映射到该类的属性是公共 API，但是该类本身的内容并不意味着可以直接使用。

<span></span>



>[!note]
>
>这种约定依赖于默认的空构造函数，并且 getter 和 setter 通常是强制性的，因为绑定是通过标准 Java Beans属性描述符进行的，就像在Spring MVC 中一样。在以下情况下，可以忽略 setter ：
>
>* 只要将 Maps 初始化，它们就需要 getter，但不一定需要使用 setter，因为它们可以被绑定器转换。
>* 可以通过索引（通常是使用 YAML 的时候）或使用单个逗号分隔的值（属性）来访问集合和数组。在后一种情况下，必须要有 setter。我们建议始终为此类型添加 setter。如果初始化集合，请确保它并不是不可变的（如上例所示）。
>* 如果嵌套的 POJO 属性（如前面示例中的`Security`字段）被初始化了，则不需要 setter。如果希望绑定器通过使用其默认构造函数动态创建实例，则需要一个 setter。
>
>有些人使用 Lombok 项目自动添加 getter 和 setter。确保 Lombok 不会为这种类型生成任何特定的构造函数，因为容器会自动使用它来实例化该对象。
>
>最后，仅考虑标准 Java Bean 属性，并且不支持对静态属性的绑定。



### 2.8.2 构造器绑定

上一节中的示例可以用不变的方式重写，如下例所示：

```java
package com.example;

import java.net.InetAddress;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.ConstructorBinding;
import org.springframework.boot.context.properties.DefaultValue;

@ConstructorBinding
@ConfigurationProperties("acme")
public class AcmeProperties {

    private final boolean enabled;

    private final InetAddress remoteAddress;

    private final Security security;

    public AcmeProperties(boolean enabled, InetAddress remoteAddress, Security security) {
        this.enabled = enabled;
        this.remoteAddress = remoteAddress;
        this.security = security;
    }

    public boolean isEnabled() { ... }

    public InetAddress getRemoteAddress() { ... }

    public Security getSecurity() { ... }

    public static class Security {

        private final String username;

        private final String password;

        private final List<String> roles;

        public Security(String username, String password,
                @DefaultValue("USER") List<String> roles) {
            this.username = username;
            this.password = password;
            this.roles = roles;
        }

        public String getUsername() { ... }

        public String getPassword() { ... }

        public List<String> getRoles() { ... }

    }

}
```

在此设置中，`@ConstructorBinding`注解用于标识应使用构造器绑定。这意味着绑定器将期望找到带有您希望绑定的参数的构造器。

`@ConstructorBinding`类的嵌套成员（例如上例中的`Security`）也将通过其构造器进行绑定。

可以使用`@DefaultValue`指定默认值，并且将应用相同的转换服务将`String`值强制为缺少属性的目标类型。

>[!note]
>
>要使用构造器绑定，必须使用`@EnableConfigurationProperties`或配置属性扫描来启用该类。您不能将构造器绑定与常规 Spring 机制创建的 bean 一起使用（例如：`@Component` bean、通过`@Bean`方法创建的bean、使用`@Import`加载的 bean）。

<span></span>



>[!tip]
>
>如果您的类具有多个构造函数，则还可以直接在应绑定的构造函数上使用`@ConstructorBinding`。



### 2.8.3 启用`@ConfigurationProperties`标注的类型

Spring Boot 提供了绑定`@ConfigurationProperties`类型并将其注册为 Bean 的基础架构。您可以逐类启用配置属性，也可以启用与组件扫描类似的方式进行配置属性扫描。

有时，用`@ConfigurationProperties`标注的类可能不适合扫描，例如，如果您正在开发自己的自动配置，或者想要有条件地启用它们。在这些情况下，请使用`@EnableConfigurationProperties`注解指定要处理的类型列表。可以在任何`@Configuration`类上完成此操作，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
@EnableConfigurationProperties(AcmeProperties.class)
public class MyConfiguration {
}
```

要使用配置属性扫描，请将`@ConfigurationPropertiesScan`注解添加到您的应用程序。通常，它被添加到用`@SpringBootApplication`标注的主应用程序类中，但是也可以将其添加到任何`@Configuration`类中。默认情况下，将从声明注解的类的包中进行扫描。如果要定义要扫描的特定程序包，可以按照以下示例所示进行操作：

```java
@SpringBootApplication
@ConfigurationPropertiesScan({ "com.example.app", "org.acme.another" })
public class MyApplication {
}
```

>[!note]
>
>使用配置属性扫描或通过`@EnableConfigurationProperties`注册`@ConfigurationProperties` Bean时，该Bean具有常规名称：`<prefix>-<fqn>`，其中`<prefix>`是`@ConfigurationProperties`注解指定的环境键前缀，`<fqn>`中是该 Bean 的完全限定名称。如果注解不提供任何前缀，则仅使用 Bean 的完全限定名称。
>
>上例中的 bean 名称是`acme-com.example.AcmeProperties`。

我们建议`@ConfigurationProperties`仅处理环境，尤其不要从上下文中注入其他 bean。对于极端情况，可以使用 setter 注入或框架提供的任何`*Aware`接口（例如，需要访问`Environment`的`EnvironmentAware`）。如果仍要使用构造函数注入其他 bean，则必须使用`@Component`注解配置属性 bean，并使用基于 JavaBean 的属性绑定。



### 2.8.4 使用@`ConfigurationProperties`标注的类型

这种配置风格与`SpringApplication`外部 YAML 配置一起使用特别有效，如以下示例所示：

```yaml
# application.yml

acme:
    remote-address: 192.168.1.1
    security:
        username: admin
        roles:
          - USER
          - ADMIN

# additional configuration as required
```

要使用`@ConfigurationProperties` Bean，可以像其他任何 Bean 一样注入它们，如以下示例所示：

```java
@Service
public class MyService {

    private final AcmeProperties properties;

    @Autowired
    public MyService(AcmeProperties properties) {
        this.properties = properties;
    }

    //...

    @PostConstruct
    public void openConnection() {
        Server server = new Server(this.properties.getRemoteAddress());
        // ...
    }

}
```

>[!tip]
>
>使用`@ConfigurationProperties`还可让您生成元数据文件，IDE 可以使用这些元数据文件为您自己的键提供自动补全功能。有关详细信息，请参见[附录](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/appendix-configuration-metadata.html#configuration-metadata)。



### 2.8.5 第三方配置

除了使用`@ConfigurationProperties`标注类外，还可以在公共`@Bean`方法上使用它。当您要将属性绑定到控件之外的第三方组件时，这样做特别有用。

要从`Environment`属性配置 Bean，请将`@ConfigurationProperties`添加到其 Bean 注册中，如以下示例所示：

```java
@ConfigurationProperties(prefix = "another")
@Bean
public AnotherComponent anotherComponent() {
    ...
}
```

用`another`前缀定义的任何 JavaBean 属性都以类似于前面的`AcmeProperties`示例的方式映射到该`AnotherComponent` bean。



### 2.8.6 宽松绑定

Spring Boot 使用一些宽松的规则将`Environment`属性绑定到`@ConfigurationProperties` bean，因此`Environment`属性名称和 bean 属性名称之间不需要完全匹配。有用的常见示例包括破折号分隔的环境属性（例如，`context-path`绑定到`contextPath`）和大写的环境属性（例如`PORT`绑定到`port`）。

例如，考虑以下`@ConfigurationProperties`类：

```java
@ConfigurationProperties(prefix="acme.my-project.person")
public class OwnerProperties {

    private String firstName;

    public String getFirstName() {
        return this.firstName;
    }

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

}
```

使用前面的代码，以下属性名称可以全部使用：

  

**表2：宽松绑定**

| 属性                                | 备注                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| `acme.my-project.person.first-name` | 短横线，建议在`.properties`和`.yml`文件中使用。              |
| `acme.myProject.person.firstName`   | 标准驼峰式语法。                                             |
| `acme.my_project.person.first_name` | 下划线表示法，是`.properties`和`.yml`文件中使用的另一种格式。 |
| `ACME_MYPROJECT_PERSON_FIRSTNAME`   | 大写格式，使用系统环境变量时建议使用。                       |

>[!note]
>
>注解的`prefix`值必须为短横线（小写，并用-分隔，例如`acme.my-project.person`）。

  

**表3：每个属性源的宽松绑定规则**

| 属性源          | 简单的                                                | 列表                                                         |
| :-------------- | :---------------------------------------------------- | :----------------------------------------------------------- |
| Properties 文件 | 驼峰、短横线或下划线                                  | 使用`[ ]`或逗号分隔值的标准 List 语法                        |
| YAML 文件       | 驼峰、短横线或下划线                                  | 标准 YAML 列表语法或逗号分隔值                               |
| 环境变量        | 以下划线作为分隔符的大写格式。不应在属性名称中使用`_` | 包含下划线的数字值，例如`MY_ACME_1_OTHER = my.acme[1].other` |
| 系统属性        | 驼峰、短横线或下划线                                  | 使用`[ ]`或逗号分隔值的标准 List 语法                        |

>[!tip]
>
>我们建议，如果可能的话，属性以小写短横线的格式存储，例如`my.property-name = acme`。

绑定到`Map`属性时，如果`key`包含除小写字母数字字符或`-`以外的任何其他字符，则需要使用方括号表示法，以便保留原始值。如果`key`没有被`[]`包围，则所有非字母数字或`-`的字符都将被删除。例如，考虑将以下属性绑定到`Map`：

```yaml
acme:
  map:
    "[/key1]": value1
    "[/key2]": value2
    /key3: value3
```

上面的属性将以`/key1`，`/key2`和`key3`作为 map 中的键绑定到`Map`。

>[!note]
>
>对于 YAML 文件，方括号必须用引号引起来，以便正确解析 key。



### 2.8.7 合并复杂类型

如果在多个位置配置了列表，则通过替换整个列表来进行覆盖。

例如，假设`MyPojo`对象的`name`和`description`属性默认为`null`。下面的示例通过`AcmeProperties`暴露`MyPojo`对象的列表：

```java
@ConfigurationProperties("acme")
public class AcmeProperties {

    private final List<MyPojo> list = new ArrayList<>();

    public List<MyPojo> getList() {
        return this.list;
    }

}
```

考虑以下配置：

```yaml
acme:
  list:
    - name: my name
      description: my description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
```

如果当前不是`dev`环境，则`AcmeProperties.list`包含一个`MyPojo`实体，如先前所定义。但是，如果当前是`dev`环境，则该列表仍仅包含一个实体（`name`为`my another name`，并且`description`为`null`）。此配置不会将第二个`MyPojo`实例添加到列表中，并且不会合并元素。

在多个环境中指定`List`时，将使用优先级最高的列表（并且仅使用那个列表）。 考虑以下示例：

```yaml
acme:
  list:
    - name: my name
      description: my description
    - name: another name
      description: another description
---
spring:
  profiles: dev
acme:
  list:
    - name: my another name
```

在前面的示例中，如果当前处于`dev`环境，则`AcmeProperties.list`包含一个`MyPojo`实体（其`name`为`my another name`，`description`为`null`）。对于 YAML，可以使用逗号分隔的列表和 YAML 列表来完全覆盖列表的内容。

对于`Map`属性，可以绑定从多个来源的属性值。但是，对于多个源中的同一属性，将使用优先级最高的属性。下面的示例通过`AcmeProperties`暴露`Map <String，MyPojo>`：

```java
@ConfigurationProperties("acme")
public class AcmeProperties {

    private final Map<String, MyPojo> map = new HashMap<>();

    public Map<String, MyPojo> getMap() {
        return this.map;
    }

}
```

考虑以下配置：

```yaml
acme:
  map:
    key1:
      name: my name 1
      description: my description 1
---
spring:
  profiles: dev
acme:
  map:
    key1:
      name: dev name 1
    key2:
      name: dev name 2
      description: dev description 2
```

如果当前不处于`dev`环境，则`AcmeProperties.map`包含一个键为`key1`的实体（`name`为`my name 1`，`description`为`my description 1`）。但是，如果当前是`dev`环境，则`map`包含两个实体，其中键为`key1`（名称为`dev name 1`，其描述为`my description 1`）和`key2`（名称为`dev name 2`，其描述为`dev description 2`） 。

>[!note]
>
>前面的合并规则不仅适用于 YAML 文件，而且适用于所有属性源中的属性。



### 2.8.8 属性转换

当 Spring Boot 绑定到`@ConfigurationProperties` bean时，它尝试将外部应用程序属性强转为正确的类型。如果需要自定义类型转换，则可以提供一个`ConversionService` bean（具有一个名为`conversionService`的bean）或自定义属性编辑器（通过`CustomEditorConfigurer` bean）或自定义`Converters`（使用`@ConfigurationPropertiesBinding`标注的bean定义）。

>[!note]
>
>由于在应用程序生命周期中非常早就请求了此bean，因此请确保限制您的`ConversionService`使用的依赖项。通常，您需要的任何依赖项在创建时可能都没有完全初始化。如果配置键强制不需要自定义`ConversionService`，而仅依赖于具有`@ConfigurationPropertiesBinding`限定的自定义转换器，则可能需要重命名自定义`ConversionService`。



#### 转换持续时间

Spring Boot 为表达持续时间提供了专门的支持。如果暴露`java.time.Duration`属性，则应用程序属性中的以下格式可用：

* 常规的`long`表示（使用毫秒作为默认单位，除非已指定`@DurationUnit`）
* [`java.time.Duration`使用](https://docs.oracle.com/javase/8/docs/api//java/time/Duration.html#parse-java.lang.CharSequence-)的标准 ISO-8601 格式
* 值和单位相结合的更易读的格式（例如`10s`表示10秒）

考虑以下示例：

```java
@ConfigurationProperties("app.system")
public class AppSystemProperties {

    @DurationUnit(ChronoUnit.SECONDS)
    private Duration sessionTimeout = Duration.ofSeconds(30);

    private Duration readTimeout = Duration.ofMillis(1000);

    public Duration getSessionTimeout() {
        return this.sessionTimeout;
    }

    public void setSessionTimeout(Duration sessionTimeout) {
        this.sessionTimeout = sessionTimeout;
    }

    public Duration getReadTimeout() {
        return this.readTimeout;
    }

    public void setReadTimeout(Duration readTimeout) {
        this.readTimeout = readTimeout;
    }

}
```

要设定30秒 session 超时，则`30`，`PT30S`和`30s`都是等效的。可以使用以下任意形式指定500ms的读取超时：`500`，`PT0.5S`和`500ms`。

还支持以下单位：

- `ns` 纳秒
- `us` 微秒
- `ms` 毫秒
- `s` 秒
- `m` 分
- `h` 小时
- `d` 天

默认单位是毫秒，可以使用`@DurationUnit`覆盖，如上面的示例所示。

>[!tip]
>
>如果您要从仅使用`Long`表示持续时间的先前版本进行升级，并且当前单位不是毫秒，则请确保在转换到`Duration`的同时定义单位（使用`@DurationUnit`）。这样做可以提供透明的升级路径，同时支持更丰富的格式。



#### 转换数据大小

Spring 框架具有`DataSize`值类型，以字节为单位表示大小。如果暴露`DataSize`属性，则应用程序属性中的以下格式可用：

* 常规的`long`表示形式（除非已指定`@DataSizeUnit`，否则使用字节作为默认单位）
* 值和单位连在一起的更易读的格式（例如`10MB`表示10兆字节）

考虑如下示例：

```java
@ConfigurationProperties("app.io")
public class AppIoProperties {

    @DataSizeUnit(DataUnit.MEGABYTES)
    private DataSize bufferSize = DataSize.ofMegabytes(2);

    private DataSize sizeThreshold = DataSize.ofBytes(512);

    public DataSize getBufferSize() {
        return this.bufferSize;
    }

    public void setBufferSize(DataSize bufferSize) {
        this.bufferSize = bufferSize;
    }

    public DataSize getSizeThreshold() {
        return this.sizeThreshold;
    }

    public void setSizeThreshold(DataSize sizeThreshold) {
        this.sizeThreshold = sizeThreshold;
    }

}
```

若要指定10 MB的缓冲区大小，则`10`和`10MB`是等效的。 256个字节的阈值可以指定为`256`或`256B`。

还支持以下单位：

- `B` 字节
- `KB` 千字节
- `MB` 兆字节
- `GB` 千兆字节
- `TB` 兆兆字节

默认单位是字节，可以使用`@DataSizeUnit`覆盖，如上面的示例所示。

>[!tip]
>
>如果您要从仅使用`Long`表示大小的先前版本进行升级，并且当前单位不是字节，则请确保在转换到`DataSize`的同时定义单位（使用`@DataSizeUnit`）。这样做可以提供透明的升级路径，同时支持更丰富的格式。



### 2.8.9 @ConfigurationProperties 校验

当使用 Spring 的`@Validated`注解标注`@ConfigurationProperties`类时，Spring Boot 就会尝试对其进行校验。您可以在配置类上直接使用 JSR-303 `javax.validation`约束注解。 为此，请确保在类路径上有兼容的 JSR-303 实现，然后将约束注解添加到字段中，如以下示例所示：

```java
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

    @NotNull
    private InetAddress remoteAddress;

    // ... getters and setters

}
```

>[!tip]
>
>您还可以通过使用`@Validated`标注创建配置属性的`@Bean`方法来触发校验。

为了确保始终为嵌套的属性触发校验，即使未找到任何属性，也必须使用`@Valid`标注关联的字段。以下示例基于前面的`AcmeProperties`示例：

```java
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

    @NotNull
    private InetAddress remoteAddress;

    @Valid
    private final Security security = new Security();

    // ... getters and setters

    public static class Security {

        @NotEmpty
        public String username;

        // ... getters and setters

    }

}
```

您还可以通过创建一个名为`configurationPropertiesValidator`的 bean 定义来添加自定义的Spring `Validator`。`@Bean`方法应声明为静态（`static`）的。配置属性校验器是在应用程序生命周期的早期创建的，将`@Bean`方法声明为`static`可以使创建该 Bean 而不必实例化`@Configuration`类。这样做避免了由早期实例化引起的任何问题。

>[!tip]
>
>`spring-boot-actuator`模块包括一个暴露所有`@ConfigurationProperties` bean的端点。将您的Web浏览器指向`/actuator/configprops`或使用等效的 JMX 端点。有关详细信息，请参见“[生产就绪功能](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/production-ready-features.html#production-ready-endpoints)”部分。



### 2.8.10 `@ConfigurationProperties`和`@Value`

`@Value`注解是核心容器功能，它没有提供与类型安全的配置属性相同的功能。下表总结了`@ConfigurationProperties`和`@Value`支持的功能：

| 功能                                                         | `@ConfigurationProperties` | `@Value` |
| :----------------------------------------------------------- | :------------------------- | :------- |
| [宽松绑定](spring-boot-features.md#286-宽松绑定)             | 支持                       | 不支持   |
| [元数据支持](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/appendix-configuration-metadata.html#configuration-metadata) | 支持                       | 不支持   |
| `SpEL`评价                                                   | 不支持                     | 支持     |

如果您为自己的组件定义了一组配置键，我们建议您将它们组合在用`@ConfigurationProperties`标注的 POJO 中。您还应该意识到，由于`@Value`不支持宽松绑定，因此如果您需要使用环境变量来提供值，那么它不是一个很好的选择。

最后，尽管您可以在`@Value`中编写`SpEL`表达式，但不会从[应用属性文件](spring-boot-features.md#23-应用属性文件)中处理此类表达式。



# 3. Profiles

Spring Profiles 提供了一种隔离应用程序配置的部分并使之仅在某些环境中可用的方法可以使用`@Profile`标记任何`@Component`、`@Configuration`或`@ConfigurationProperties`，以限制其何时加载，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```

>[!note]
>
>如果`@ConfigurationProperties` Bean是通过`@EnableConfigurationProperties`而非自动扫描注册的，则需要在使用`@EnableConfigurationProperties`标注的`@Configuration`类上标注`@Profile`。在扫描`@ConfigurationProperties`的情况下，可以在`@ConfigurationProperties`类本身上标注`@Profile`。

您可以使用`spring.profiles.active` `Environment`属性来指定哪个 profiles 处于激活状态。您可以通过本章前面介绍的任何方式指定属性。例如，您可以将其包含在`application.properties`中，如以下示例所示：

```properties
spring.profiles.active=dev,hsqldb
```

您也可以使用以下开关在命令行上指定它：`--spring.profiles.active=dev,hsqldb`。



## 3.1 添加激活的 Profiles

`spring.profiles.active`属性遵循与其他属性相同的排序规则：使用最高的`PropertySource`属性。这意味着您可以在`application.properties`中指定激活的 profiles，然后使用命令行开关替换它们。

有时，将特定于配置文件的属性添加到激活的 profiles 而不是替换它们是很有用的。`spring.profiles.include`属性可用于无条件添加激活的 profiles。`SpringApplication`入口点还具有 Java API，用于设置其他profiles（即，在由`spring.profiles.active`属性激活的 profiles 之上）。 请参阅[`SpringApplication`](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/api//org/springframework/boot/SpringApplication.html)中的`setAdditionalProfiles()`方法。

例如，使用开关`--spring.profiles.active=prod`运行具有以下属性的应用程序时，`proddb`和`prodmq` 两个profile 也会被激活：

```yaml
---
my.property: fromyamlfile
---
spring.profiles: prod
spring.profiles.include:
  - proddb
  - prodmq
```

>[!note]
>
>请记住，可以在 YAML 文档中定义`spring.profiles`属性，以确定何时将该特定文档包括在配置中。有关更多详细信息，请参见[`howto.html`](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto-change-configuration-depending-on-the-environment)。



## 3.2 使用编程的方式设置 Profiles

您可以在应用程序运行之前通过调用`SpringApplication.setAdditionalProfiles(…)`以编程方式设置激活的 Profiles。也可以使用 Spring 的`ConfigurableEnvironment`接口来激活 profiles。



## 3.3 特定环境的配置文件

`application.properties`（或`application.yml`）和通过`@ConfigurationProperties`引用的文件的特定环境的配置文件的变体都被视为文件并已加载。有关详细信息，请参见“[Profiles-specific 属性](spring-boot-features.md#24-profile-specific-属性)”。



# 4. 日志

Spring Boot 使用[Commons Logging](https://commons.apache.org/logging)进行所有内部日志记录，但是对底层日志实现保持开放的状态。提供了[Java Util Logging](https://docs.oracle.com/javase/8/docs/api//java/util/logging/package-summary.html)，[Log4J2](https://logging.apache.org/log4j/2.x/)和[Logback](https://logback.qos.ch/)的默认配置。在每种情况下，记录器（ logger）都已预先配置为使用控制台输出，同时还提供可选文件输出。

默认情况下，如果使用“Starters”，则使用 Logback 进行日志记录。还引入适当的 Logback 路由，以确保使用 Java Util Logging，Commons Logging，Log4J 或 SLF4J 的从属库都能正常工作。

>[!tip]
>
>有许多可用于 Java 的日志记录框架。如果上面的列表看起来令人困惑，请不要担心。通常，您不需要更改日志记录依赖项，并且使用 Spring Boot 默认的也可以正常工作。

<span></span>



>[!tip]
>
>将应用程序部署到 servlet 容器或应用程序服务器时，通过 Java Util Logging API 记录的日志记录不会路由到应用程序的日志中。这样可以防止容器或其他已部署到容器中的应用程序记录的日志出现在应用程序的日志中。



## 4.1 日志格式

Spring Boot 的默认日志输出看起来像以下示例：

```
2019-03-05 10:57:51.112  INFO 45469 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/7.0.52
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2019-03-05 10:57:51.253  INFO 45469 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1358 ms
2019-03-05 10:57:51.698  INFO 45469 --- [ost-startStop-1] o.s.b.c.e.ServletRegistrationBean        : Mapping servlet: 'dispatcherServlet' to [/]
2019-03-05 10:57:51.702  INFO 45469 --- [ost-startStop-1] o.s.b.c.embedded.FilterRegistrationBean  : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
```

输出的有以下项目：

* 日期和时间：精确到毫秒，易于排序。
* 日志级别：`ERROR`、`WARN`、`INFO`、`DEBUG`或`TRACE`
* 进程ID
* 一个`---`分隔符，用于标识出实际日志消息的开始。
* 线程名称：用方括号括起来（对于控制台输出可能会被截断）。
* Logger名称：通常是源类名称（通常缩写）。
* 日志信息

>[!note]
>
>Logback没有`FATAL`级别，它对应的是`ERROR`。



## 4.2 控制台输出

默认日志配置在写入消息时会将消息回显到控制台。默认情况下记录`ERROR`级，`WARN`级和`INFO`级消息。您还可以通过使用`--debug`启动应用程序来启用“调试”模式。

```shell
$ java -jar myapp.jar --debug
```

>[!note]
>
>你也可以在`application.properties`中指定`debug=true`。

启用调试模式后，将配置一些核心 logger（嵌入式容器，Hibernate 和 Spring Boot）以输出更多信息。启用调试模式不会将您的应用程序配置为记录所有`DEBUG`级别的消息。

另外，您可以通过使用`--trace`（或`application.properties`中的`trace = true`）启动应用程序来启用“跟踪”模式。这样做可以为某些核心 logger（嵌入式容器，Hibernate 模式生成以及整个 Spring 产品组合）启用跟踪记录。



### 4.2.1 彩色输出

如果您的终端支持 ANSI，则使用彩色输出来提高可读性。您可以将`spring.output.ansi.enabled`设置为[支持的值](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/api//org/springframework/boot/ansi/AnsiOutput.Enabled.html)，以覆盖自动检测。

使用`％clr`转换字配置颜色编码。转换器以最简单的形式根据日志级别为输出着色，如以下示例所示：

```
%clr(%5p)
```

下表展示了日志级别与颜色的映射关系：

| 级别    | 颜色 |
| :------ | :--- |
| `FATAL` | 红色 |
| `ERROR` | 红色 |
| `WARN`  | 黄色 |
| `INFO`  | 绿色 |
| `DEBUG` | 绿色 |
| `TRACE` | 绿色 |

另外，您可以通过将其提供为转换的选项来指定应使用的颜色或样式。例如，为了得到黄色的文本，使用如下配置：

```
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){yellow}
```

支持如下颜色和格式：

- `blue`
- `cyan`
- `faint`
- `green`
- `magenta`
- `red`
- `yellow`



## 4.3 文件输出

默认情况下，Spring Boot 日志只记录到控制台，不写日志文件。如果除了在控制台输出外还想写日志文件，则需要设置`logging.file.name`或`logging.file.path`属性（例如，在`application.properties`中设置）。

**表4. 日志属性**

| `logging.file.name` | `logging.file.path` | 例子       | 描述                                                         |
| :------------------ | :------------------ | :--------- | :----------------------------------------------------------- |
| *(不指定)*          | *(不指定)*          |            | 只输出到控制台                                               |
| Specific file       | *(不指定)*          | `my.log`   | 写入指定的日志文件。名称可以是确切的位置，也可以相对于当前目录的位置。 |
| *(不指定)*          | Specific directory  | `/var/log` | 写入`spring.log` 到指定的目录。名称可以是确切的位置，也可以相对于当前目录的位置。 |

日志文件达到 10 MB 时会开始轮换，并且与控制台输出一样，默认情况下会记录`ERROR`级别，`WARN`级别和`INFO`级别的消息。可以使用`logging.file.max-size`属性更改大小限制。除非已设置`logging.file.max-history`属性，否则以前轮换的文件将无限期存档。可以使用`logging.file.total-size-cap`限制日志归档文件的总大小。当日志归档的总大小超过该阈值时，将删除备份。要在应用程序启动时强制清除日志存档，请使用`logging.file.clean-history-on-start`属性。

>[!tip]
>
>日志属性独立于实际的日志基础架构。因此，特定的配置键（例如 Logback 的`logback.configurationFile`）不是由 Spring Boot 管理的。



## 4.4 日志级别

通过使用`logging.level.<logger-name>=<level>`，可以在Spring `Environment`中（例如，在`application.properties`中）设置所有支持的日志记录器级别。其中`level`是`TRACE`，`DEBUG`，`INFO`， `WARN`，`ERROR`，`FATAL`或`OFF`。可以使用`logging.level.root`配置`root`记录器。

以下示例显示了`application.properties`中可能的日志记录设置：

```properties
logging.level.root=warn
logging.level.org.springframework.web=debug
logging.level.org.hibernate=error
```

也可以使用环境变量设置日志记录级别。例如，`LOGGING_LEVEL_ORG_SPRINGFRAMEWORK_WEB=DEBUG`会将`org.springframework.web`设置为`DEBUG`。

>[!note]
>
>以上方法仅适用于程序包级别的日志记录。由于宽松的绑定总是将环境变量转换为小写，因此无法以这种方式为单个类配置日志记录。如果需要为类配置日志记录，则可以使用[SPRING_APPLICATION_JSON](spring-boot-features.md#2-外部化配置)变量。



## 4.5 日志组

能够将相关 logger 组合在一起通常可以很有用，以便可以同时配置它们。例如，您可能通常会更改所有与Tomcat相关的记录器的日志记录级别，但是您不容易记住顶级软件包。

为了解决这个问题，Spring Boot 允许您在 Spring `Environment`中定义日志组。例如，下面是如何在`application.properties`中定义一个“tomcat”组。

```properties
logging.group.tomcat=org.apache.catalina, org.apache.coyote, org.apache.tomcat
```

定义好之后，就可以使用一行代码改变组内所有 logger 的级别：

```properties
logging.level.tomcat=TRACE
```

Spring Boot 包含以下预定义的日志记录组，它们可以直接使用：

| 名称 | Loggers                                                      |
| :--- | :----------------------------------------------------------- |
| web  | `org.springframework.core.codec`, `org.springframework.http`, `org.springframework.web`, `org.springframework.boot.actuate.endpoint.web`, `org.springframework.boot.web.servlet.ServletContextInitializerBeans` |
| sql  | `org.springframework.jdbc.core`, `org.hibernate.SQL`, `org.jooq.tools.LoggerListener` |



## 4.6 自定义日志配置

可以通过在类路径上引入适当的库来激活各种日志记录系统，并可以通过在类路径的根目录或以下Spring `Environment`属性：`logging.config`指定的位置中提供适当的配置文件来进一步自定义日志。

您可以通过使用`org.springframework.boot.logging.LoggingSystem`系统属性来强制 Spring Boot 使用特定的日志记录系统。该值应该是`LoggingSystem`实现的全限定类名。您也可以使用`none`作为值来完全禁用 Spring Boot 的日志记录配置。

>[!note]
>
>由于日志记录是在创建`ApplicationContext`之前初始化的，因此无法通过 Spring `@Configuration`文件中的`@PropertySources`来控制日志记录。更改日志记录系统或完全禁用它的唯一方法是通过系统属性进行设置。

根据您的日志记录系统，将加载以下文件：

| 日志记录系统            | 定制化                                                       |
| :---------------------- | :----------------------------------------------------------- |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml`, or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

>[!note]
>
>如果可能，我们建议您在日志配置中使用`-spring`变体（例如，`logback-spring.xml`而不是`logback.xml`）。如果使用标准配置位置，Spring 将无法完全控制日志初始化。

>[!waring]
>
>从“可执行jar”运行时，Java Util Logging 存在一些已知的类加载问题可能会引起一些问题。我们建议您从“可执行jar”运行时尽可能避免使用它。

为了帮助定制，其他一些属性从Spring `Environment`转移到了系统属性，如下表所述：

| Spring Environment                    | 系统属性                          | 备注                                                         |
| :------------------------------------ | :-------------------------------- | :----------------------------------------------------------- |
| `logging.exception-conversion-word`   | `LOG_EXCEPTION_CONVERSION_WORD`   | 记录异常时使用的转换字。                                     |
| `logging.file.clean-history-on-start` | `LOG_FILE_CLEAN_HISTORY_ON_START` | 是否在启动时清除存档日志文件（如果启用了LOG_FILE）（仅支持默认Logback设置） |
| `logging.file.name`                   | `LOG_FILE`                        | 如果定义了，它将在默认日志配置中使用。                       |
| `logging.file.max-size`               | `LOG_FILE_MAX_SIZE`               | 日志文件的最大值（如果启用了LOG_FILE）（仅支持默认Logback设置） |
| `logging.file.max-history`            | `LOG_FILE_MAX_HISTORY`            | 要保留的最大归档日志文件数（如果启用了LOG_FILE）（仅支持默认Logback设置） |
| `logging.file.path`                   | `LOG_PATH`                        | 如果定义了，它将在默认日志配置中使用。                       |
| `logging.file.total-size-cap`         | `LOG_FILE_TOTAL_SIZE_CAP`         | 要保留的日志备份的总大小（如果启用了LOG_FILE）（仅支持默认Logback设置） |
| `logging.pattern.console`             | `CONSOLE_LOG_PATTERN`             | 控制台上要使用的日志模式（stdout）（仅支持默认Logback设置）  |
| `logging.pattern.dateformat`          | `LOG_DATEFORMAT_PATTERN`          | 日志日期格式的拼接模式（仅支持默认Logback设置）              |
| `logging.pattern.file`                | `FILE_LOG_PATTERN`                | 文件中使用的日志模式 （如果`LOG_FILE`已启用）（仅支持默认Logback设置） |
| `logging.pattern.level`               | `LOG_LEVEL_PATTERN`               | 呈现日志级别时使用的格式（默认`%5p`）（仅支持默认Logback设置） |
| `logging.pattern.rolling-file-name`   | `ROLLING_FILE_NAME_PATTERN`       | 转存日志文件名的模式（默认`${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz`）（仅支持默认Logback设置） |
| `PID`                                 | `PID`                             | 当前进程ID（如果能发现并且尚未将其定义为OS环境变量）         |

所有支持的日志记录系统在解析其配置文件时都可以查阅系统属性。 有关示例，请参见`spring-boot.jar`中的默认配置：

- [Logback](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)
- [Log4j 2](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/log4j2/log4j2.xml)
- [Java Util logging](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/java/logging-file.properties)

>[!tip]
>
>如果要在日志记录属性中使用占位符，则应使用[Spring Boot的语法](spring-boot-features.md#25-属性中的占位符)而不是基础框架的语法。需要注意的是，如果使用 Logback，则应使用`:`作为属性名称与其默认值之间的分隔符，而不应使用`:-`。

<span></span>



>[!tip]
>
>您可以通过仅覆盖`LOG_LEVEL_PATTERN`（或 Logback 的`logging.pattern.level`）来将MDC和其他特别内容添加到日志行。例如，如果使用`logging.pattern.level=user:%X{user} %5p`，则默认日志格式包含“ user”的MDC条目（如果存在），如以下示例所示：
>
>```
>2019-08-30 12:30:04.031 user:someone INFO 22174 --- [  nio-8080-exec-0] demo.Controller
>Handling authenticated request
>```



## 4.7 Logback 扩展

Spring Boot 包含许多 Logback 扩展，可以帮助进行高级配置。你可以在`logback-spring.xml`配置文件中使用这些扩展。

>[!note]
>
>由于标准`logback.xml`配置文件加载得太早，因此无法在其中使用扩展。您需要使用`logback-spring.xml`或定义`logging.config`属性来使用扩展。

<span></span>



>[!warning]
>
>这些扩展不能与 Logback 的[配置扫描](https://logback.qos.ch/manual/configuration.html#autoScan)一起使用。如果尝试这样做，则对配置文件进行更改时将导致类似于以下记录之一的错误：

```
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProperty], current ElementPath is [[configuration][springProperty]]
ERROR in ch.qos.logback.core.joran.spi.Interpreter@4:71 - no applicable action for [springProfile], current ElementPath is [[configuration][springProfile]]
```



### 4.7.1 Profile-specific 配置

通过`<springProfile>`标记，您可以根据激活的 Spring Profiles 有选择地引入或排除配置部分。在`<configuration>`元素内的任何位置都支持 Profile 部分。使用`name`属性指定哪个 profile 接受配置。`<springProfile>`标记可以包含简单的 profile 名称（例如，`staging`）或 profile 表达式。profile 表达式允许表达逻辑更复杂的 profile，例如`production & (eu-central | eu-west)`。有关更多详细信息，请参阅[参考指南](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java)。下面是三个 profile 的例子：

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```



### 4.7.2 `Environment`属性

`<springProperty>`标记使您可以从Spring `Environment`中暴露属性，以在 Logback 中使用。这样做有助于访问`application.properties`文件中 Logback 配置的值。该标签的工作方式类似于 Logback 的标准`<property>`标签。但是，您没有直接指定`value`，而是指定了属性的`source`（来自`Environment`）。如果需要将属性存储在`local`范围以外的其他位置，则可以使用`scope`属性。如果需要后备值（如果未在`Environment`中设置该属性），则可以使用`defaultValue`属性。以下示例显示如何在使用 Logback 中暴露的属性：

```xml
<springProperty scope="context" name="fluentHost" source="myapp.fluentd.host"
        defaultValue="localhost"/>
<appender name="FLUENT" class="ch.qos.logback.more.appenders.DataFluentAppender">
    <remoteHost>${fluentHost}</remoteHost>
    ...
</appender>
```

>[!note]
>
>必须用短横线指定`source`（例如`my.property-name`）。但是，可以使用宽松的规则将属性添加到`Environment`中。



# 5. 国际化

Spring Boot 支持本地化信息，因此您的应用程序可以迎合不同语言首选项的用户。默认情况下，Spring Boot 在类路径的根目录下查找`messages`资源包。

>[!note]
>
>当配置资源包的默认属性文件可用时，将应用自动配置（例如：默认`messages.properties`）。如果您的资源包仅包含特定于语言的属性文件，则需要添加默认文件。如果找不到与任何配置的基本名称匹配的属性文件，将没有自动配置的`MessageSource`。

可以使用`spring.messages`命名空间配置资源包的基本名称以及其他几个属性，如以下示例所示：

```properties
spring.messages.basename=messages,config.i18n.messages
spring.messages.fallback-to-system-locale=false
```

>[!tip]
>
>`spring.messages.basename`支持以逗号分隔的位置列表，可以是包限定符，也可以是从类路径根目录解析的资源。

有关更多支持的选项，请参见[`MessageSourceProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/context/MessageSourceProperties.java)。



# 6. JSON

Spring Boot 提供了与三个 JSON 映射库的集成：

- Gson
- Jackson
- JSON-B

Jackson 是首选的默认库。



## 6.1 Jackson

Spring Boot 提供了 Jackson 的自动配置，并且 Jackson 是`spring-boot-starter-json`的一部分。当 Jackson 放在类路径上时，将自动配置`ObjectMapper` Bean。提供了几个配置属性，用于[自定义ObjectMapper的配置](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto-customize-the-jackson-objectmapper)。



## 6.2 Gson

Spring Boot 提供了 Gson 的自动配置，当 Gson 在类路径上时，将自动配置`Gson` bean。提供了一些`spring.gson.*`配置属性用于自定义配置。为了获得更多控制权，可以使用一个或多个`GsonBuilderCustomizer` bean。



## 6.3 JSON-B

Spring Boot 提供了 JSON-B 的自动配置，当 JSON-B API 和其实现位于类路径上时，将自动配置`Jsonb` bean。首选的 JSON-B 实现是提供依赖管理的 Apache Johnzon。



# 7. 开发Web应用

Spring Boot 非常适合 Web 应用程序开发。您可以使用嵌入式 Tomcat，Jetty，Undertow 或 Netty 创建独立的 HTTP 服务器。大多数 Web 应用程序都使用`spring-boot-starter-web`模块来快速启动和运行。您还可以选择使用`spring-boot-starter-webflux`模块构建响应式 Web 应用程序。

如果您尚未开发过 Spring Boot Web 应用程序，则可以参考[入门指南](getting-started.md#4-开发你的第一个-spring-boot-应用)中的“ Hello World!”示例。



## 7.1 Spring Web MVC 框架

[Spring Web MVC 框架](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web.html#mvc)（通常简称为“ Spring MVC”）是一个优秀的“模型、视图、控制器” Web框架。Spring MVC 使您可以创建专门的`@Controller`或`@RestController` Bean 来处理传入的 HTTP 请求。使用`@RequestMapping`注解将控制器中的方法映射到 HTTP。

以下代码显示了提供 JSON 数据的典型`@RestController`：

```java
@RestController
@RequestMapping(value="/users")
public class MyRestController {

    @RequestMapping(value="/{user}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
    List<Customer> getUserCustomers(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}", method=RequestMethod.DELETE)
    public User deleteUser(@PathVariable Long user) {
        // ...
    }

}
```

Spring MVC 是核心 Spring 框架的一部分，有关详细信息，请参阅[参考文档](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web.html#mvc)。 在[spring.io/guides](https://spring.io/guides)上还有一些涵盖 Spring MVC 的指南。



### 7.1.1 Spring MVC 自动配置

Spring Boot 为 Spring MVC 提供了自动配置，可与大多数应用程序完美配合。

自动配置在 Spring 的默认设置之上添加了以下功能：

- 包含 `ContentNegotiatingViewResolver` 和`BeanNameViewResolver` bean
- 支持静态资源服务，包括对 WebJars 的支持（在本文档的[后面部分](spring-boot-features.md#715-静态内容)有介绍）
- 自动注册 Converter，GenericConverter 和 Formatter Bean
- 支持 `HttpMessageConverters`（在本文档的[后面部分](spring-boot-features.md#712-httpmessageconverters)有介绍）
- 自动注册`MessageCodesResolver` （在本文档的[后面部分](spring-boot-features.md#714-messagecodesresolver)有介绍）
- 支持静态`index.html` 
- 支持自定义`Favicon` （在本文档的[后面部分](spring-boot-features.md#717-自定义网站图标)有介绍）
- 自动使用`ConfigurableWebBindingInitializer` Bean（在本文档的[后面部分](spring-boot-features.md#719-configurablewebbindinginitializer)有介绍）

如果您想保留 Spring Boot MVC 功能，并且想要添加其他[MVC 配置](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web.html#mvc)（拦截器，格式化器，视图控制器和其他功能），则可以添加自己的类型为`WebMvcConfigurer`的`@Configuration`类，但不要添加`@EnableWebMvc`。如果希望提供`RequestMappingHandlerMapping`、`RequestMappingHandlerAdapter`或`ExceptionHandlerExceptionResolver`的自定义实例，则可以声明`WebMvcRegistrationsAdapter`实例以提供此类组件。

如果要完全控制 Spring MVC，则可以添加用`@EnableWebMvc`标注的自己的`@Configuration`类。



### 7.1.2 `HttpMessageConverters`

Spring MVC 使用`HttpMessageConverter`接口转换 HTTP 请求和响应。默认设置即是合理的，可以直接使用。例如，可以将对象自动转换为 JSON（通过使用 Jackson 库）或 XML（通过使用 Jackson XML 扩展（如果可用），或者通过使用 JAXB（如果 Jackson XML 扩展不可用））。默认情况下，字符串采用`UTF-8`编码。

如果你需要添加或自定义转换器，可以使用 Spring Boot 的`HttpMessageConverters`类，如下所示：

```java
import org.springframework.boot.autoconfigure.http.HttpMessageConverters;
import org.springframework.context.annotation.*;
import org.springframework.http.converter.*;

@Configuration(proxyBeanMethods = false)
public class MyConfiguration {

    @Bean
    public HttpMessageConverters customConverters() {
        HttpMessageConverter<?> additional = ...
        HttpMessageConverter<?> another = ...
        return new HttpMessageConverters(additional, another);
    }

}
```

上下文中存在的所有`HttpMessageConverter` bean 都将添加到转换器列表中。您也可以用相同的方法覆盖默认转换器。



### 7.1.3 自定义 JSON 序列化器和反序列化器

如果使用 Jackson 序列化和反序列化 JSON 数据，则可能要编写自己的`JsonSerializer`和`JsonDeserializer`类。自定义序列化程序通常是[通过模块注册 Jackson](https://github.com/FasterXML/jackson-docs/wiki/JacksonHowToCustomSerializers)的，但是 Spring Boot 提供了一种替代性的`@JsonComponent`注解，这使得直接注册 Spring Bean 更加容易。

您可以直接在`JsonSerializer`，`JsonDeserializer`或`KeyDeserializer`实现上使用`@JsonComponent`注解。您还可以在包含序列化器/反序列化器作为内部类的类上使用它，如以下示例所示：

```java
import java.io.*;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.*;
import org.springframework.boot.jackson.*;

@JsonComponent
public class Example {

    public static class Serializer extends JsonSerializer<SomeObject> {
        // ...
    }

    public static class Deserializer extends JsonDeserializer<SomeObject> {
        // ...
    }

}
```

`ApplicationContext`中的所有`@JsonComponent` bean都会自动注册 Jackson。因为`@JsonComponent`使用`@Component`进行元标注，所以通常的组件扫描规则都适用。

Spring Boot 还提供了[`JsonObjectSerializer`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectSerializer.java)和[`JsonObjectDeserializer`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/jackson/JsonObjectDeserializer.java)基类，这些基类在序列化对象时为标准 Jackson 版本提供了有用的替代方法。有关详细信息，请参见 Javadoc 中的[`JsonObjectSerializer`](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/api//org/springframework/boot/jackson/JsonObjectSerializer.html)和[`JsonObjectDeserializer`](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/api//org/springframework/boot/jackson/JsonObjectDeserializer.html)。



### 7.1.4 `MessageCodesResolver`

Spring MVC 有一个生成错误代码以从绑定错误中呈现错误消息的策略：`MessageCodesResolver`。如果设置`spring.mvc.message-codes-resolver-format`属性`PREFIX_ERROR_CODE`或`POSTFIX_ERROR_CODE`，Spring Boot 会为您创建一个（请参见[`DefaultMessageCodesResolver.Format`](https://docs.spring.io/spring/docs/5.2.2.RELEASE/javadoc-api/org/springframework/validation/DefaultMessageCodesResolver.Format.html)中的枚举）。



### 7.1.5 静态内容

默认情况下，Spring Boot 从类路径中的`/static`目录（或`/public`或`/resources`或`/META-INF/resources`）或`ServletContext`的根目录中提供静态内容。它使用 Spring MVC 中的`ResourceHttpRequestHandler`，以便您可以通过添加自己的`WebMvcConfigurer`并覆盖`addResourceHandlers`方法来修改对应的行为。

在独立的 Web 应用程序中，还启用了容器中的默认 Servlet，并将其用作后备，如果 Spring 决定不处理，则从`ServletContext`的根目录提供内容。但是在大多数情况下，这并不会发生（除非您修改默认的 MVC 配置），因为 Spring 始终可以通过`DispatcherServlet`处理请求。

默认情况下，资源映射在`/**`上，但是您可以使用`spring.mvc.static-path-pattern`属性进行调整。例如，将所有资源重定位到`/resources/**`可以按如下实现：

```properties
spring.mvc.static-path-pattern=/resources/**
```

您还可以使用`spring.resources.static-locations`属性来自定义静态资源位置（用目录位置列表替换默认值）。根`Servlet`上下文路径、`“/”`也会自动添加为位置。

除了前面提到的“标准”静态资源位置，也有一种特殊情况是·[Webjar 内容](https://www.webjars.org/)。如果 jar 文件以 Webjars 格式打包，则jar 文件中`/webjars/**`路径下的所有资源都会被提供。

>[!tip]
>
>如果您的应用程序打包为 jar，则不要使用`src/main/webapp`目录。尽管此目录是一个通用标准，但它仅与 打 war 包一起使用，并且如果生成 jar，大多数构建工具都将其忽略。

Spring Boot 还支持 Spring MVC 提供的高级资源处理功能，例如允许使用缓存清除（ache-busting）静态资源或对 Webjars 使用版本无关的 URL。

要使用版本无关的 Webjars URL，请添加`webjars-locator-core`依赖项。然后声明您的 Webjar。以 jQuery 为例，添加`"/webjars/jquery/jquery.min.js"`将得到`"/webjars/jquery/x.y.z/jquery.min.js"`，其中`x.y.z`是 Webjar 版本。

>[!note]
>
>如果使用 JBoss，则需要声明`webjars-locator-jboss-vfs`依赖关系，而不是`webjars-locator-core`。否则，所有 Webjar 都解析为`404`。

要使用缓存清除，以下配置为所有静态资源配置了缓存清除解决方案，实际上在 URL 中添加了内容哈希，例如`<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>`：

```properties
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
```

>[!note]
>
>借助为`Thymeleaf`和`FreeMarker`自动配置的`ResourceUrlEncodingFilter`，可以在运行时在模板中重写资源链接。使用JSP时，您应该手动声明此过滤器。当前不自动支持其他模板引擎，但可以与自定义模板宏/助手一起使用，以及使用`ResourceUrlProvider`。

例如，当使用 JavaScript 模块加载器动态加载资源时，不能重命名文件。这就是为什么其他策略也受支持并且可以组合的原因。“固定”策略在 URL 中添加静态版本字符串，而不更改文件名，如以下示例所示：

```properties
spring.resources.chain.strategy.content.enabled=true
spring.resources.chain.strategy.content.paths=/**
spring.resources.chain.strategy.fixed.enabled=true
spring.resources.chain.strategy.fixed.paths=/js/lib/
spring.resources.chain.strategy.fixed.version=v12
```

通过这种配置，位于`"/js/lib/"`下的 JavaScript 模块使用固定的版本控制策略（`"/v12/js/lib/mymodule.js"`），而其他资源仍使用满足的版本（`<link href="/css/spring-2a2d595e6ed9a0b24f027f2b63b134d6.css"/>`）。

有关更多受支持的选项，请参见[`ResourceProperties`](https://github.com/spring-projects/spring-boot/tree/v2.2.2.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ResourceProperties.java)。

>[!tip]
>
>该功能已在专门的[博客文章](https://spring.io/blog/2014/07/24/spring-framework-4-1-handling-static-web-resources)和 Spring Framework 的[参考文档](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web.html#mvc-config-static-resources)中进行了详细说明。



### 7.1.6 欢迎页面

Spring Boot 支持静态和模板欢迎页面。 它首先在配置的静态内容位置中查找`index.html`文件。如果未找到，则寻找`index`模板。如果找到任何一个，它将自动用作应用程序的欢迎页面。



### 7.1.7 自定义网站图标

与其他静态资源一样，Spring Boot 在已配置的静态内容位置中查找`favicon.ico`。如果存在这样的文件，它将自动用作应用程序的网站图标。



### 7.1.8 路径匹配和内容协商

Spring MVC 通过查看请求路径并将其匹配到应用程序中定义的映射（例如，Controller 方法上的`@GetMapping`注解）来将传入的 HTTP 请求映射到处理程序。

Spring Boot 默认选择禁用后缀模式匹配，这意味着`"GET /projects/spring-boot.json"`之类的请求将不会与`@GetMapping("/projects/spring-boot")`映射进行匹配。这被认为是[Spring MVC应用程序的最佳实践](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web.html#mvc-ann-requestmapping-suffix-pattern-match)。过去，此功能主要用于未发送正确的“Accept”请求头的 HTTP 客户端。我们需要确保将正确的内容类型发送给客户端。如今，内容协商已变得更加可靠。

还有其他处理 HTTP 客户端不能始终发送正确的“Accept”请求标头的方式。除了使用后缀匹配，我们还可以使用查询参数来确保将诸如`"GET /projects/spring-boot?format=json"`之类的请求映射到`@GetMapping("/projects/spring-boot")`：

```properties
spring.mvc.contentnegotiation.favor-parameter=true

# We can change the parameter name, which is "format" by default:
# spring.mvc.contentnegotiation.parameter-name=myparam

# We can also register additional file extensions/media types with:
spring.mvc.contentnegotiation.media-types.markdown=text/markdown
```

如果您了解了注意事项，但仍希望您的应用程序使用后缀模式匹配，则需要以下配置：

```properties
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-suffix-pattern=true
```

另外，与其打开所有后缀模式，不如只支持已注册的后缀模式，这更安全：

```properties
spring.mvc.contentnegotiation.favor-path-extension=true
spring.mvc.pathmatch.use-registered-suffix-pattern=true

# You can also register additional file extensions/media types with:
# spring.mvc.contentnegotiation.media-types.adoc=text/asciidoc
```



### 7.1.9 `ConfigurableWebBindingInitializer`

Spring MVC 使用`WebBindingInitializer`初始化特定请求的`WebDataBinder`。如果创建自己的`ConfigurableWebBindingInitializer` `@Bean`，Spring Boot 会自动配置 Spring MVC 以使用它。



### 7.1.10 模板引擎

除了 REST Web 服务之外，您还可以使用 Spring MVC 来提供动态 HTML 内容。Spring MVC 支持各种模板技术，包括 Thymeleaf、FreeMarker 和 JSP。同样，许多其他模板引擎包括它们自己的 Spring MVC 集成。

Spring Boot 支持对以下模板引擎自动配置：

- [FreeMarker](https://freemarker.apache.org/docs/)
- [Groovy](http://docs.groovy-lang.org/docs/next/html/documentation/template-engines.html#_the_markuptemplateengine)
- [Thymeleaf](https://www.thymeleaf.org/)
- [Mustache](https://mustache.github.io/)

>[!tip]
>
>如果可能，应避免使用 JSP。将它们与嵌入式servlet容器一起使用时，存在几个[已知的限制](spring-boot-features.md#745-jsp-限制)。

在默认配置下使用这些模板引擎之一时，将从`src/main/resources/templates`中自动提取模板。

>[!tip]
>
>根据您运行应用程序的方式，IntelliJ IDEA 对类路径的排序有不同方式。使用 Maven 或 Gradle 或从打包的 jar 运行应用程序，与从 IDE 运行应用程序的主方法顺序会有所不同。这可能会导致 Spring Boot 无法在类路径上找到模板。如果遇到此问题，可以在IDE中重新排序类路径，以首先放置模块的类和资源。或者，您可以配置模板前缀以搜索类路径上的每个`templates`目录，如下所示：`classpath*:/templates/`。



### 7.1.11 错误处理

默认情况下，Spring Boot 提供了一个`/error`映射，以一种合理的方式处理所有错误，并且在 servlet 容器中被注册为“全局”错误页面。对于机器客户端，它将生成 JSON 响应，其中包含错误，HTTP 状态码和异常消息的详细信息。对于浏览器客户端，有一个“ whitelabel”错误视图以 HTML 格式呈现相同的数据（如需对其进行自定义，请添加一个可处理`error`的`view`）。要完全替换默认行为，可以实现`ErrorController`并注册该类型的bean定义，或者添加类型为`ErrorAttributes`的 bean 以使用现有机制但替换其内容。

>[!tip]
>
>`BasicErrorController`可用作自定义`ErrorController`的基类。如果要为新的内容类型添加处理程序（默认是专门处理`text/html`并为其他所有内容提供后备功能），则此功能特别有用。为此，请扩展`BasicErrorController`，添加具有`@produceMapping`且具有`Produces`属性的公共方法，并创建新类型的 bean。

您还可以定义一个用`@ControllerAdvice`标注的类，以自定义 JSON 文档以针对特定的控制器和（或）异常类型返回，如以下示例所示：

```java
@ControllerAdvice(basePackageClasses = AcmeController.class)
public class AcmeControllerAdvice extends ResponseEntityExceptionHandler {

    @ExceptionHandler(YourException.class)
    @ResponseBody
    ResponseEntity<?> handleControllerException(HttpServletRequest request, Throwable ex) {
        HttpStatus status = getStatus(request);
        return new ResponseEntity<>(new CustomErrorType(status.value(), ex.getMessage()), status);
    }

    private HttpStatus getStatus(HttpServletRequest request) {
        Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
        if (statusCode == null) {
            return HttpStatus.INTERNAL_SERVER_ERROR;
        }
        return HttpStatus.valueOf(statusCode);
    }

}
```

在前面的示例中，如果与`AcmeController`在同一包中定义的 controller 抛出`YourException`，则将使用`CustomErrorType` POJO 的 JSON 表示形式而不是`ErrorAttributes`表示形式。



#### 自定义错误页面

如果要显示给定状态码的自定义 HTML 错误页面，可以将文件添加到`/error`文件夹。错误页面可以是静态 HTML（即添加到任何静态资源文件夹下），也可以使用模板来构建。文件名应为确切的状态代码或系列掩码。

例如，要将`404`映射到静态 HTML 文件，您的文件夹结构需要如下这样：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- public/
             +- error/
             |   +- 404.html
             +- <other public assets>
```

要使用 FreeMarker 模板映射所有`5xx`错误，您的文件夹结构如下：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.ftlh
             +- <other templates>
```

对于更复杂的映射，还可以添加实现`ErrorViewResolver`接口的 bean，如以下示例所示：

```java
public class MyErrorViewResolver implements ErrorViewResolver {

    @Override
    public ModelAndView resolveErrorView(HttpServletRequest request,
            HttpStatus status, Map<String, Object> model) {
        // Use the request or status to optionally return a ModelAndView
        return ...
    }

}
```

您还可以使用常规的 Spring MVC 功能，例如[`@ExceptionHandler`方法](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web.html#mvc-exceptionhandlers)和[`@ControllerAdvice`](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web.html#mvc-ann-controller-advice)。 然后，`ErrorController`处理所有未处理的异常。



#### 在 Spring MVC 外部映射错误页面

对于不使用 Spring MVC 的应用程序，可以使用`ErrorPageRegistrar`接口直接注册`ErrorPages`。此抽象直接与底层的嵌入式 servlet 容器一起使用，即使您没有 Spring MVC `DispatcherServlet`，它也可以使用。

```java
@Bean
public ErrorPageRegistrar errorPageRegistrar(){
    return new MyErrorPageRegistrar();
}

// ...

private static class MyErrorPageRegistrar implements ErrorPageRegistrar {

    @Override
    public void registerErrorPages(ErrorPageRegistry registry) {
        registry.addErrorPages(new ErrorPage(HttpStatus.BAD_REQUEST, "/400"));
    }

}
```

>[!note]
>
>如果您注册了一个最终由`Filter`处理的路径的`ErrorPage`（这在某些非 Spring Web 框架（如 Jersey 和 Wicket ）中很常见），则必须将`Filter`显式注册为`ERROR`调度器，如 下面的例子：

```java
@Bean
public FilterRegistrationBean myFilter() {
    FilterRegistrationBean registration = new FilterRegistrationBean();
    registration.setFilter(new MyFilter());
    ...
    registration.setDispatcherTypes(EnumSet.allOf(DispatcherType.class));
    return registration;
}
```

请注意，默认的`FilterRegistrationBean`不包含`ERROR`调度器类型。

警告：当部署到 servlet 容器时，Spring Boot 使用其错误页面过滤器将具有错误状态的请求转发到适当的错误页面。如果尚未提交响应，则只能将请求转发到正确的错误页面。默认情况下，WebSphere Application Server 8.0及更高版本在成功完成 servlet 的 service 方法后提交响应。要想禁用此行为可以通过将`com.ibm.ws.webcontainer.invokeFlushAfterService`设置为`false`。



### 7.1.12 Spring HATEOAS

如果您开发使用超媒体的 RESTful API，Spring Boot 会为 Spring HATEOAS 提供自动配置，该配置可与大多数应用程序很好地兼容。自动配置取代了使用`@EnableHypermediaSupport`的需要，并注册了一些 bean 来简化基于超媒体的应用程序的构建，其中包括`LinkDiscoverers`（用于客户端支持）和`ObjectMapper`，配置是为了将响应正确地编组为所需的表示形式。通过设置各种`spring.jackson.*`属性，或通过`Jackson2ObjectMapperBuilder` bean（如果存在）来定制`ObjectMapper`。

您可以使用`@EnableHypermediaSupport`来控制 Spring HATEOAS 的配置。请注意，这样做会禁用前面所述的`ObjectMapper`定制。



### 7.1.13 CORS 支持

[跨域资源共享](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)（CORS）是由[大多数浏览器](https://caniuse.com/#feat=cors)实现的[W3C 规范](https://www.w3.org/TR/cors/)，使您可以灵活地指定对哪种类型的跨域请求进行授权，而不是使用诸如 IFRAME 或 JSONP 之类的安全性较差的方法。

从4.2版本开始，Spring MVC [支持 CORS](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web.html#mvc-cors)。在 Spring Boot 应用程序中使用带有`@CrossOrigin`注解的[控制器方法 CORS 配置](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web.html#mvc-cors-controller)不需要任何特定的配置。可以通过注册一个带有自定义的`addCorsMappings(CorsRegistry)`方法的`WebMvcConfigurer` bean 来定义[全局 CORS 配置](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web.html#mvc-cors-global)，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
public class MyConfiguration {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**");
            }
        };
    }
}
```



## 7.2 “Spring WebFlux Framework”

Spring WebFlux 是 Spring 框架 5.0 中引入的新的响应式 Web 框架。 与 Spring MVC 不同，它不需要Servlet API，是完全异步且无阻塞的，并通过[Reactor项目](https://projectreactor.io/)实现[Reactive Streams](https://www.reactive-streams.org/)规范。

Spring WebFlux有两种形式：函数式的的和基于注解的。基于注解的非常类似于 Spring MVC 模型，如以下示例所示：

```java
@RestController
@RequestMapping("/users")
public class MyRestController {

    @GetMapping("/{user}")
    public Mono<User> getUser(@PathVariable Long user) {
        // ...
    }

    @GetMapping("/{user}/customers")
    public Flux<Customer> getUserCustomers(@PathVariable Long user) {
        // ...
    }

    @DeleteMapping("/{user}")
    public Mono<User> deleteUser(@PathVariable Long user) {
        // ...
    }

}
```

函数式变体“ WebFlux.fn”将路由配置与请求的实际处理分开，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
public class RoutingConfiguration {

    @Bean
    public RouterFunction<ServerResponse> monoRouterFunction(UserHandler userHandler) {
        return route(GET("/{user}").and(accept(APPLICATION_JSON)), userHandler::getUser)
                .andRoute(GET("/{user}/customers").and(accept(APPLICATION_JSON)), userHandler::getUserCustomers)
                .andRoute(DELETE("/{user}").and(accept(APPLICATION_JSON)), userHandler::deleteUser);
    }

}

@Component
public class UserHandler {

    public Mono<ServerResponse> getUser(ServerRequest request) {
        // ...
    }

    public Mono<ServerResponse> getUserCustomers(ServerRequest request) {
        // ...
    }

    public Mono<ServerResponse> deleteUser(ServerRequest request) {
        // ...
    }
}
```

WebFlux 是 Spring 框架的一部分，其[参考文档](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web-reactive.html#webflux-fn)中提供了详细信息。

>[!tip]
>
>您可以根据需要定义任意数量的`RouterFunction` bean，以对路由器的定义进行模块化。如果需要应用优先级，可以通过 Bean 控制。

首先，将`spring-boot-starter-webflux`模块添加到您的应用程序。

>[!note]
>
>在应用程序中添加`spring-boot-starter-web`和`spring-boot-starter-webflux`模块会导致 Spring Boot 自动配置 Spring MVC，而不是 WebFlux。之所以选择这种行为，是因为许多 Spring 开发人员将`spring-boot-starter-webflux`添加到其 Spring MVC 应用程序中以使用响应式 WebClient。您仍然可以通过将所选应用程序类型设置为`SpringApplication.setWebApplicationType(WebApplicationType.REACTIVE)`来强制执行你的选择。



### 7.2.1 Spring WebFlux 自动配置

Spring Boot 为 Spring WebFlux 提供了自动配置，可与大多数应用程序很好地配合使用。

自动配置在 Spring 的默认设置之上添加了以下功能：

* 为`HttpMessageReader`和`HttpMessageWriter`接口配置解码器（在[本文档后面](spring-boot-features.md#722-带有httpmessagereaders和httpmessagewriters的http解码器)介绍）。
* 支持提供静态资源服务，包括对 WebJars 的支持（在[本文档后面](spring-boot-features.md#723-静态内容)介绍）。

如果您想保留 Spring Boot WebFlux 功能并想要添加其他 WebFlux 配置，则可以添加自己的类型为`WebFluxConfigurer`的`@Configuration`类，但**不要**添加`@EnableWebFlux`。

如果要完全控制 Spring WebFlux，则可以添加自己的使用`@EnableWebFlux`注解的`@Configuration`类。



### 7.2.2 带有HttpMessageReaders和HttpMessageWriters的HTTP解码器

Spring WebFlux 使用`HttpMessageReader`和`HttpMessageWriter`接口来转换 HTTP 请求和响应。它们配置了`CodecConfigurer`，通过查看类路径中可用的库，可以得到合理的默认值。

Spring Boot 为解码器提供了专用的配置属性`spring.codec.*`。它还通过使用`CodecCustomizer`接口来应用进一步的自定义。例如，将`spring.jackson.*`配置 key 应用于 Jackson 解码器。

如果需要添加或自定义解码器，则可以创建一个自定义`CodecCustomizer`组件，如以下示例所示：

```java
import org.springframework.boot.web.codec.CodecCustomizer;

@Configuration(proxyBeanMethods = false)
public class MyConfiguration {

    @Bean
    public CodecCustomizer myCodecCustomizer() {
        return codecConfigurer -> {
            // ...
        };
    }

}
```

您还可以利用 [Boot的自定义 JSON 序列化器和反序列化器](spring-boot-features.md#713-自定义-json-序列化器和反序列化器)。



### 7.2.3 静态内容

默认情况下，Spring Boot 从类路径中名为`/static`（或`/public`或`/resources`或`/META-INF/resources`）的目录中提供静态内容。它使用 Spring WebFlux 中的`ResourceWebHandler`，以便您可以通过添加自己的`WebFluxConfigurer`并覆盖`addResourceHandlers`方法来修改该行为。

默认情况下，资源映射在`/**`上，但是您可以通过设置`spring.webflux.static-path-pattern`属性来对其进行调整。例如，将所有资源重定位到`/resources/**`的实现如下：

```properties
spring.webflux.static-path-pattern=/resources/**
```

您还可以使用`spring.resources.static-locations`自定义静态资源的位置。这样做会将默认的值替换为一个目录位置列表。如果这样做，默认的欢迎页面检测将切换到您的自定义位置。因此，如果启动时您的任意自定义位置有`index.html`，则它将作为应用程序的主页。

除了前面列出的“标准”静态资源位置外，有一种特殊情况是[Webjar 内容](https://www.webjars.org/)。任何在`/webjars/**`路径下的资源都可以从 jar 文件中获得，前提是它们是以 Webjars 格式打包的。

>[!tip]
>
>Spring WebFlux 应用程序不严格依赖 Servlet API，因此不能将它们部署为 war 文件，也不使用`src/main/webapp`目录。



### 7.2.4 模板引擎

除了 REST Web 服务之外，您还可以使用 Spring WebFlux 来提供动态 HTML 内容。Spring WebFlux 支持各种模板技术，包括 Thymeleaf，FreeMarker 和 Mustache。

Spring Boot 为以下模板引擎提供了自动配置支持;

- [FreeMarker](https://freemarker.apache.org/docs/)
- [Thymeleaf](https://www.thymeleaf.org/)
- [Mustache](https://mustache.github.io/)

当您使用这些带有默认配置的模板引擎之一时，您的模板将自动从`src/main/resources/templates`中获取。



### 7.2.5 错误处理

Spring Boot 提供了一个`WebExceptionHandler`，以一种合理的方式处理所有错误。它在处理顺序中的位置直接在 WebFlux 提供的 handler 之前，被认为是最后一个。对于机器客户端，它将生成一个 JSON 响应，其中包含错误的详细信息、HTTP 状态和异常消息。对于浏览器客户端，是一个“ whitelabel”错误 handler，以 HTML 格式呈现相同的数据。您也可以提供自己的 HTML 模板展示错误信息（参考[下一小节](spring-boot-features.md#自定义错误页面_1)）。

定制此功能的第一步通常包括使用现有机制，而不是替换或增加错误内容。 为此，您可以添加类型为`ErrorAttributes`的 bean。

要更改错误 handler 的行为，可以实现`ErrorWebExceptionHandler`并注册该类型的 bean 定义。由于`WebExceptionHandler`的级别很低，因此 Spring Boot 还提供了一个方便的`AbstractErrorWebExceptionHandler`，可让您以 WebFlux 函数的方式处理错误，如以下示例所示：

```java
public class CustomErrorWebExceptionHandler extends AbstractErrorWebExceptionHandler {

    // Define constructor here

    @Override
    protected RouterFunction<ServerResponse> getRoutingFunction(ErrorAttributes errorAttributes) {

        return RouterFunctions
                .route(aPredicate, aHandler)
                .andRoute(anotherPredicate, anotherHandler);
    }

}
```

为了获得更完整的描述，您还可以直接创建`DefaultErrorWebExceptionHandler`的子类并重写特定方法。



#### 自定义错误页面

如果要显示给定状态码的自定义 HTML 错误页面，可以将文件添加到`/error`文件夹。错误页面可以是静态 HTML（即可以添加到任何静态资源文件夹下），也可以使用模板构建。文件名应为确切的状态代码或系列掩码。

例如，要将`404`映射到静态 HTML 文件，您的文件夹结构如下：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- public/
             +- error/
             |   +- 404.html
             +- <other public assets>
```

要使用 Mustache 模板映射所有`5xx`错误，您的文件夹结构应该如下所示：

```
src/
 +- main/
     +- java/
     |   + <source code>
     +- resources/
         +- templates/
             +- error/
             |   +- 5xx.mustache
             +- <other templates>
```



### 7.2.6 Web 过滤器

Spring WebFlux 提供了一个`WebFilter`接口，可以通过实现该接口来过滤 HTTP 请求-响应交换。在应用程序上下文中找到的`WebFilter` bean 将自动用于过滤每个交换。

如果过滤器的顺序很重要，则可以实现`Ordered`或使用`@Order`进行注释。Spring Boot 自动配置可能会为您配置 Web 过滤器。这样做时，将使用下表中显示的顺序：

| Web 过滤器                              | 顺序                             |
| :-------------------------------------- | :------------------------------- |
| `MetricsWebFilter`                      | `Ordered.HIGHEST_PRECEDENCE + 1` |
| `WebFilterChainProxy` (Spring Security) | `-100`                           |
| `HttpTraceWebFilter`                    | `Ordered.LOWEST_PRECEDENCE - 10` |



## 7.3 JAX-RS 和 Jersey

如果您更喜欢 REST 端点的 JAX-RS 编程模型，则可以使用某个可用的实现来代替 Spring MVC。[Jersey](https://jersey.github.io/)和[Apache CXF](https://cxf.apache.org/)可以开箱即用。CXF 要求您在应用程序上下文中将其`Servlet`或`Filter`注册为`@Bean`。Jersey 提供了一些本地的 Spring 支持，因此我们在 Spring Boot 中还与启动器一起为其提供了自动配置支持。

要开始使用 Jersey，请将`spring-boot-starter-jersey`作为依赖项引入，然后需要一个`ResourceConfig`类型的`@Bean`，在其中注册所有端点，如以下示例所示：

```java
@Component
public class JerseyConfig extends ResourceConfig {

    public JerseyConfig() {
        register(Endpoint.class);
    }

}
```

>[!warning]
>
> Jersey 对扫描可执行文档的支持非常有限。例如，在运行可执行的war文件时，它无法扫描在[完全可执行的jar文件](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/deployment.html#deployment-install)或`WEB-INF/classs`中找到的包中的端点。 为了避免这种限制，不应该使用`packages`方法，并且应该使用`register`方法分别注册端点，如前面的示例所示。

对于更高级的定制，您还可以注册任意数量的实现了`ResourceConfigCustomizer`的 bean。

所有注册的端点应为具有 HTTP 资源（`@GET`和其他注解）注解的`@Components`，如以下示例所示：

```java
@Component
@Path("/hello")
public class Endpoint {

    @GET
    public String message() {
        return "Hello";
    }

}
```

由于`Endpoint`是Spring 的`@Component`，因此其生命周期由 Sprin g管理，您可以使用`@Autowired`注解注入依赖，并使用`@Value`注解注入外部配置。默认情况下，Jersey servlet 被注册并映射到`/*`。您可以通过将`@ApplicationPath`添加到`ResourceConfig`上来更改映射。

默认情况下，Jersey 在名为`ServletServletRegistration`的`ServletRegistrationBean`类型的`@Bean`中设置为 Servlet。默认情况下，该 Servlet 是懒加载的，但是您可以通过设置`spring.jersey.servlet.load-on-startup`来自定义该行为。您可以通过创建自己的同名 bean 之一来禁用或覆盖该 bean。您还可以通过设置`spring.jersey.type=filter`来使用过滤器而不是 servlet（在这种情况下，要替换或覆盖的`@Bean`是`jerseyFilterRegistration`）。过滤器具有`@Order`，您可以使用`spring.jersey.filter.order`进行设置。可以通过使用`spring.jersey.init.*`来指定属性映射，从而为 servlet 和过滤器注册赋予初始化参数。



## 7.4 嵌入式 Servlet 容器支持

Spring Boot 包括对嵌入式[Tomcat](https://tomcat.apache.org/)，[Jetty](https://www.eclipse.org/jetty/)和[Undertow](https://github.com/undertow-io/undertow)服务器的支持。大多数开发人员使用适当的“启动器”来获取完全配置好的实例。默认情况下，嵌入式服务器在端口`8080`上监听 HTTP 请求。



### 7.4.1 Servlet、过滤器、监听器

使用嵌入式 Servlet 容器时，可以通过使用 Spring Bean或扫描 Servlet 组件来注册 Servlet、过滤器和 Servlet 规范下的所有监听器（例如`HttpSessionListener`）。



#### 将Servlet、过滤器和监听器注册为 Spring Bean

所有`Servlet`、`Filter`或 servlet `*Listener`实例都作为 Spring Bean 向嵌入式容器注册。如果要在配置过程中引用`application.properties`中的值，这可能特别方便。

默认情况下，如果上下文仅包含单个 Servlet，则将其映射到`/`。对于多个 servlet bean，bean 名称用作路径前缀。 过滤器映射到`/*`。

如果基于约定的映射不够灵活，则可以使用`ServletRegistrationBean`、`FilterRegistrationBean`和`ServletListenerRegistrationBean`类进行完全控制。

过滤器 bean 处于无序状态通常是安全的。如果需要特定的顺序，则应使用`@Order`标注`Filter`或使其实现`Ordered`。您不能通过使用`@Order`标注`Filter`的 bean 方法来配置`Filter`的顺序。

如果您不能更改`Filter`类以添加`@Order`或实现`Ordered`，则必须为`Filter`定义一个`FilterRegistrationBean`并使用`setOrder(int)`方法设置注册 bean 的顺序。

应避免配置一个在`Ordered.HIGHEST_PRECEDENCE`上读取 request body 的过滤器，因为它可能与应用程序的字符编码配置不符。如果 Servlet 过滤器包装了请求，则应使用小于或等于`OrderedFilter.REQUEST_WRAPPER_FILTER_MAX_ORDER`的顺序来配置它。

>[!tip]
>
>要查看应用程序中每个过滤器的顺序，请为`web`[日志组](spring-boot-features.md#45-日志组)（`logging.level.web=debug`）启用 debug 级别的日志记录。然后，将在启动时记录已注册过滤器的详细信息，包括其顺序和URL模式。

<span></span>



>[!warning]
>
>注册`Filter` Bean 时要小心，因为它们是在应用程序生命周期中很早就初始化的。如果需要注册与其他 bean 交互的`Filter`，请考虑改用`DelegatingFilterProxyRegistrationBean`。



### 7.4.2 Servlet 上下文初始化

嵌入式 Servlet 容器不会直接执行 Servlet 3.0+ `javax.servlet.ServletContainerInitializer`接口或 Spring 的`org.springframework.web.WebApplicationInitializer`接口。这是一个有意的设计决定，旨在降低在 war 包中运行的第三方库可能破坏 Spring Boot 应用程序的风险。

如果您需要在 Spring Boot 应用程序中执行 Servlet 上下文初始化，则应该注册一个实现了`org.springframework.boot.web.servlet.ServletContextInitializer`接口的 bean。单个`onStartup`方法提供对`ServletContext`的访问，并且在必要时可以轻松地用作现有`WebApplicationInitializer`的适配器。



#### 扫描 Servlet、过滤器和监听器

使用嵌入式容器时，可以使用`@ServletComponentScan`来启用自动注册带有`@WebServlet`、`@WebFilter`和`@WebListener`的类。

>[!tip]
>
>`@ServletComponentScan`在独立容器中不生效，而是使用容器内置的发现机制。



### 7.4.3 ServletWebServerApplicationContext

在后台，Spring Boot 使用另一种类型的`ApplicationContext`来支持嵌入式Servlet容器。`ServletWebServerApplicationContext`是`WebApplicationContext`的一种特殊类型，它通过搜索单个`ServletWebServerFactory` bean来启动自己。通常，已经自动配置了`TomcatServletWebServerFactory`、`JettyServletWebServerFactory`或`UndertowServletWebServerFactory`。

>[!note]
>
>通常，您不需要了解这些实现类。大多数应用程序都是自动配置的，并且代表您创建了相应的`ApplicationContext`和`ServletWebServerFactory`。



### 7.4.4 自定义嵌入式 Servlet 容器

可以使用 Spring `Environment`属性来配置常见的 servlet 容器设置。通常，您可以在`application.properties`文件中定义这些属性。

常用服务器设置包括：

* 网络设置：监听 HTTP 请求的端口（`server.port`），绑定到`server.address`的接口地址，等等。

* Session 设置：Session 是否持久化（`server.servlet.session.persistent`）、Session 超时（`server.servlet.session.timeout`）、Session 数据的位置（`server.servlet.session.store-dir`）和 session-cookie 配置（`server.servlet.session.cookie.*`）。
* 错误管理：错误页面的位置（`server.error.path`）等等
* [SSL](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto-configure-ssl)
* [HTTP 压缩](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#how-to-enable-http-response-compression)

Spring Boot 尝试尽可能多地暴露通用设置，但这并不总是可以的。对于这些场景，专用命名空间提供特定于服务器的定制（参考`server.tomcat`和`server.undertow`）。例如，可以使用嵌入式 servlet 容器的特定功能配置[访问日志](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/howto.html#howto-configure-accesslogs)。

>[!tip]
>
>有关完整清单，请参见`ServerProperties`类。



#### 程序化定制

如果需要以编程方式配置嵌入式 Servlet 容器，则可以注册一个实现了`WebServerFactoryCustomizer`接口的Spring Bean。`WebServerFactoryCustomizer`提供对`ConfigurableServletWebServerFactory`的访问，其中包括许多自定义的 setter 方法。以下示例展示了以编程方式设置端口：

```java
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.stereotype.Component;

@Component
public class CustomizationBean implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {

    @Override
    public void customize(ConfigurableServletWebServerFactory server) {
        server.setPort(9000);
    }

}
```

>[!note]
>
>`TomcatServletWebServerFactory`、`JettyServletWebServerFactory`和`UndertowServletWebServerFactory`是`ConfigurableServletWebServerFactory`的专用变体，分别具有针对 Tomcat，Jetty 和 Undertow 的其他自定义的 setter 方法。



#### 直接自定义ConfigurableServletWebServerFactory

如果上述定制技术有太多限制，则可以自己注册`TomcatServletWebServerFactory`、`JettyServletWebServerFactory`或`UndertowServletWebServerFactory` bean。

```java
@Bean
public ConfigurableServletWebServerFactory webServerFactory() {
    TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
    factory.setPort(9000);
    factory.setSessionTimeout(10, TimeUnit.MINUTES);
    factory.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/notfound.html"));
    return factory;
}
```

有许多配置选项的 setter。如果您需要做一些更特殊的操作，还提供了几种受保护的方法“hooks”。有关详细信息，请参见[源代码文档](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/api//org/springframework/boot/web/servlet/server/ConfigurableServletWebServerFactory.html)。



### 7.4.5 JSP 限制

运行使用嵌入式 servlet 容器（并打包为可执行包）的 Spring Boot 应用程序时，JSP 支持存在一些限制。

* 对于 Jetty 和 Tomcat，如果使用 war 包，它应该可以工作。可执行的 war 使用`java -jar`启动时，它将正常工作，并且还可部署到任何标准容器中。使用可执行 jar 时，不支持 JSP。
* Undertow 不支持 JSP。
* 创建自定义的`error.jsp`页面不会覆盖默认视图以进行错误处理，应改用[自定义错误页面](spring-boot-features.md#自定义错误页面)。



## 7.5 嵌入式的响应式服务器支持

Spring Boot 包含对以下嵌入式的响应式 Web 服务器的支持：Reactor Netty、Tomcat、Jetty 和 Undertow。大多数开发人员使用适当的“启动器”来获取完全配置的实例。默认情况下，嵌入式服务器在端口 8080 上监听 HTTP 请求。



## 7.6 响应式服务器资源配置

当自动配置 Reactor Netty 或 Jetty 服务器时，Spring Boot 将创建特定的 bean：`ReactorResourceFactory`或`JettyResourceFactory`，这些 bean 将向服务器实例提供 HTTP 资源。

默认情况下，这些资源还将与 Reactor Netty 和 Jetty 客户端共享，以实现最佳性能，前提是：

* 服务器和客户端使用相同的技术
* 客户端实例是使用 Spring Boot 自动配置的`WebClient.Builder` bean构建的

通过提供自定义的`ReactorResourceFactory`或`JettyResourceFactory` bean，开发人员可以覆盖 Jetty 和 Reactor Netty 的资源配置，这将同时应用于客户端和服务器。

您可以在[WebClient 运行时间](spring-boot-features.md#151-webclient-运行时间)部分了解有关客户端资源配置的更多信息。



# 8. RSocket

[RSocket](https://rsocket.io/)是用于字节流传输的二进制协议。它通过在单个连接上传递异步消息来支持对称交互模型。

Spring 框架的`spring-messaging`模块在客户端和服务器端都提供了对 RSocket 请求者和响应者的支持。有关更多详细信息，请参见 Spring 框架参考中的[RSocket 部分](https://docs.spring.io/spring/docs/5.2.2.RELEASE/spring-framework-reference/web-reactive.html#rsocket-spring)，其中包括 RSocket 协议的概述。



## 8.1 RSocket 策略自动配置

Spring Boot 自动配置一个`RSocketStrategies` bean，该 bean 提供了编码和解码 RSocket 有效负载所需的所有基础结构。默认情况下，自动配置将尝试（按顺序）配置以下内容：

1. 使用 Jackson 的[CBOR](https://cbor.io/)解码器
2. 使用 Jackson 的 JSON 解码器

`spring-boot-starter-socket`启动器提供了所有的依赖项。查阅[Jackson 支持](spring-boot-features.md#61-jackson)章节，以了解有关定制可能性的更多信息。

开发人员可以通过创建实现了`RSocketStrategiesCustomizer`接口的 bean 来自定义`RSocketStrategies`组件。请注意，它们的`@Order`很重要，因为它决定解码器的顺序。



## 8.2 RSocket 服务器自动配置

Spring Boot 提供了 RSocket 服务器自动配置，所需的依赖由`spring-boot-starter-rsocket`提供。

Spring Boot 允许从 WebFlux 服务器通过 WebSocket 暴露 RSocket，或支持独立的 RSocket 服务器。这取决于应用程序的类型及其配置。

对于 WebFlux 应用程序（例如`WebApplicationType.REACTIVE`类型），仅当以下属性匹配时，RSocket 服务器才会添加到 Web 服务器：

```properties
spring.rsocket.server.mapping-path=/rsocket # a mapping path is defined
spring.rsocket.server.transport=websocket # websocket is chosen as a transport
#spring.rsocket.server.port= # no port is defined
```

>[!warning]
>
>只有 Reactor Netty 支持将 RSocket 添加到 Web 服务器，因为 RSocket本身是使用该库构建的。

另外，RSocket TCP 或 websocket 服务器也可以作为独立的嵌入式服务器启动。除了对依赖的要求之外，唯一需要的配置是为该服务器定义端口：

```properties
spring.rsocket.server.port=9898 # the only required configuration
spring.rsocket.server.transport=tcp # you're free to configure other properties
```



## 8.3 Spring Messaging RSocket 支持

Spring Boot 将为 RSocket 自动配置 Spring Messaging 基础结构。

这意味着 Spring Boot 将创建一个`RSocketMessageHandler` bean，该 bean 将处理您的应用程序收到的 RSocket 请求。



## 8.4 使用`RSocketRequester`调用 RSocket 服务

在服务器和客户端之间建立`RSocket`通道后，任何一方都可以向另一方发送或接收请求。

在服务器端，您可以在RSocket `@Controller`的任何处理方法上注入`RSocketRequester`实例。在客户端，您需要首先配置和建立 RSocket 连接。在这种情况下，Spring Boot 会使用期望的解码器自动配置`RSocketRequester.Builder`。

`RSocketRequester.Builder`实例是一个原型 bean，这意味着每个注入点将为您提供一个新实例。这样做是有目的的，因为此构建器是有状态的，因此您不应使用同一实例创建具有不同设置的请求者。

下面的代码展示了一个典型的例子：

```java
@Service
public class MyService {

    private final RSocketRequester rsocketRequester;

    public MyService(RSocketRequester.Builder rsocketRequesterBuilder) {
        this.rsocketRequester = rsocketRequesterBuilder
                .connectTcp("example.org", 9898).block();
    }

    public Mono<User> someRSocketCall(String name) {
        return this.requester.route("user").data(name)
                .retrieveMono(User.class);
    }

}
```



# 9. 安全

如果[Spring Security](https://spring.io/projects/spring-security)在类路径上，则默认情况下 Web 应用程序是安全的。Spring Boot 依靠 Spring Security 的内容协商策略来确定是使用`httpBasic`还是`formLogin`。要将方法级安全性添加到 Web 应用程序，还可以使用所需的设置添加`@EnableGlobalMethodSecurity`。可以在[Spring Security参考指南](https://docs.spring.io/spring-security/site/docs/5.2.1.RELEASE/reference/htmlsingle/#jc-method)中找到更多信息。

默认的`UserDetailsService`具有一个用户。用户名是`user`，密码是随机的，并在应用程序启动时以 INFO 级别显示，如下例所示：

```
Using generated security password: 78fa095d-3f4c-48b1-ad50-e24c31d5cf35
```

>[!note]
>
>如果您微调日志记录配置，请确保将`org.springframework.boot.autoconfigure.security`类别设置为记录`INFO`级别的消息。否则，不会打印默认密码。

您可以通过提供`spring.security.user.name`和`spring.security.user.password`来更改用户名和密码。

默认情况下，您在 Web 应用程序中获得的基本功能是：

* 具有内存存储的`UserDetailsService`（如果是 WebFlux 应用程序，则为`ReactiveUserDetailsService`）bean 和具有生成的密码的单个用户（有关用户的属性，请参阅[SecurityProperties.User](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/api//org/springframework/boot/autoconfigure/security/SecurityProperties.User.html)）。
* 整个应用程序的基于表单的登录或 HTTP 基本安全性（取决于请求中的`Accept` header）（如果执行器位于类路径上，则包括执行器端点）。
* 用于发布身份验证事件的`DefaultAuthenticationEventPublisher`。

您可以通过添加一个 bean 来提供另一个`AuthenticationEventPublisher`。



## 9.1 MVC 安全

默认的安全配置是在`SecurityAutoConfiguration`和`UserDetailsServiceAutoConfiguration`中实现的。`SecurityAutoConfiguration`导入了用于网络安全的`SpringBootWebSecurityConfiguration`，而`UserDetailsServiceAutoConfiguration`用于配置身份验证，对于非 Web 应用也同样适用。要完全关闭默认的 Web 应用程序的安全性配置或者要组合多个 Spring Security 组件（例如OAuth 2 Client 和 Resource Server），请添加类型为`WebSecurityConfigurerAdapter`的 bean（这样做不会禁用`UserDetailsService`配置或 Actuator 的安全性）。

要想把`UserDetailsService`配置也关闭掉，可以添加类型为`UserDetailsService`、`AuthenticationProvider`或`AuthenticationManager`的 bean。

通过添加自定义`WebSecurityConfigurerAdapter`可以覆盖访问规则。Spring Boot提供了方便的方法，可用于覆盖执行器端点和静态资源的访问规则。`EndpointRequest`可用于创建基于`management.endpoints.web.base-path`属性的`RequestMatcher`。可以使用`PathRequest`为常用路径的资源创建一个`RequestMatcher`。



## 9.2 WebFlux 安全

与 Spring MVC 应用程序类似，您可以通过添加`spring-boot-starter-security`依赖来保护 WebFlux 应用程序。默认的安全配置是在`ReactiveSecurityAutoConfiguration`和`UserDetailsServiceAutoConfiguration`中实现的。`ReactiveSecurityAutoConfiguration`导入了用于网络安全的`WebFluxSecurityConfiguration`，而`UserDetailsServiceAutoConfiguration`用于配置身份验证，对于非 Web 应用也同样适用。要完全关闭默认的 Web 应用程序的安全性配置，可以添加`WebFilterChainProxy`类型的 bean（这样做不会禁用`UserDetailsService`配置或 Actuator 的安全性）。

要想把`UserDetailsService`配置也关闭掉，可以添加`ReactiveUserDetailsService`或`ReactiveAuthenticationManager`类型的 bean。

通过添加自定义`SecurityWebFilterChain` bean，可以配置访问规则以及使用多个 Spring Security 组件（例如OAuth 2 Client 和 Resource Server）。Spring Boot 提供了方便的方法，可用于覆盖执行器端点和静态资源的访问规则。`EndpointRequest`可用于创建基于`management.endpoints.web.base-path`属性的`ServerWebExchangeMatcher`。

可以使用`PathRequest`为常用路径下的资源创建`ServerWebExchangeMatcher`。

例如，您可以通过添加以下内容来自定义安全配置：

```java
@Bean
public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
    return http
        .authorizeExchange()
            .matchers(PathRequest.toStaticResources().atCommonLocations()).permitAll()
            .pathMatchers("/foo", "/bar")
                .authenticated().and()
            .formLogin().and()
        .build();
}
```



## 9.3 OAuth2

[OAuth2](https://oauth.net/2/)是 Spring 支持的一种广泛使用的鉴权框架。



### 9.3.1 客户端

如果您在类路径中具有`spring-security-oauth2-client`，则可以利用一些自动配置功能来轻松设置OAuth2/Open ID Connect 客户端。此配置使用`OAuth2ClientProperties`下的属性。相同的属性适用于 servlet 和响应式应用程序。

您可以使用`spring.security.oauth2.client`前缀注册多个 OAuth2 客户端和提供者，如以下示例所示：

```properties
spring.security.oauth2.client.registration.my-client-1.client-id=abcd
spring.security.oauth2.client.registration.my-client-1.client-secret=password
spring.security.oauth2.client.registration.my-client-1.client-name=Client for user scope
spring.security.oauth2.client.registration.my-client-1.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-1.scope=user
spring.security.oauth2.client.registration.my-client-1.redirect-uri=https://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-1.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-1.authorization-grant-type=authorization_code

spring.security.oauth2.client.registration.my-client-2.client-id=abcd
spring.security.oauth2.client.registration.my-client-2.client-secret=password
spring.security.oauth2.client.registration.my-client-2.client-name=Client for email scope
spring.security.oauth2.client.registration.my-client-2.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-2.scope=email
spring.security.oauth2.client.registration.my-client-2.redirect-uri=https://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-2.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-2.authorization-grant-type=authorization_code

spring.security.oauth2.client.provider.my-oauth-provider.authorization-uri=https://my-auth-server/oauth/authorize
spring.security.oauth2.client.provider.my-oauth-provider.token-uri=https://my-auth-server/oauth/token
spring.security.oauth2.client.provider.my-oauth-provider.user-info-uri=https://my-auth-server/userinfo
spring.security.oauth2.client.provider.my-oauth-provider.user-info-authentication-method=header
spring.security.oauth2.client.provider.my-oauth-provider.jwk-set-uri=https://my-auth-server/token_keys
spring.security.oauth2.client.provider.my-oauth-provider.user-name-attribute=name
```

对于支持 [OpenID Connect 发现](https://openid.net/specs/openid-connect-discovery-1_0.html)的 OpenID Connect 提供者，可以进一步简化配置。提供者需要配置有一个`issuer-uri`，这是它声明为其发布者标识符的 URI。例如，如果提供的`issuer-uri`是"https://example.com "，则将向"https://example.com/.well-known/openid-configuration "发出`OpenID Provider Configuration Request`。结果应为`OpenID Provider Configuration Response`。以下示例展示了如何使用`issuer-uri`配置 OpenID Connect提供者：

```properties
spring.security.oauth2.client.provider.oidc-provider.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/
```

默认情况下，Spring Security 的`OAuth2LoginAuthenticationFilter`仅处理与`/login/oauth2/code/*`匹配的URL。如果要自定义`redirect-uri`以使用其他模式，则需要提供配置以处理该自定义模式。例如，对于 servlet 应用程序，您可以添加自己的类似于以下内容的`WebSecurityConfigurerAdapter`：

```java
public class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .anyRequest().authenticated()
                .and()
            .oauth2Login()
                .redirectionEndpoint()
                    .baseUri("/custom-callback");
    }
}
```



#### 公共提供者的OAuth2客户端注册

对于常见的 OAuth2 和 OpenID 提供者，包括Google、Github、Facebook 和 Okta，我们提供了一组提供者默认值（分别为Google、github、facebook 和 okta）。

如果不需要自定义这些提供者，则可以将`provider`属性设置为需要为其推断默认值的属性。另外，如果用于客户端注册的密钥与默认支持的提供者匹配，则 Spring Boot 也会进行推断。

换句话说，以下示例中的两个配置都使用 Google 提供者：

```properties
spring.security.oauth2.client.registration.my-client.client-id=abcd
spring.security.oauth2.client.registration.my-client.client-secret=password
spring.security.oauth2.client.registration.my-client.provider=google

spring.security.oauth2.client.registration.google.client-id=abcd
spring.security.oauth2.client.registration.google.client-secret=password
```



### 9.3.2 资源服务器

如果您的类路径上有`spring-security-oauth2-resource-server`，则 Spring Boot 可以设置 OAuth2 资源服务器。对于 JWT 配置，需要指定 JWK Set URI 或 OIDC 颁发者 URI，如以下示例所示：

```properties
spring.security.oauth2.resourceserver.jwt.jwk-set-uri=https://example.com/oauth2/default/v1/keys
```

```properties
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/
```

>[!note]
>
>如果授权服务器不支持 JWK Set URI，则可以使用用于验证 JWT 签名的公钥来配置资源服务器。可以使用`spring.security.oauth2.resourceserver.jwt.public-key-location`属性来完成此操作，该属性值需要指向包含 PEM 编码的 x509 格式的公钥文件。

相同的属性适用于 servlet 和响应式应用程序。

另外，您可以为 Servlet 应用程序定义自己的`JwtDecoder` bean，或者为响应式应用程序定义`ReactiveJwtDecoder`。

如果使用模糊的 token 而不是 JWT，则可以配置以下属性以通过内省来验证 token：

```properties
spring.security.oauth2.resourceserver.opaquetoken.introspection-uri=https://example.com/check-token
spring.security.oauth2.resourceserver.opaquetoken.client-id=my-client-id
spring.security.oauth2.resourceserver.opaquetoken.client-secret=my-client-secret
```

同样，相同的属性适用于 servlet 和响应式应用程序。

另外，您可以为 Servlet 应用程序定义自己的`OpaqueTokenIntrospector` bean，或者为响应式应用程序定义`ReactiveOpaqueTokenIntrospector`。



### 9.3.3 授权服务器

目前，Spring Security 不支持实现 OAuth 2.0 授权服务器。但是，这个功能可以从[Spring Security OAuth](https://spring.io/projects/spring-security-oauth)项目获得，Spring Security 最终将完全取代这个项目。在此之前，您可以使用`spring-security- oau2 -autoconfigure`模块轻松设置 OAuth 2.0 授权服务器。有关说明，请参阅其[文档](https://docs.spring.io/spring-security-oauth2-boot)。



## 9.4 SAML 2.0

### 9.4.1 依赖方

如果您的类路径上有`spring-security-saml2-service-provider`，那么您可以利用一些自动配置来简化 SAML 2.0 依赖方的设置。此配置使用`Saml2RelyingPartyProperties`下的属性。

依赖方注册代表身份提供者、IDP 和服务提供者、SP 之间的成对配置。您可以在`spring.security.saml2.relyingparty`前缀下注册多个依赖方，如下例所示：

```properties
spring.security.saml2.relyingparty.registration.my-relying-party1.signing.credentials[0].private-key-location=path-to-private-key
spring.security.saml2.relyingparty.registration.my-relying-party1.signing.credentials[0].certificate-location=path-to-certificate
spring.security.saml2.relyingparty.registration.my-relying-party1.identityprovider.verification.credentials[0].certificate-location=path-to-verification-cert
spring.security.saml2.relyingparty.registration.my-relying-party1.identityprovider.entity-id=remote-idp-entity-id1
spring.security.saml2.relyingparty.registration.my-relying-party1.identityprovider.sso-url=https://remoteidp1.sso.url

spring.security.saml2.relyingparty.registration.my-relying-party2.signing.credentials[0].private-key-location=path-to-private-key
spring.security.saml2.relyingparty.registration.my-relying-party2.signing.credentials[0].certificate-location=path-to-certificate
spring.security.saml2.relyingparty.registration.my-relying-party2.identityprovider.verification.credentials[0].certificate-location=path-to-other-verification-cert
spring.security.saml2.relyingparty.registration.my-relying-party2.identityprovider.entity-id=remote-idp-entity-id2
spring.security.saml2.relyingparty.registration.my-relying-party2.identityprovider.sso-url=https://remoteidp2.sso.url
```



## 9.5 执行器安全

为了安全起见，默认情况下禁用`/health`和`/info`以外的所有执行器。`management.endpoints.web.exposure.include`属性可用于启用执行器。

如果 Spring Security 位于类路径上，并且不存在其他`WebSecurityConfigurerAdapter`，则除`/health`和`/info`以外的所有执行器均由 Spring Boot 自动配置保护。如果定义自定义`WebSecurityConfigurerAdapter`，则 Spring Boot 自动配置将退出，您将完全控制执行器访问规则。

>[!note]
>
>在设置`management.endpoints.web.exposure.include`之前，请确保暴露的执行器不包含敏感信息和通过将它们放置在防火墙后面或通过诸如 Spring Security 之类的方法进行了保护。



### 9.5.1 跨站点请求伪造保护

由于 Spring Boot 依赖于 Spring Security 的默认设置，因此 CSRF 保护默认情况下处于启用状态。这意味着在使用默认安全配置时，需要`POST`（关闭和日志记录器端点）、`PUT`或`DELETE`的执行器端点将收到 403 的禁止错误。

>[!note]
>
>我们建议仅在创建非浏览器客户端使用的服务时才完全禁用 CSRF 保护。

关于 CSRF 保护的其他信息可以在[Spring Security 参考指南](https://docs.spring.io/spring-security/site/docs/5.2.1.RELEASE/reference/htmlsingle/#csrf)中找到。



# 10. 使用 SQL 数据库

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



# 11. 使用 NoSQL 技术

Spring Data提供了其他项目来帮助您访问各种NoSQL技术，包括：

- [MongoDB](https://spring.io/projects/spring-data-mongodb)
- [Neo4J](https://spring.io/projects/spring-data-neo4j)
- [Elasticsearch](https://spring.io/projects/spring-data-elasticsearch)
- [Solr](https://spring.io/projects/spring-data-solr)
- [Redis](https://spring.io/projects/spring-data-redis)
- [GemFire](https://spring.io/projects/spring-data-gemfire)或[Geode](https://spring.io/projects/spring-data-geode)
- [Cassandra](https://spring.io/projects/spring-data-cassandra)
- [Couchbase](https://spring.io/projects/spring-data-couchbase)
- [LDAP](https://spring.io/projects/spring-data-ldap)

Spring Boot为Redis，MongoDB，Neo4j，Elasticsearch，Solr Cassandra，Couchbase 和 LDAP 提供了自动配置。您也可以使用其他项目，但必须自己进行配置。请参阅[spring.io/projects/spring-data](https://spring.io/projects/spring-data)中的相应参考文档。



## 11.1 Redis

[Redis](https://redis.io/)是一种缓存、消息代理和多功能的键值存储。Spring Boot 为[Lettuce](https://github.com/lettuce-io/lettuce-core/)和[Jedis](https://github.com/xetorthio/jedis/)客户端库以及在此之上由[Spring Data Redis](https://github.com/spring-projects/spring-data-redis)提供的抽象提供了基本的自动配置。

`spring-boot-starter-data-redis`“启动器”可以以方便的方式收集依赖。默认情况下，使用[Lettuce](https://github.com/lettuce-io/lettuce-core/)。该启动器可以处理传统应用程序和响应式应用程序。

>[!tip]
>
>我们还提供了一个`spring-boot-starter-data-redis-active`“启动器”，以与其他具有响应支持的存储保持一致。



### 11.1.1 连接到Redis

您可以像注入其他任何 Spring Bean 一样注入自动配置的`RedisConnectionFactory`，`StringRedisTemplate`或普通的`RedisTemplate`实例。默认情况下，该实例尝试连接到位于`localhost:6379`的 Redis 服务器。下面的列表展示示了这种 bean 的示例：

```java
@Component
public class MyBean {

    private StringRedisTemplate template;

    @Autowired
    public MyBean(StringRedisTemplate template) {
        this.template = template;
    }

    // ...

}
```

>[!tip]
>
>您还可以注册任意数量的 Bean，这些 Bean 实现了`LettuceClientConfigurationBuilderCustomizer`以获得更高级的自定义特性。如果使用 Jedis，则还可以使用`JedisClientConfigurationBuilderCustomizer`。

如果添加自己的任何自动配置类型的`@Bean`，则它将替换默认值（`RedisTemplate`在以下场景下除外：当排除基于 bean 名称`redisTemplate`而不是其类型时）。默认情况下，如果`commons-pool2`在类路径上，则会得到一个池化连接工厂。



## 11.2 MongoDB

[MongoDB](https://www.mongodb.com/)是一个开源的 NoSQL 文档数据库，它使用类 Json 的形式，而不是传统的基于表的关系数据。Spring Boot 为使用 MongoDB 提供了一些便利，包括`spring-boot-starter-data-mongodb`和`spring-boot-starter-data-mongodb-reactive`“启动器”。



### 11.2.1 连接到 MongoDB 数据库

要访问 Mongo 数据库，可以注入自动配置的`org.springframework.data.mongodb.MongoDbFactory`。默认情况下，该实例尝试通过`mongodb://localhost/test`连接到 MongoDB 服务器。以下示例展示了如何连接到 MongoDB 数据库：

```java
import org.springframework.data.mongodb.MongoDbFactory;
import com.mongodb.DB;

@Component
public class MyBean {

    private final MongoDbFactory mongo;

    @Autowired
    public MyBean(MongoDbFactory mongo) {
        this.mongo = mongo;
    }

    // ...

    public void example() {
        DB db = mongo.getDb();
        // ...
    }

}
```

您可以设置`spring.data.mongodb.uri`属性来更改 URL 并配置其他设置，例如副本集，如以下示例所示：

```properties
spring.data.mongodb.uri=mongodb://user:secret@mongo1.example.com:12345,mongo2.example.com:23456/test
```

另外，只要您使用 Mongo 2.x，就可以指定`host`/`port`。例如，您可以在`application.properties`中声明以下设置：

```properties
spring.data.mongodb.host=mongoserver
spring.data.mongodb.port=27017
```

如果您定义了自己的`MongoClient`，它将用于自动配置合适的`MongoDbFactory`。`com.mongodb.MongoClient`和`com.mongodb.client.MongoClient`均支持。

>[!note]
>
>如果使用 Mongo 3.0 Java 驱动程序，则不支持`spring.data.mongodb.host`和`spring.data.mongodb.port`。在这种情况下，应使用`spring.data.mongodb.uri`提供所有配置。

<span></span>



>[!tip]
>
>如果未指定`spring.data.mongodb.port`，则使用默认值`27017`。您可以从前面展示的示例中删除此行。

<span></span>



>[!tip]
>
>如果不使用 Spring Data Mongo，则可以注入`com.mongodb.MongoClient` bean，而不是使用`MongoDbFactory`。如果您想完全控制建立 MongoDB 连接的方式，还可以声明自己的`MongoDbFactory`或`MongoClient` bean。

<span></span>



>[!note]
>
>如果您使用响应式驱动，则 SSL 需要 Netty。如果可以使用 Netty 并且尚未自定义要使用的工厂，则自动配置会自动配置该工厂。



### 11.2.2 MongoTemplate

[Spring Data MongoDB](https://spring.io/projects/spring-data-mongodb)提供了一个`MongoTemplate`类，该类的设计与 Spring 的`JdbcTemplate`非常相似。与`JdbcTemplate`一样，Spring Boot 为您自动配置一个 bean 来注入模板，如下所示：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.stereotype.Component;

@Component
public class MyBean {

    private final MongoTemplate mongoTemplate;

    @Autowired
    public MyBean(MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
    }

    // ...

}
```

有关完整的详细信息，请参见[`MongoOperations` Javadoc](https://docs.spring.io/spring-data/mongodb/docs/2.2.3.RELEASE/api/org/springframework/data/mongodb/core/MongoOperations.html)。



# 15. 使用`WebClient`调用 REST 服务



## 15.1 WebClient 运行时间









































































































