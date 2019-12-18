# 使用 Spring Boot

本节将详细介绍如何使用 Spring Boot，它涵盖了诸如构建系统、自动配置以及如何运行应用程序之类的主题，也包括了一些 Spring Boot 的最佳实践。尽管 Spring Boot 并没有什么特别的地方（它只是另一个可以使用的库），但是有一些建议可以使你的开发过程更轻松一些。

如果你是从 Spring Boot 开始的，那么在进入本节之前，您可能应该阅读一下*[入门指南](docs/2.2.2.RELEASE/getting-started.md)*。



# 1. 构建系统

强烈建议你选择一个支持*[依赖管理](https://docs.spring.io/spring-boot/docs/2.2.2.RELEASE/reference/html/using-spring-boot.html#using-boot-dependency-management)*并且可以使用 artifacts 发布到 Maven 中央仓库的构建系统，我们建议你选择 Maven 或 Gradle。也可以其他构建系统（例如 Ant），但是它们并没有得到很好的支持。



## 1.1 依赖管理

每个 Spring Boot 版本都提供了它所支持的依赖的精选清单。实际上，你不需要为构建配置中的所有这些依赖项提供版本，因为 Spring Boot 会为您管理该版本。当你升级 Spring Boot 本身时，这些依赖项也会以一致的方式升级。

>[!note]
>
>如果有需要，你也可以自己指定版本，覆盖 Spring Boot 建议的版本。

精选列表包含可与 Spring Boot 一起使用的所有 spring 模块以及完善的第三方库列表。