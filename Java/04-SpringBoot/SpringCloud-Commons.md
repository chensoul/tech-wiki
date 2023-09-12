# 云原生应用程序

[云原生](https://pivotal.io/platform-as-a-service/migrating-to-cloud-native-application-architectures-ebook)是一种应用程序开发风格，鼓励在持续交付和价值驱动开发领域轻松采用最佳实践。一个相关的学科是构建 [12 因素应用程序](https://12factor.net/)，其中开发实践与交付和运营目标保持一致 - 例如，通过使用声明性编程以及管理和监控。 Spring Cloud 通过多种特定方式促进这些开发风格。出发点是分布式系统中的所有组件都需要轻松访问的一组功能。

其中许多功能都由 Spring Cloud 构建的 Spring Boot 涵盖。 Spring Cloud 作为两个库提供了更多功能：Spring Cloud Context 和 Spring Cloud Commons。 Spring Cloud Context 为 Spring Cloud 应用程序的 `ApplicationContext` 提供实用程序和特殊服务（引导上下文、加密、刷新范围和环境端点）。 Spring Cloud Commons 是不同 Spring Cloud 实现（例如 Spring Cloud Netflix 和 Spring Cloud Consul）中使用的一组抽象和通用类。

如果您因“非法密钥大小”而出现异常，并且您使用 Sun 的 JDK，则需要安装 Java 加密扩展 (JCE) 无限强度管辖策略文件。请参阅以下链接了解更多信息：

- [Java 6 JCE Java 6 JCE](https://www.oracle.com/technetwork/java/javase/downloads/jce-6-download-429243.html)
- [Java 7 JCE Java 7 JCE](https://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html)
- [Java 8 JCE Java 8 JCE](https://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)

将文件提取到您使用的 JRE/JDK x64/x86 版本的 JDK/jre/lib/security 文件夹中。

## 1. Spring Cloud Context：应用程序上下文服务


Spring Boot 对于如何使用 Spring 构建应用程序有自己的看法。例如，它具有用于公共配置文件的常规位置，并具有用于公共管理和监视任务的端点。 Spring Cloud 在此基础上构建，并添加了系统中许多组件会使用或偶尔需要的一些功能。

### 1.1.引导程序应用程序上下文


Spring Cloud 应用程序通过创建“引导”上下文来运行，该上下文是主应用程序的父上下文。此上下文负责从外部源加载配置属性并解密本地外部配置文件中的属性。这两个上下文共享一个 `Environment` ，它是任何 Spring 应用程序的外部属性的来源。默认情况下，引导属性（不是 `bootstrap.properties` 而是在引导阶段加载的属性）以高优先级添加，因此它们不能被本地配置覆盖。

引导上下文使用与主应用程序上下文不同的约定来定位外部配置。您可以使用 `bootstrap.yml` 来代替 `application.yml` （或 `.properties` ），从而使引导程序和主上下文的外部配置很好地分开。以下清单显示了一个示例：
示例 1.bootstrap.yml

```
spring:
  application:
    name: foo
  cloud:
    config:
      uri: ${SPRING_CONFIG_URI:http://localhost:8888}
```


如果您的应用程序需要来自服务器的任何特定于应用程序的配置，最好设置 `spring.application.name` （在 `bootstrap.yml` 或 `application.yml` 中）。为了将属性 `spring.application.name` 用作应用程序的上下文 ID，您必须在 `bootstrap.[properties | yml]` 中设置它。


如果您想检索特定的配置文件配置，还应该在 `bootstrap.[properties | yml]` 中设置 `spring.profiles.active` 。


您可以通过设置 `spring.cloud.bootstrap.enabled=false` （例如，在系统属性中）来完全禁用引导进程。

### 1.2.应用程序上下文层次结构


如果您从 `SpringApplication` 或 `SpringApplicationBuilder` 构建应用程序上下文，则 Bootstrap 上下文将作为父上下文添加到该上下文。 Spring 的一个功能是子上下文从其父上下文继承属性源和配置文件，因此与不使用 Spring Cloud Config 构建相同的上下文相比，“主”应用程序上下文包含额外的属性源。额外的财产来源是：

- “bootstrap”：如果在引导上下文中找到任何 `PropertySourceLocators` 并且它们具有非空属性，则可选的 `CompositePropertySource` 将以高优先级出现。一个例子是来自 Spring Cloud Config Server 的属性。有关如何自定义此属性源的内容，请参阅“自定义 Bootstrap 属性源”。

  > 在 Spring Cloud 2022.0.3 之前 `PropertySourceLocators` （包括 Spring Cloud Config 的）在主应用程序上下文中运行，而不是在 Bootstrap 上下文中运行。您可以通过在 `bootstrap.[properties | yaml]` 中设置 `spring.cloud.config.initialize-on-context-refresh=true` 来强制 `PropertySourceLocators` 在 Bootstrap 上下文期间运行。

- “applicationConfig: [classpath:bootstrap.yml]”（以及相关文件，如果 Spring 配置文件处于活动状态）：如果您有 `bootstrap.yml` （或 `.properties` ），这些属性用于配置引导上下文。然后，当设置其父上下文时，它们将被添加到子上下文中。它们的优先级低于 `application.yml` （或 `.properties` ）以及作为创建 Spring Boot 应用程序过程的正常部分添加到子级的任何其他属性源。有关如何自定义这些属性源的内容，请参阅“更改引导属性的位置”。


由于属性源的排序规则，“bootstrap”条目优先。但是，请注意，它们不包含来自 `bootstrap.yml` 的任何数据，该数据的优先级非常低，但可用于设置默认值。



您可以通过设置您创建的任何 `ApplicationContext` 的父上下文来扩展上下文层次结构 - 例如，通过使用其自己的接口或使用 `SpringApplicationBuilder` 便捷方法 ( `parent()` 、 `child()` 和 `sibling()` ）。引导上下文是您自己创建的最高级祖先的父级。层次结构中的每个上下文都有自己的“引导程序”（可能为空）属性源，以避免无意中将值从父母传递到其后代。如果存在配置服务器，则层次结构中的每个上下文也可以（原则上）具有不同的 `spring.application.name` ，因此也可以具有不同的远程属性源。正常的 Spring 应用程序上下文行为规则适用于属性解析：子上下文中的属性按名称以及属性源名称覆盖父上下文中的属性。 （如果子级具有与父级同名的属性源，则来自父级的值不会包含在子级中）。


请注意， `SpringApplicationBuilder` 允许您在整个层次结构中共享 `Environment` ，但这不是默认设置。因此，同级上下文（特别是）不需要具有相同的配置文件或属性源，即使它们可能与其父级共享共同的值。

### 1.3.更改引导程序属性的位置


`bootstrap.yml` （或 `.properties` ）位置可以通过设置 `spring.cloud.bootstrap.name` （默认： `bootstrap` ）、 `spring.cloud.bootstrap.location` 来指定（默认：空）或 `spring.cloud.bootstrap.additional-location` （默认：空）- 例如，在系统属性中。


这些属性的行为类似于具有相同名称的 `spring.config.*` 变体。使用 `spring.cloud.bootstrap.location` 时，默认位置将被替换，并且仅使用指定的位置。要将位置添加到默认位置列表中，可以使用 `spring.cloud.bootstrap.additional-location` 。事实上，它们用于通过在 `Environment` 中设置这些属性来设置引导 `ApplicationContext` 。如果存在活动配置文件（来自 `spring.profiles.active` 或通过您正在构建的上下文中的 `Environment` API），该配置文件中的属性也会被加载，与常规 Spring 中的属性相同启动应用程序 - 例如，从 `bootstrap-development.properties` 启动 `development` 配置文件。

### 1.4.覆盖远程属性的值


通过引导上下文添加到应用程序的属性源通常是“远程”的（例如，来自 Spring Cloud Config Server）。默认情况下，它们不能在本地被覆盖。如果您想让应用程序使用自己的系统属性或配置文件覆盖远程属性，则远程属性源必须通过设置 `spring.cloud.config.allowOverride=true` 授予其权限（在本地设置此设置不起作用）。设置该标志后，两个更细粒度的设置将控制远程属性相对于系统属性和应用程序本地配置的位置：

- `spring.cloud.config.overrideNone=true` ：从任何本地属性源覆盖。
- `spring.cloud.config.overrideSystemProperties=false`: 只有系统属性、命令行参数和环境变量（而不是本地配置文件）应该覆盖远程设置。

### 1.5.自定义引导程序配置


通过将条目添加到名为 `org.springframework.cloud.bootstrap.BootstrapConfiguration` 的键下的 `/META-INF/spring.factories` ，可以将引导上下文设置为执行任何您喜欢的操作。它包含用于创建上下文的 Spring `@Configuration` 类的逗号分隔列表。您希望主应用程序上下文可用于自动装配的任何 bean 都可以在此处创建。 `ApplicationContextInitializer` 类型的 `@Beans` 有一个特殊的约定。如果要控制启动顺序，可以使用 `@Order` 注解标记类（默认顺序为 `last` ）。

>添加自定义 `BootstrapConfiguration` 时，请注意您添加的类不会被 `@ComponentScanned` 错误地添加到您的“主”应用程序上下文中，因为它们可能不需要。对启动配置类使用单独的包名称，并确保该名称尚未被 `@ComponentScan` 或 `@SpringBootApplication` 带注释的配置类覆盖。


引导过程通过将初始化程序注入到主 `SpringApplication` 实例中结束（这是正常的 Spring Boot 启动序列，无论它作为独立应用程序运行还是部署在应用程序服务器中）。首先，从 `spring.factories` 中找到的类创建引导上下文。然后，在启动之前，将 `ApplicationContextInitializer` 类型的所有 `@Beans` 添加到主 `SpringApplication` 中。

### 1.6.自定义 Bootstrap 属性源


引导程序过程添加的外部配置的默认属性源是 Spring Cloud Config Server，但是您可以通过将 `PropertySourceLocator` 类型的 bean 添加到引导程序上下文（通过 `spring.factories` ）。例如，您可以从不同的服务器或数据库插入其他属性。


作为示例，请考虑以下自定义定位器：

```java
@Configuration
public class CustomPropertySourceLocator implements PropertySourceLocator {

    @Override
    public PropertySource<?> locate(Environment environment) {
        return new MapPropertySource("customProperty",
                Collections.<String, Object>singletonMap("property.from.sample.custom.source", "worked as intended"));
    }

}
```


传入的 `Environment` 是即将创建的 `ApplicationContext` 的那个 - 换句话说，我们为其提供额外的属性源。它已经具有正常的 Spring Boot 提供的属性源，因此您可以使用它们来定位特定于此 `Environment` 的属性源（例如，通过将其键入 `spring.application.name` ，如下所示在默认的 Spring Cloud Config Server 属性源定位器中完成）。


如果您创建一个包含此类的 jar，然后添加包含以下设置的 `META-INF/spring.factories` ，则 `customProperty` `PropertySource` 会出现在包含该 jar 的任何应用程序中它的类路径：

```
org.springframework.cloud.bootstrap.BootstrapConfiguration=sample.custom.CustomPropertySourceLocator
```


从 Spring Cloud 2022.0.3 开始，Spring Cloud 现在将调用 `PropertySourceLocators` 两次。第一次获取将检索没有任何配置文件的任何属性源。这些属性源将有机会使用 `spring.profiles.active` 激活配置文件。主应用程序上下文启动后， `PropertySourceLocators` 将被第二次调用，这次任何活动配置文件都允许 `PropertySourceLocators` 找到任何带有配置文件的附加 `PropertySources` 。

### 1.7.日志记录配置

If you use Spring Boot to configure log settings, you should place this configuration in `bootstrap.[yml | properties]` if you would like it to apply to all events.
如果您使用 Spring Boot 配置日志设置，并且希望将此配置应用于所有事件，则应将此配置放在 `bootstrap.[yml | properties]` 中。

> 为了让 Spring Cloud 正确初始化日志配置，您不能使用自定义前缀。例如，在初始化日志系统时，Spring Cloud 无法识别使用 `custom.loggin.logpath` 。

### 8.环境变化


应用程序侦听 `EnvironmentChangeEvent` 并以几种标准方式对更改做出反应（可以以正常方式将附加 `ApplicationListeners` 添加为 `@Beans` ）。当观察到 `EnvironmentChangeEvent` 时，它具有已更改的键值列表，应用程序使用这些值来：

- 重新绑定上下文中的任何 `@ConfigurationProperties` bean。
- 设置 `logging.level.*` 中任何属性的记录器级别。


请注意，默认情况下，Spring Cloud Config Client 不会轮询 `Environment` 中的更改。一般来说，我们不建议使用这种方法来检测更改（尽管您可以使用 `@Scheduled` 注释进行设置）。如果您有横向扩展的客户端应用程序，最好将 `EnvironmentChangeEvent` 广播到所有实例，而不是让它们轮询更改（例如，通过使用 Spring Cloud Bus）。


`EnvironmentChangeEvent` 涵盖了一大类刷新用例，只要您可以实际对 `Environment` 进行更改并发布事件即可。请注意，这些 API 是公共的，并且是 Spring 核心的一部分）。您可以通过访问 `/configprops` 端点（标准 Spring Boot Actuator 功能）来验证更改是否已绑定到 `@ConfigurationProperties` beans。例如， `DataSource` 可以在运行时更改其 `maxPoolSize` （Spring Boot创建的默认 `DataSource` 是 `@ConfigurationProperties` bean）并且动态增长容量。重新绑定 `@ConfigurationProperties` 不涵盖另一大类用例，在这些用例中，您需要对刷新进行更多控制，并且需要对整个 `ApplicationContext` 进行原子更改。为了解决这些问题，我们有 `@RefreshScope` 。

### 1.9.刷新范围


当配置发生更改时，标记为 `@RefreshScope` 的 Spring `@Bean` 会得到特殊处理。此功能解决了有状态 Bean 仅在初始化时才注入配置的问题。例如，如果在通过 `Environment` 更改数据库 URL 时 `DataSource` 具有打开的连接，您可能希望这些连接的持有者能够完成他们正在做的事情。然后，下次从池中借用连接时，它会获取带有新 URL 的连接。


有时，甚至可能强制在某些只能初始化一次的 bean 上应用 `@RefreshScope` 注释。如果 bean 是“不可变的”，则必须使用 `@RefreshScope` 注释该 bean，或者在属性键下指定类名： `spring.cloud.refresh.extra-refreshable` 。

> 如果您的 `DataSource` bean 是 `HikariDataSource` ，则无法刷新它。它是 `spring.cloud.refresh.never-refreshable` 的默认值。如果您需要刷新，请选择不同的 `DataSource` 实现。


刷新范围 bean 是惰性代理，在使用它们时（即调用方法时）进行初始化，并且范围充当初始化值的缓存。要强制 Bean 在下一个方法调用时重新初始化，您必须使其缓存条目无效。


`RefreshScope` 是上下文中的 bean，并具有公共 `refreshAll()` 方法，可通过清除目标缓存来刷新范围内的所有 bean。 `/refresh` 端点公开此功能（通过 HTTP 或 JMX）。要按名称刷新单个 bean，还有一个 `refresh(String)` 方法。


要公开 `/refresh` 端点，您需要向应用程序添加以下配置：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: refresh
        
```

>  `@RefreshScope` （技术上）适用于 `@Configuration` 类，但它可能会导致令人惊讶的行为。例如，这并不意味着该类中定义的所有 `@Beans` 本身都在 `@RefreshScope` 中。具体来说，任何依赖于这些 bean 的东西都不能依赖它们在刷新启动时更新，除非它本身位于 `@RefreshScope` 中。在这种情况下，它会在刷新时重建，并重新注入其依赖项。此时，它们将从刷新的 `@Configuration` 中重新初始化）。
>
>  删除配置值然后执行刷新不会更新配置值的存在。配置属性必须存在才能在刷新后更新值。如果您依赖应用程序中某个值的存在，您可能需要将逻辑切换为依赖该值的不存在。另一种选择是依赖值的更改，而不是不存在于应用程序的配置中。

### 1.10.加密与解密


Spring Cloud 有一个 `Environment` 预处理器，用于在本地解密属性值。它遵循与 Spring Cloud Config Server 相同的规则，并通过 `encrypt.*` 具有相同的外部配置。因此，您可以使用 `{cipher}*` 形式的加密值，并且只要存在有效密钥，它们就会在主应用程序上下文获取 `Environment` 设置之前解密。要在应用程序中使用加密功能，您需要在类路径中包含 Spring Security RSA（Maven 坐标： `org.springframework.security:spring-security-rsa` ），并且还需要 JVM 中的完整强度 JCE 扩展。


如果您因“非法密钥大小”而出现异常，并且您使用 Sun 的 JDK，则需要安装 Java 加密扩展 (JCE) 无限强度管辖策略文件。请参阅以下链接了解更多信息：

- [Java 6 JCE Java 6 JCE](https://www.oracle.com/technetwork/java/javase/downloads/jce-6-download-429243.html)
- [Java 7 JCE Java 7 JCE](https://www.oracle.com/technetwork/java/javase/downloads/jce-7-download-432124.html)
- [Java 8 JCE Java 8 JCE](https://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)


将文件提取到您使用的 JRE/JDK x64/x86 版本的 JDK/jre/lib/security 文件夹中。

### 1.11.端点


对于 Spring Boot Actuator 应用程序，可以使用一些额外的管理端点。您可以使用：

- `POST` 到 `/actuator/env` 以更新 `Environment` 并重新绑定 `@ConfigurationProperties` 和日志级别。要启用此端点，您必须设置 `management.endpoint.env.post.enabled=true` 。
- `/actuator/refresh` 重新加载引导上下文并刷新 `@RefreshScope` beans。
- `/actuator/restart` 关闭 `ApplicationContext` 并重新启动它（默认情况下禁用）。
- `/actuator/pause` 和 `/actuator/resume` 用于调用 `Lifecycle` 方法（ `ApplicationContext` 和 `start()` > ).

> 虽然为 `/actuator/env` 端点启用 `POST` 方法可以为管理应用程序环境变量提供灵活性和便利性，但确保端点受到保护和监控以防止潜在的安全风险至关重要。添加 `spring-boot-starter-security` 依赖项来配置执行器端点的访问控制。
>
>  如果禁用 `/actuator/restart` 端点，则 `/actuator/pause` 和 `/actuator/resume` 端点也将被禁用，因为它们只是 `/actuator/restart` 的特殊情况。

## 2.Spring Cloud Commons：公共抽象


服务发现、负载均衡和断路器等模式提供了一个公共抽象层，所有 Spring Cloud 客户端都可以使用该抽象层，而与实现无关（例如，使用 Eureka 或 Consul 进行发现）。

### 2.1. `@EnableDiscoveryClient` 注释


Spring Cloud Commons 提供了 `@EnableDiscoveryClient` 注解。这将查找 `DiscoveryClient` 和 `ReactiveDiscoveryClient` 接口与 `META-INF/spring.factories` 的实现。发现客户端的实现将配置类添加到 `org.springframework.cloud.client.discovery.EnableDiscoveryClient` 键下的 `spring.factories` 中。 `DiscoveryClient` 实现的示例包括 Spring Cloud Netflix Eureka、Spring Cloud Consul Discovery 和 Spring Cloud Zookeeper Discovery。


Spring Cloud 默认提供阻塞式和反应式服务发现客户端。您可以通过设置 `spring.cloud.discovery.blocking.enabled=false` 或 `spring.cloud.discovery.reactive.enabled=false` 轻松禁用阻塞和/或反应式客户端。要完全禁用服务发现，您只需设置 `spring.cloud.discovery.enabled=false` 。


默认情况下， `DiscoveryClient` 的实现会自动将本地 Spring Boot 服务器注册到远程发现服务器。可以通过在 `@EnableDiscoveryClient` 中设置 `autoRegister=false` 来禁用此行为。

> 不再需要 `@EnableDiscoveryClient` 。您可以将 `DiscoveryClient` 实现放在类路径上，以使 Spring Boot 应用程序向服务发现服务器注册。

#### 2.1.1.健康指标


Commons 自动配置以下 Spring Boot 健康指标。

##### DiscoveryClientHealthIndicator


该运行状况指示器基于当前注册的 `DiscoveryClient` 实现。

- 要完全禁用，请设置 `spring.cloud.discovery.client.health-indicator.enabled=false` 。
- 要禁用描述字段，请设置 `spring.cloud.discovery.client.health-indicator.include-description=false` 。否则，它可能会像卷起的 `HealthIndicator` 的 `description` 一样冒泡。
- 要禁用服务检索，请设置 `spring.cloud.discovery.client.health-indicator.use-services-query=false` 。默认情况下，指标调用客户端的 `getServices` 方法。在具有许多注册服务的部署中，在每次检查期间检索所有服务的成本可能太高。这将跳过服务检索，而是使用客户端的 `probe` 方法。

##### DiscoveryCompositeHealthContributor


该综合运行状况指示器基于所有已注册的 `DiscoveryHealthIndicator` beans。要禁用，请设置 `spring.cloud.discovery.client.composite-indicator.enabled=false` 。

#### 2.1.2.排序 `DiscoveryClient` 实例


`DiscoveryClient` 接口扩展 `Ordered` 。这在使用多个发现客户端时非常有用，因为它允许您定义返回的发现客户端的顺序，类似于对 Spring 应用程序加载的 bean 进行排序的方式。默认情况下，任何 `DiscoveryClient` 的顺序设置为 `0` 。如果您想为自定义 `DiscoveryClient` 实现设置不同的顺序，则只需重写 `getOrder()` 方法，以便它返回适合您的设置的值。除此之外，您还可以使用属性来设置 Spring Cloud 提供的 `DiscoveryClient` 实现的顺序，其中包括 `ConsulDiscoveryClient` 、 `EurekaDiscoveryClient` 和 `ZookeeperDiscoveryClient` （或 Eureka 的 `eureka.client.order` ）属性设置为所需的值。

#### 2.1.3.简单发现客户端


如果类路径中没有 Service-Registry 支持的 `DiscoveryClient` ，则将使用使用属性获取服务和实例信息的 `SimpleDiscoveryClient` 实例。


有关可用实例的信息应按以下格式通过属性传递： `spring.cloud.discovery.client.simple.instances.service1[0].uri=http://s11:8080` ，其中 `spring.cloud.discovery.client.simple.instances` 是公共前缀，然后 `service1` 代表 ID相关服务的编号，而 `[0]` 表示实例的索引号（如示例中所示，索引以 `0` 开头），然后是 `uri`

### 2.2.服务注册中心


Commons 现在提供 `ServiceRegistry` 接口，该接口提供 `register(Registration)` 和 `deregister(Registration)` 等方法，使您可以提供自定义注册服务。 `Registration` 是一个标记接口。


以下示例显示了正在使用的 `ServiceRegistry` ：

```java
@Configuration
@EnableDiscoveryClient(autoRegister=false)
public class MyConfiguration {
    private ServiceRegistry registry;

    public MyConfiguration(ServiceRegistry registry) {
        this.registry = registry;
    }

    // called through some external process, such as an event or a custom actuator endpoint
    public void register() {
        Registration registration = constructRegistration();
        this.registry.register(registration);
    }
}
```


每个 `ServiceRegistry` 实现都有自己的 `Registry` 实现。

- `ZookeeperRegistration` 与 `ZookeeperServiceRegistry` 一起使用
- `EurekaRegistration` 与 `EurekaServiceRegistry` 一起使用
- `ConsulRegistration` 与 `ConsulServiceRegistry` 一起使用

如果您使用 `ServiceRegistry` 接口，则需要为您正在使用的 `ServiceRegistry` 实现传递正确的 `Registry` 实现。

#### 2.2.1. ServiceRegistry自动注册


默认情况下， `ServiceRegistry` 实现会自动注册正在运行的服务。要禁用该行为，您可以设置： * `@EnableDiscoveryClient(autoRegister=false)` 以永久禁用自动注册。 * `spring.cloud.service-registry.auto-registration.enabled=false` 通过配置禁用该行为。

##### 自动注册事件


服务自动注册时将触发两个事件。第一个事件称为 `InstancePreRegisteredEvent` ，在注册服务之前触发。第二个事件称为 `InstanceRegisteredEvent` ，在服务注册后触发。您可以注册一个 `ApplicationListener` 来监听这些事件并做出反应。

> 如果 `spring.cloud.service-registry.auto-registration.enabled` 属性设置为 `false` ，这些事件将不会被触发。

#### 2.2.2.服务注册表执行器端点


Spring Cloud Commons 提供了一个 `/service-registry` 执行器端点。此端点依赖于 Spring 应用程序上下文中的 `Registration` bean。使用 GET 调用 `/service-registry` 会返回 `Registration` 的状态。使用 POST 到同一端点并使用 JSON 正文会将当前 `Registration` 的状态更改为新值。 JSON 正文必须包含具有首选值的 `status` 字段。请参阅您在更新状态时使用的允许值以及为状态返回的值的 `ServiceRegistry` 实现的文档。例如，Eureka 支持的状态是 `UP` 、 `DOWN` 、 `OUT_OF_SERVICE` 和 `UNKNOWN` 。

### 2.3. Spring RestTemplate 作为负载均衡器客户端


您可以配置 `RestTemplate` 以使用负载平衡器客户端。要创建负载平衡的 `RestTemplate` ，请创建 `RestTemplate` `@Bean` 并使用 `@LoadBalanced` 限定符，如以下示例所示：

```java
@Configuration
public class MyConfiguration {

    @LoadBalanced
    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

public class MyClass {
    @Autowired
    private RestTemplate restTemplate;

    public String doOtherStuff() {
        String results = restTemplate.getForObject("http://stores/stores", String.class);
        return results;
    }
}
```

>  `RestTemplate` bean 不再通过自动配置创建。单独的应用程序必须创建它。


URI需要使用虚拟主机名（即服务名，而不是主机名）。 BlockingLoadBalancerClient 用于创建完整的物理地址。

> 要使用负载平衡 `RestTemplate` ，您需要在类路径中实现负载平衡器。将 Spring Cloud LoadBalancer starter 添加到您的项目中以便使用它。

### 2.4. Spring WebClient 作为负载均衡器客户端


您可以将 `WebClient` 配置为自动使用负载平衡器客户端。要创建负载平衡的 `WebClient` ，请创建 `WebClient.Builder` `@Bean` 并使用 `@LoadBalanced` 限定符，如下所示：

```java
@Configuration
public class MyConfiguration {

    @Bean
    @LoadBalanced
    public WebClient.Builder loadBalancedWebClientBuilder() {
        return WebClient.builder();
    }
}

public class MyClass {
    @Autowired
    private WebClient.Builder webClientBuilder;

    public Mono<String> doOtherStuff() {
        return webClientBuilder.build().get().uri("http://stores/stores")
                        .retrieve().bodyToMono(String.class);
    }
}
```


URI需要使用虚拟主机名（即服务名，而不是主机名）。 Spring Cloud LoadBalancer 用于创建完整的物理地址。

> 如果您想使用 `@LoadBalanced WebClient.Builder` ，则需要在类路径中实现负载均衡器。我们建议您将 Spring Cloud LoadBalancer 启动器添加到您的项目中。然后，在下面使用 `ReactiveLoadBalancer` 。

#### 2.4.1.重试失败的请求


负载平衡的 `RestTemplate` 可以配置为重试失败的请求。默认情况下，此逻辑被禁用。对于非响应式版本（使用 `RestTemplate` ），您可以通过将 Spring Retry 添加到应用程序的类路径来启用它。对于反应式版本（使用 `WebTestClient` ），您需要设置 `spring.cloud.loadbalancer.retry.enabled=true` 。


如果您想在类路径上使用 Spring Retry 或 Reactive Retry 禁用重试逻辑，您可以设置 `spring.cloud.loadbalancer.retry.enabled=false` 。


对于非响应式实现，如果您想在重试中实现 `BackOffPolicy` ，则需要创建一个 `LoadBalancedRetryFactory` 类型的 bean 并覆盖 `createBackOffPolicy()` 方法。


对于反应式实现，您只需将 `spring.cloud.loadbalancer.retry.backoff.enabled` 设置为 `false` 即可启用它。

您可以设置：

- `spring.cloud.loadbalancer.retry.maxRetriesOnSameServiceInstance` - 指示应在同一 `ServiceInstance` 上重试请求的次数（对每个选定实例单独计数）
- `spring.cloud.loadbalancer.retry.maxRetriesOnNextServiceInstance` - 指示新选择的 `ServiceInstance` 请求应重试多少次
- `spring.cloud.loadbalancer.retry.retryableStatusCodes` - - 始终重试失败请求的状态代码。


对于反应式实现，您还可以设置： - `spring.cloud.loadbalancer.retry.backoff.minBackoff` - 设置最小退避持续时间（默认情况下，5 毫秒） - `spring.cloud.loadbalancer.retry.backoff.maxBackoff` - 设置最大退避持续时间（默认情况下，最大long 毫秒值） - `spring.cloud.loadbalancer.retry.backoff.jitter` - 设置用于计算每次调用的实际退避持续时间的抖动（默认为 0.5）。


对于反应式实现，您还可以实现自己的 `LoadBalancerRetryPolicy` 以对负载平衡调用重试进行更详细的控制。


对于这两种实现，您还可以通过在 `spring.cloud.loadbalancer.[serviceId].retry.retryable-exceptions` 属性下添加值列表来设置触发回复的异常。如果您这样做，我们请确保将 `RetryableStatusCodeExceptions` 添加到您提供的异常列表中，以便我们也重试可重试的状态代码。如果您没有通过属性指定任何异常，则我们默认使用的异常是 `IOException` 、 `TimeoutException` 和 `RetryableStatusCodeException` 。您还可以通过将 `spring.cloud.loadbalancer.[serviceId].retry.retry-on-all-exceptions` 设置为 `true` 来启用对所有异常的重试。

> 如果您将阻塞实现与 Spring Retries 结合使用，并且想要保留以前版本的行为，请将 `spring.cloud.loadbalancer.[serviceId].retry.retry-on-all-exceptions` 设置为 `true` ，因为它曾经是阻塞实现的默认模式。
>
> 各个负载均衡器客户端可以单独配置为具有与上述相同的属性，但前缀为 `spring.cloud.loadbalancer.clients.<clientId>.*` ，其中 `clientId` 是负载均衡器的名称。
>
> 对于负载平衡重试，默认情况下，我们用 `RetryAwareServiceInstanceListSupplier` 包装 `ServiceInstanceListSupplier` bean，以选择与先前选择的实例不同的实例（如果可用）。您可以通过将 `spring.cloud.loadbalancer.retry.avoidPreviousInstance` 的值设置为 `false` 来禁用此行为。

```java
@Configuration
public class MyConfiguration {
    @Bean
    LoadBalancedRetryFactory retryFactory() {
        return new LoadBalancedRetryFactory() {
            @Override
            public BackOffPolicy createBackOffPolicy(String service) {
                return new ExponentialBackOffPolicy();
            }
        };
    }
}
```


如果您想向重试功能添加一个或多个 `RetryListener` 实现，则需要创建一个 `LoadBalancedRetryListenerFactory` 类型的 bean 并返回您想要的 `RetryListener` 数组用于给定服务，如以下示例所示：

```java
@Configuration
public class MyConfiguration {
    @Bean
    LoadBalancedRetryListenerFactory retryListenerFactory() {
        return new LoadBalancedRetryListenerFactory() {
            @Override
            public RetryListener[] createRetryListeners(String service) {
                return new RetryListener[]{new RetryListener() {
                    @Override
                    public <T, E extends Throwable> boolean open(RetryContext context, RetryCallback<T, E> callback) {
                        //TODO Do you business...
                        return true;
                    }

                    @Override
                     public <T, E extends Throwable> void close(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
                        //TODO Do you business...
                    }

                    @Override
                    public <T, E extends Throwable> void onError(RetryContext context, RetryCallback<T, E> callback, Throwable throwable) {
                        //TODO Do you business...
                    }
                }};
            }
        };
    }
}
```

### 2.5.多个 `RestTemplate` 对象


如果您想要一个非负载平衡的 `RestTemplate` ，请创建一个 `RestTemplate` bean 并注入它。要访问负载平衡的 `RestTemplate` ，请在创建 `@Bean` 时使用 `@LoadBalanced` 限定符，如以下示例所示：

```java
@Configuration
public class MyConfiguration {

    @LoadBalanced
    @Bean
    RestTemplate loadBalanced() {
        return new RestTemplate();
    }

    @Primary
    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

public class MyClass {
    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    @LoadBalanced
    private RestTemplate loadBalanced;

    public String doOtherStuff() {
        return loadBalanced.getForObject("http://stores/stores", String.class);
    }

    public String doStuff() {
        return restTemplate.getForObject("http://example.com", String.class);
    }
}
```

> 请注意，在前面的示例中，在纯 `RestTemplate` 声明上使用 `@Primary` 注释来消除不合格的 `@Autowired` 注入的歧义。
>
>  如果您看到诸如 `java.lang.IllegalArgumentException: Can not set org.springframework.web.client.RestTemplate field com.my.app.Foo.restTemplate to com.sun.proxy.$Proxy89` 之类的错误，请尝试注入 `RestOperations` 或设置 `spring.aop.proxyTargetClass=true` 。

### 2.6。多个 WebClient 对象


如果您想要一个非负载平衡的 `WebClient` ，请创建一个 `WebClient` bean 并注入它。要访问负载平衡的 `WebClient` ，请在创建 `@Bean` 时使用 `@LoadBalanced` 限定符，如以下示例所示：

```java
@Configuration
public class MyConfiguration {

    @LoadBalanced
    @Bean
    WebClient.Builder loadBalanced() {
        return WebClient.builder();
    }

    @Primary
    @Bean
    WebClient.Builder webClient() {
        return WebClient.builder();
    }
}

public class MyClass {
    @Autowired
    private WebClient.Builder webClientBuilder;

    @Autowired
    @LoadBalanced
    private WebClient.Builder loadBalanced;

    public Mono<String> doOtherStuff() {
        return loadBalanced.build().get().uri("http://stores/stores")
                        .retrieve().bodyToMono(String.class);
    }

    public Mono<String> doStuff() {
        return webClientBuilder.build().get().uri("http://example.com")
                        .retrieve().bodyToMono(String.class);
    }
}
```

### 2.7. Spring WebFlux `WebClient` 作为负载均衡器客户端


Spring WebFlux 可以使用反应式和非反应式 `WebClient` 配置，如主题所述：

- [Spring WebFlux `WebClient` with `ReactorLoadBalancerExchangeFilterFunction`](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#webflux-with-reactive-loadbalancer)
- [Spring WebFlux `WebClient` with a Non-reactive Load Balancer Client](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#load-balancer-exchange-filter-function)

#### 2.7.1. Spring WebFlux `WebClient` 与 `ReactorLoadBalancerExchangeFilterFunction`


您可以配置 `WebClient` 以使用 `ReactiveLoadBalancer` 。如果您将 Spring Cloud LoadBalancer 启动器添加到项目中，并且 `spring-webflux` 位于类路径上，则 `ReactorLoadBalancerExchangeFilterFunction` 会自动配置。以下示例显示如何配置 `WebClient` 以使用反应式负载均衡器：

```java
public class MyClass {
    @Autowired
    private ReactorLoadBalancerExchangeFilterFunction lbFunction;

    public Mono<String> doOtherStuff() {
        return WebClient.builder().baseUrl("http://stores")
            .filter(lbFunction)
            .build()
            .get()
            .uri("/stores")
            .retrieve()
            .bodyToMono(String.class);
    }
}
```


URI需要使用虚拟主机名（即服务名，而不是主机名）。 `ReactorLoadBalancer` 用于创建完整的物理地址。

#### 2.7.2. Spring WebFlux `WebClient` 与非反应式负载均衡器客户端


如果 `spring-webflux` 位于类路径上，则自动配置 `LoadBalancerExchangeFilterFunction` 。但请注意，这在幕后使用了非响应式客户端。以下示例显示如何配置 `WebClient` 以使用负载均衡器：

```java
public class MyClass {
    @Autowired
    private LoadBalancerExchangeFilterFunction lbFunction;

    public Mono<String> doOtherStuff() {
        return WebClient.builder().baseUrl("http://stores")
            .filter(lbFunction)
            .build()
            .get()
            .uri("/stores")
            .retrieve()
            .bodyToMono(String.class);
    }
}
```


URI需要使用虚拟主机名（即服务名，而不是主机名）。 `LoadBalancerClient` 用于创建完整的物理地址。


警告：此方法现已弃用。我们建议您使用带有反应式负载均衡器的 WebFlux。

### 2.8.忽略网络接口


有时，忽略某些命名网络接口很有用，以便将它们排除在服务发现注册之外（例如，在 Docker 容器中运行时）。可以设置正则表达式列表来忽略所需的网络接口。以下配置忽略 `docker0` 接口以及以 `veth` 开头的所有接口：


示例 2.application.yml

```yaml
spring:
  cloud:
    inetutils:
      ignoredInterfaces:
        - docker0
        - veth.*
```


您还可以通过使用正则表达式列表强制仅使用指定的网络地址，如以下示例所示：


示例 3.bootstrap.yml

```yaml
spring:
  cloud:
    inetutils:
      preferredNetworks:
        - 192.168
        - 10.0
```


您还可以强制仅使用站点本地地址，如以下示例所示：


示例 4.application.yml

```yaml
spring:
  cloud:
    inetutils:
      useOnlySiteLocalInterfaces: true
```


有关站点本地地址的构成的更多详细信息，请参阅 Inet4Address.html.isSiteLocalAddress()。

### 2.9. HTTP 客户端工厂


Spring Cloud Commons 提供了用于创建 Apache HTTP 客户端 ( `ApacheHttpClientFactory` ) 和 OK HTTP 客户端 ( `OkHttpClientFactory` ) 的 bean。仅当 OK HTTP jar 位于类路径上时才会创建 `OkHttpClientFactory` bean。此外，Spring Cloud Commons 还提供了用于创建两个客户端使用的连接管理器的 bean： `ApacheHttpClientConnectionManagerFactory` 用于 Apache HTTP 客户端， `OkHttpClientConnectionPoolFactory` 用于 OK HTTP 客户端。如果您想自定义在下游项目中创建 HTTP 客户端的方式，您可以提供您自己的这些 bean 实现。此外，如果您提供 `HttpClientBuilder` 或 `OkHttpClient.Builder` 类型的 bean，默认工厂将使用这些构建器作为返回下游项目的构建器的基础。您还可以通过将 `spring.cloud.httpclientfactories.apache.enabled` 或 `spring.cloud.httpclientfactories.ok.enabled` 设置为 `false` 来禁用这些 Bean 的创建。

### 2.10.启用的功能


Spring Cloud Commons 提供了一个 `/features` 执行器端点。此端点返回类路径上可用的功能以及它们是否已启用。返回的信息包括功能类型、名称、版本和供应商。

#### 2.10.1.特征类型


“特征”有两种类型：抽象特征和命名特征。


抽象功能是定义接口或抽象类并创建实现的功能，例如 `DiscoveryClient` 、 `LoadBalancerClient` 或 `LockService` 。抽象类或接口用于在上下文中查找该类型的 bean。显示的版本是 `bean.getClass().getPackage().getImplementationVersion()` 。


命名功能是没有实现特定类的功能。这些功能包括“Circuit Breaker”、“API Gateway”、“Spring Cloud Bus”等。这些功能需要名称和 bean 类型。

#### 2.10.2.声明功能


任何模块都可以声明任意数量的 `HasFeature` bean，如以下示例所示：

```java
@Bean
public HasFeatures commonsFeatures() {
  return HasFeatures.abstractFeatures(DiscoveryClient.class, LoadBalancerClient.class);
}

@Bean
public HasFeatures consulFeatures() {
  return HasFeatures.namedFeatures(
    new NamedFeature("Spring Cloud Bus", ConsulBusAutoConfiguration.class),
    new NamedFeature("Circuit Breaker", HystrixCommandAspect.class));
}

@Bean
HasFeatures localFeatures() {
  return HasFeatures.builder()
      .abstractFeature(Something.class)
      .namedFeature(new NamedFeature("Some Other Feature", Someother.class))
      .abstractFeature(Somethingelse.class)
      .build();
}
```


这些 bean 中的每一个都应该放在适当保护的 `@Configuration` 中。

### 2.11. Spring Cloud兼容性验证


由于部分用户在设置Spring Cloud应用程序时遇到问题，我们决定添加兼容性验证机制。如果您当前的设置与 Spring Cloud 要求不兼容，它将崩溃，并附上一份报告，显示到底出了什么问题。


目前，我们验证哪个版本的 Spring Boot 已添加到您的类路径中。


报告示例

```
***************************
APPLICATION FAILED TO START
***************************

Description:

Your project setup is incompatible with our requirements due to following reasons:

- Spring Boot [2.1.0.RELEASE] is not compatible with this Spring Cloud release train


Action:

Consider applying the following actions:

- Change Spring Boot version to one of the following versions [1.2.x, 1.3.x] .
You can find the latest Spring Boot versions here [https://spring.io/projects/spring-boot#learn].
If you want to learn more about the Spring Cloud Release train compatibility, you can visit this page [https://spring.io/projects/spring-cloud#overview] and check the [Release Trains] section.
```


要禁用此功能，请将 `spring.cloud.compatibility-verifier.enabled` 设置为 `false` 。如果要覆盖兼容的 Spring Boot 版本，只需使用逗号分隔的兼容 Spring Boot 版本列表设置 `spring.cloud.compatibility-verifier.compatible-boot-versions` 属性即可。

## 3. Spring Cloud LoadBalancer


Spring Cloud 提供了自己的客户端负载均衡器抽象和实现。对于负载均衡机制，添加了 `ReactiveLoadBalancer` 接口，并为其提供了基于Round-Robin和Random的实现。为了让实例从反应式中进行选择，使用了 `ServiceInstanceListSupplier` 。目前，我们支持基于服务发现的 `ServiceInstanceListSupplier` 实现，它使用类路径中提供的发现客户端从服务发现中检索可用实例。

> 可以通过将 `spring.cloud.loadbalancer.enabled` 的值设置为 `false` 来禁用 Spring Cloud LoadBalancer。

### 3.1. LoadBalancer 上下文的预加载


Spring Cloud LoadBalancer 为每个服务 id 创建一个单独的 Spring 子上下文。默认情况下，每当对服务 ID 的第一个请求进行负载平衡时，这些上下文都会被延迟初始化。


您可以选择立即加载这些上下文。为此，请使用 `spring.cloud-loadbalancer.eager-load.clients` 属性指定要执行预加载的服务 ID。

### 3.2.负载均衡算法之间的切换


默认情况下使用的 `ReactiveLoadBalancer` 实现是 `RoundRobinLoadBalancer` 。要切换到不同的实现（对于选定的服务或所有服务），您可以使用自定义 LoadBalancer 配置机制。


例如，可以通过 `@LoadBalancerClient` 注释传递以下配置以切换为使用 `RandomLoadBalancer` ：

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(Environment environment,
            LoadBalancerClientFactory loadBalancerClientFactory) {
        String name = environment.getProperty(LoadBalancerClientFactory.PROPERTY_NAME);
        return new RandomLoadBalancer(loadBalancerClientFactory
                .getLazyProvider(name, ServiceInstanceListSupplier.class),
                name);
    }
}
```

> 作为 `@LoadBalancerClient` 或 `@LoadBalancerClients` 配置参数传递的类不应使用 `@Configuration` 进行注释，或者位于组件扫描范围之外。

### 3.3. Spring Cloud LoadBalancer integrations集成


为了方便使用 Spring Cloud LoadBalancer，我们提供了可与 `WebClient` 配合使用的 `ReactorLoadBalancerExchangeFilterFunction` 以及与 `RestTemplate` > .您可以在以下部分中查看更多信息和使用示例：

- [Spring RestTemplate as a Load Balancer Client](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#rest-template-loadbalancer-client)
- [Spring WebClient as a Load Balancer Client](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#webclinet-loadbalancer-client)
- [Spring WebFlux WebClient with `ReactorLoadBalancerExchangeFilterFunction`](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#webflux-with-reactive-loadbalancer)

### 3.4. Spring Cloud LoadBalancer 缓存


除了每次必须选择实例时通过 `DiscoveryClient` 检索实例的基本 `ServiceInstanceListSupplier` 实现之外，我们还提供了两种缓存实现。

#### 3.4.1. Caffeine 支持的 LoadBalancer 缓存实现


如果类路径中有 `com.github.ben-manes.caffeine:caffeine` ，则将使用基于咖啡因的实现。有关如何配置它的信息，请参阅 LoadBalancerCacheConfiguration 部分。


如果您使用 Caffeine，还可以通过在 `spring.cloud.loadbalancer.cache.caffeine.spec` 属性中传递您自己的 Caffeine 规范来覆盖 LoadBalancer 的默认 Caffeine 缓存设置。


警告：传递您自己的 Caffeine 规范将覆盖任何其他 LoadBalancerCache 设置，包括常规 LoadBalancer 缓存配置字段，例如 `ttl` 和 `capacity` 。

#### 3.4.2.默认 LoadBalancer 缓存实现


如果类路径中没有 Caffeine，则将使用 `spring-cloud-starter-loadbalancer` 自动附带的 `DefaultLoadBalancerCache` 。有关如何配置它的信息，请参阅 LoadBalancerCacheConfiguration 部分。

> 要使用 Caffeine 而不是默认缓存，请将 `com.github.ben-manes.caffeine:caffeine` 依赖项添加到类路径。

#### 3.4.3.负载均衡器缓存配置


您可以设置自己的 `ttl` 值（写入后的时间，在此之后条目应过期），表示为 `Duration` ，通过传递符合 Spring 的 `String` 将 `String` 引导到 `Duration` 转换器语法。作为 `spring.cloud.loadbalancer.cache.ttl` 属性的值。您还可以通过设置 `spring.cloud.loadbalancer.cache.capacity` 属性的值来设置自己的LoadBalancer缓存初始容量。


默认设置包括将 `ttl` 设置为 35 秒，默认 `initialCapacity` 为 `256` 。


您还可以通过将 `spring.cloud.loadbalancer.cache.enabled` 的值设置为 `false` 来完全禁用 loadBalancer 缓存。

> 尽管基本的非缓存实现对于原型设计和测试很有用，但它的效率比缓存版本低得多，因此我们建议在生产中始终使用缓存版本。如果 `DiscoveryClient` 实现已完成缓存，例如 `EurekaDiscoveryClient` ，则应禁用负载均衡器缓存以防止双重缓存。
>
> 当您创建自己的配置时，如果您使用 `CachingServiceInstanceListSupplier` ，请确保将其直接放置在通过网络检索实例的供应商之后的层次结构中，例如 `DiscoveryClientServiceInstanceListSupplier` ，位于任何之前其他过滤供应商。

### 3.5.加权负载平衡


为了启用加权负载平衡，我们提供 `WeightedServiceInstanceListSupplier` 。我们使用 `WeightFunction` 来计算每个实例的权重。默认情况下，我们尝试从元数据映射中读取并解析权重（键为 `weight` ）。


如果元数据映射中没有指定权重，我们默认该实例的权重为1。


您可以通过将 `spring.cloud.loadbalancer.configurations` 的值设置为 `weighted` 或提供您自己的 `ServiceInstanceListSupplier` bean 来配置它，例如：

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withWeighted()
                    .withCaching()
                    .build(context);
    }
}
```

> 您还可以通过提供 `WeightFunction` 来自定义权重计算逻辑。


您可以使用此示例配置使所有实例具有随机权重：

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withWeighted(instance -> ThreadLocalRandom.current().nextInt(1, 101))
                    .withCaching()
                    .build(context);
    }
}
```

### 3.6.基于区域的负载平衡

为了启用基于区域的负载平衡，我们提供 `ZonePreferenceServiceInstanceListSupplier` 。我们使用 `DiscoveryClient` 特定的 `zone` 配置（例如 `eureka.instance.metadata-map.zone` ）来选择客户端尝试过滤可用服务实例的区域。

> 您还可以通过设置 `spring.cloud.loadbalancer.zone` 属性的值来覆盖 `DiscoveryClient` 特定区域设置。
>
> 目前，仅使用 Eureka Discovery 客户端来设置 LoadBalancer 区域。对于其他发现客户端，请设置 `spring.cloud.loadbalancer.zone` 属性。更多仪器即将推出。
>
>  为了确定检索到的 `ServiceInstance` 的区域，我们检查其元数据映射中 `"zone"` 键下的值。




`ZonePreferenceServiceInstanceListSupplier` 过滤检索到的实例并仅返回同一区域内的实例。如果区域是 `null` 或者同一区域内没有实例，则返回所有检索到的实例。


为了使用基于区域的负载平衡方法，您必须在自定义配置中实例化 `ZonePreferenceServiceInstanceListSupplier` bean。


我们使用委托来处理 `ServiceInstanceListSupplier` bean。我们建议使用 `DiscoveryClientServiceInstanceListSupplier` 委托，用 `CachingServiceInstanceListSupplier` 包装它以利用 LoadBalancer 缓存机制，然后将生成的 bean 传递到 `ZonePreferenceServiceInstanceListSupplier` 的构造函数中。


您可以使用此示例配置来进行设置：

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withCaching()
                    .withZonePreference()
                    .build(context);
    }
}
```

### 3.7. LoadBalancer 的实例健康检查


可以为 LoadBalancer 启用计划的运行状况检查。 `HealthCheckServiceInstanceListSupplier` 就是为此提供的。它定期验证委托 `ServiceInstanceListSupplier` 提供的实例是否仍然存在，并且只返回健康的实例，除非没有 - 然后它返回所有检索到的实例。

> 这种机制在使用 `SimpleDiscoveryClient` 时特别有用。对于由实际服务注册表支持的客户端，没有必要使用它，因为我们在查询外部 ServiceDiscovery 后已经获得了健康的实例。
>
>  还建议将此供应商设置为每个服务具有少量实例，以避免在失败的实例上重试调用。
>
> 如果使用任何服务发现支持的供应商，通常不需要添加此运行状况检查机制，因为我们直接从服务注册表检索实例的运行状况。
>
>  `HealthCheckServiceInstanceListSupplier` 依赖于委托通量提供的更新实例。在极少数情况下，当您想要使用不刷新实例的委托时，即使实例列表可能发生变化（例如我们提供的 `DiscoveryClientServiceInstanceListSupplier` ），您可以设置 `spring.cloud.loadbalancer.health-check.refetch-instances` 以使实例列表由 `HealthCheckServiceInstanceListSupplier` 刷新。然后，您还可以通过修改 `spring.cloud.loadbalancer.health-check.refetch-instances-interval` 的值来调整重新刷新间隔，并选择通过将 `spring.cloud.loadbalancer.health-check.repeat-health-check` 设置为 `false` 来禁用额外的运行状况检查重复，因为每个实例重新提取都会还会触发健康检查。

`HealthCheckServiceInstanceListSupplier` 使用以 `spring.cloud.loadbalancer.health-check` 为前缀的属性。您可以为调度程序设置 `initialDelay` 和 `interval` 。您可以通过设置 `spring.cloud.loadbalancer.health-check.path.default` 属性的值来设置运行状况检查 URL 的默认路径。您还可以通过设置 `spring.cloud.loadbalancer.health-check.path.[SERVICE_ID]` 属性的值，并将 `[SERVICE_ID]` 替换为您的服务的正确 ID，为任何给定服务设置特定值。如果未指定 `[SERVICE_ID]` ，则默认使用 `/actuator/health` 。如果 `[SERVICE_ID]` 设置为 `null` 或为空，则不会执行健康检查。您还可以通过设置 `spring.cloud.loadbalancer.health-check.port` 的值来设置运行状况检查请求的自定义端口。如果未设置，则请求的服务在服务实例上可用的端口。

>  如果您依赖于默认路径 ( `/actuator/health` )，请确保将 `spring-boot-starter-actuator` 添加到协作者的依赖项中，除非您计划自己添加此类端点。
>
> 默认情况下， `healthCheckFlux` 将在已检索到的每个活动 `ServiceInstance` 上发出。您可以通过将 `spring.cloud.loadbalancer.health-check.update-results-list` 的值设置为 `false` 来修改此行为。如果此属性设置为 `false` ，则整个活动实例序列首先被收集到列表中，然后才发出，这确保通量不会在属性中设置的运行状况检查间隔之间发出值。


为了使用运行状况检查调度程序方法，您必须在自定义配置中实例化 `HealthCheckServiceInstanceListSupplier` bean。


我们使用委托来处理 `ServiceInstanceListSupplier` bean。我们建议在 `HealthCheckServiceInstanceListSupplier` 的构造函数中传递 `DiscoveryClientServiceInstanceListSupplier` 委托。


您可以使用此示例配置来进行设置：

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withHealthChecks()
                    .build(context);
        }
    }
```

> 对于非反应式堆栈，使用 `withBlockingHealthChecks()` 创建此供应商。您还可以传递自己的 `WebClient` 或 `RestTemplate` 实例以用于检查。
>
>  `HealthCheckServiceInstanceListSupplier` 有自己的基于 Reactor Flux `replay()` 的缓存机制。因此，如果正在使用它，您可能需要跳过用 `CachingServiceInstanceListSupplier` 包装该供应商。
>
>  当您创建自己的配置 `HealthCheckServiceInstanceListSupplier` 时，请确保在任何其他过滤之前将其直接放置在通过网络检索实例的供应商之后的层次结构中，例如 `DiscoveryClientServiceInstanceListSupplier` 。供应商。

### 3.8. LoadBalancer 的相同实例首选项


您可以设置 LoadBalancer，使其优先选择之前选择的实例（如果该实例可用）。


为此，您需要使用 `SameInstancePreferenceServiceInstanceListSupplier` 。您可以通过将 `spring.cloud.loadbalancer.configurations` 的值设置为 `same-instance-preference` 或提供您自己的 `ServiceInstanceListSupplier` bean 来配置它 - 例如：

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withSameInstancePreference()
                    .build(context);
        }
    }
```

> 这也是 Zookeeper `StickyRule` 的替代品。

### 3.9. LoadBalancer 基于请求的粘性会话


您可以设置 LoadBalancer，使其优先使用请求 cookie 中提供的 `instanceId` 实例。如果请求通过 `ClientRequestContext` 或 `ServerHttpRequestContext` 传递到 LoadBalancer（SC LoadBalancer 交换过滤器函数和过滤器使用这些请求），我们目前支持此操作。


为此，您需要使用 `RequestBasedStickySessionServiceInstanceListSupplier` 。您可以通过将 `spring.cloud.loadbalancer.configurations` 的值设置为 `request-based-sticky-session` 或提供您自己的 `ServiceInstanceListSupplier` bean 来配置它 - 例如：

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withRequestBasedStickySession()
                    .build(context);
        }
    }
```


对于该功能，在转发请求之前更新选定的服务实例（如果原始请求 cookie 中的服务实例不可用，则该实例可能与原始请求 cookie 中的服务实例不同）很有用。为此，请将 `spring.cloud.loadbalancer.sticky-session.add-service-instance-cookie` 的值设置为 `true` 。


默认情况下，cookie 的名称是 `sc-lb-instance-id` 。您可以通过更改 `spring.cloud.loadbalancer.instance-id-cookie-name` 属性的值来修改它。

>  目前，WebClient 支持的负载平衡支持此功能。

### 3.10. Spring Cloud LoadBalancer 提示


Spring Cloud LoadBalancer 允许您设置 `String` 提示，这些提示将传递到 `Request` 对象内的 LoadBalancer，并且稍后可以在可以处理它们的 `ReactiveLoadBalancer` 实现中使用。


您可以通过设置 `spring.cloud.loadbalancer.hint.default` 属性的值来为所有服务设置默认提示。您还可以通过设置 `spring.cloud.loadbalancer.hint.[SERVICE_ID]` 属性的值，并将 `[SERVICE_ID]` 替换为您的服务的正确 ID，为任何给定服务设置特定值。如果用户未设置提示，则使用 `default` 。

### 3.11.基于提示的负载平衡


我们还提供了 `HintBasedServiceInstanceListSupplier` ，它是基于提示的实例选择的 `ServiceInstanceListSupplier` 实现。


`HintBasedServiceInstanceListSupplier` 检查提示请求标头（默认标头名称为 `X-SC-LB-Hint` ，但您可以通过更改 `spring.cloud.loadbalancer.hint-header-name` 属性的值来修改它），并且，如果找到提示请求标头，则使用标头中传递的提示值来过滤服务实例。


如果未添加提示标头， `HintBasedServiceInstanceListSupplier` 将使用属性中的提示值来过滤服务实例。


如果未通过标头或属性设置提示，则返回委托提供的所有服务实例。


过滤时， `HintBasedServiceInstanceListSupplier` 查找在其 `metadataMap` 中的 `hint` 键下设置了匹配值的服务实例。如果没有找到匹配的实例，则返回委托提供的所有实例。


您可以使用以下示例配置来进行设置：

```java
public class CustomLoadBalancerConfiguration {

    @Bean
    public ServiceInstanceListSupplier discoveryClientServiceInstanceListSupplier(
            ConfigurableApplicationContext context) {
        return ServiceInstanceListSupplier.builder()
                    .withDiscoveryClient()
                    .withCaching()
                    .withHints()
                    .build(context);
    }
}
```

### 3.12.转换负载平衡的 HTTP 请求


您可以使用所选的 `ServiceInstance` 来转换负载平衡的HTTP请求。


对于 `RestTemplate` ，您需要实现并定义 `LoadBalancerRequestTransformer` ，如下所示：

```java
@Bean
public LoadBalancerRequestTransformer transformer() {
    return new LoadBalancerRequestTransformer() {
        @Override
        public HttpRequest transformRequest(HttpRequest request, ServiceInstance instance) {
            return new HttpRequestWrapper(request) {
                @Override
                public HttpHeaders getHeaders() {
                    HttpHeaders headers = new HttpHeaders();
                    headers.putAll(super.getHeaders());
                    headers.add("X-InstanceId", instance.getInstanceId());
                    return headers;
                }
            };
        }
    };
}
```


对于 `WebClient` ，您需要实现并定义 `LoadBalancerClientRequestTransformer` ，如下所示：

```java
@Bean
public LoadBalancerClientRequestTransformer transformer() {
    return new LoadBalancerClientRequestTransformer() {
        @Override
        public ClientRequest transformRequest(ClientRequest request, ServiceInstance instance) {
            return ClientRequest.from(request)
                    .header("X-InstanceId", instance.getInstanceId())
                    .build();
        }
    };
}
```


如果定义了多个转换器，它们将按照定义 Bean 的顺序应用。或者，您可以使用 `LoadBalancerRequestTransformer.DEFAULT_ORDER` 或 `LoadBalancerClientRequestTransformer.DEFAULT_ORDER` 指定顺序。

### 3.13. Spring Cloud LoadBalancer 入门


我们还提供了一个启动器，允许您在 Spring Boot 应用程序中轻松添加 Spring Cloud LoadBalancer。为了使用它，只需将 `org.springframework.cloud:spring-cloud-starter-loadbalancer` 添加到构建文件中的 Spring Cloud 依赖项中。

>  Spring Cloud LoadBalancer 启动器包括 Spring Boot Caching 和 Evictor。

### 3.14. 传递您自己的 Spring Cloud LoadBalancer 配置


您还可以使用 `@LoadBalancerClient` 注释来传递您自己的负载均衡器客户端配置，传递负载均衡器客户端的名称和配置类，如下所示：

```java
@Configuration
@LoadBalancerClient(value = "stores", configuration = CustomLoadBalancerConfiguration.class)
public class MyConfiguration {

    @Bean
    @LoadBalanced
    public WebClient.Builder loadBalancedWebClientBuilder() {
        return WebClient.builder();
    }
}
```

> 为了使您自己的 LoadBalancer 配置更容易，我们在 `ServiceInstanceListSupplier` 类中添加了 `builder()` 方法。
>
> 您还可以使用我们的替代预定义配置来代替默认配置，方法是将 `spring.cloud.loadbalancer.configurations` 属性的值设置为 `zone-preference` 以使用 `ZonePreferenceServiceInstanceListSupplier` 进行缓存或设置为 `health-check` 将 `HealthCheckServiceInstanceListSupplier` 与缓存一起使用。


您可以使用此功能实例化 `ServiceInstanceListSupplier` 或 `ReactorLoadBalancer` 的不同实现（由您编写或由我们作为替代方案提供（例如 `ZonePreferenceServiceInstanceListSupplier` ）来覆盖）默认设置。


您可以在此处查看自定义配置的示例。

> 注释 `value` 参数（上例中的 `stores` ）指定我们应该使用给定的自定义配置向其发送请求的服务的服务 ID。


您还可以通过 `@LoadBalancerClients` 注释传递多个配置（针对多个负载均衡器客户端），如以下示例所示：

```java
@Configuration
@LoadBalancerClients({@LoadBalancerClient(value = "stores", configuration = StoresLoadBalancerClientConfiguration.class), @LoadBalancerClient(value = "customers", configuration = CustomersLoadBalancerClientConfiguration.class)})
public class MyConfiguration {

    @Bean
    @LoadBalanced
    public WebClient.Builder loadBalancedWebClientBuilder() {
        return WebClient.builder();
    }
}
```

>  作为 `@LoadBalancerClient` 或 `@LoadBalancerClients` 配置参数传递的类不应使用 `@Configuration` 进行注释，或者位于组件扫描范围之外。
>
> 当您创建自己的配置时，如果您使用 `CachingServiceInstanceListSupplier` 或 `HealthCheckServiceInstanceListSupplier` ，请确保使用其中之一，而不是同时使用两者，并确保将其直接放在供应商之后的层次结构中在任何其他过滤提供者之前通过网络检索实例，例如 `DiscoveryClientServiceInstanceListSupplier` 。

### 3.15. Spring Cloud LoadBalancer 生命周期


使用自定义 LoadBalancer 配置注册可能有用的一种 bean 类型是 `LoadBalancerLifecycle` 。


`LoadBalancerLifecycle` bean 提供名为 `onStart(Request<RC> request)` 、 `onStartRequest(Request<RC> request, Response<T> lbResponse)` 和 `onComplete(CompletionContext<RES, T, RC> completionContext)` 的回调方法，您应该实现这些方法来指定之前应该执行哪些操作以及负载均衡之后。


`onStart(Request<RC> request)` 将 `Request` 对象作为参数。它包含用于选择适当实例的数据，包括下游客户端请求和提示。 `onStartRequest` 还采用 `Request` 对象以及 `Response<T>` 对象作为参数。另一方面， `CompletionContext` 对象被提供给 `onComplete(CompletionContext<RES, T, RC> completionContext)` 方法。它包含 LoadBalancer `Response` ，包括选定的服务实例、针对该服务实例执行的请求的 `Status` 以及（如果可用）返回到下游客户端的响应，以及（如果发生异常）对应的 `Throwable` 。

`supports(Class requestContextClass, Class responseClass, Class serverTypeClass)` 方法可用于确定相关处理器是否处理所提供类型的对象。如果用户没有覆盖，它将返回 `true` 。

> 在上述方法调用中， `RC` 表示 `RequestContext` 类型， `RES` 表示客户端响应类型， `T` 表示返回的服务器类型。

### 3.16. Spring Cloud LoadBalancer 统计


我们提供了一个名为 `MicrometerStatsLoadBalancerLifecycle` 的 `LoadBalancerLifecycle` bean，它使用 Micrometer 提供负载平衡调用的统计信息。


为了将此 bean 添加到您的应用程序上下文中，请将 `spring.cloud.loadbalancer.stats.micrometer.enabled` 的值设置为 `true` 并让 `MeterRegistry` 可用（例如，通过添加 Spring将执行器启动到您的项目）。


`MicrometerStatsLoadBalancerLifecycle` 在 `MeterRegistry` 中注册以下计量表：

- `loadbalancer.requests.active`: 一个仪表，允许您监视任何服务实例当前活动请求的数量（通过标签提供的服务实例数据）；
- `loadbalancer.requests.success`:一个计时器，用于测量已结束将响应传递给底层客户端的任何负载平衡请求的执行时间；
- `loadbalancer.requests.failed`: 一个计时器，用于测量任何以异常结束的负载均衡请求的执行时间；
- `loadbalancer.requests.discard`: 一个计数器，用于测量丢弃的负载平衡请求的数量，即 LoadBalancer 尚未检索到运行请求的服务实例的请求。


有关服务实例、请求数据和响应数据的附加信息将在可用时通过标签添加到指标中。

> 对于某些实现，例如 `BlockingLoadBalancerClient` ，请求和响应数据可能不可用，因为我们从参数建立通用类型，并且可能无法确定类型并读取数据。
>
> 当为给定仪表添加至少一条记录时，仪表就会在注册表中注册。
>
> 您可以通过添加 `MeterFilters` 进一步配置这些指标的行为（例如，添加发布百分位数和直方图）。

### 3.17. 配置单独的 LoadBalancerClient


可以使用不同的前缀 `spring.cloud.loadbalancer.clients.<clientId>.` 单独配置各个负载均衡器客户端，其中 `clientId` 是负载均衡器的名称。默认配置值可以在 `spring.cloud.loadbalancer.` 命名空间中设置，并将优先与客户端特定值合并


示例 5.application.yml

```
spring:
  cloud:
    loadbalancer:
      health-check:
        initial-delay: 1s
      clients:
        myclient:
          health-check:
            interval: 30s
```


上面的示例将生成一个包含 `initial-delay=1s` 和 `interval=30s` 的合并运行状况检查 `@ConfigurationProperties` 对象。


除以下全局属性外，每个客户端配置属性适用于大多数属性：

- `spring.cloud.loadbalancer.enabled` - 全局启用或禁用负载平衡
- `spring.cloud.loadbalancer.retry.enabled` -全局启用或禁用负载平衡重试。如果全局启用它，您仍然可以使用 `client` 前缀属性禁用特定客户端的重试，但反之则不然
- `spring.cloud.loadbalancer.cache.enabled` - 全局启用或禁用 LoadBalancer 缓存。如果全局启用它，您仍然可以通过创建在 `ServiceInstanceListSupplier` 委托层次结构中不包含 `CachingServiceInstanceListSupplier` 的自定义配置来禁用特定客户端的缓存，但反之则不然。
- `spring.cloud.loadbalancer.stats.micrometer.enabled` - 全局启用或禁用 LoadBalancer Micrometer 指标

> 对于已使用映射的属性，您可以在其中为每个客户端指定不同的值，而无需使用 `clients` 关键字（例如 `hints` 、 `health-check.path` ） ，我们保留了这种行为，以保持库向后兼容。它将在下一个主要版本中进行修改。
>
> 从 `4.0.4` 开始，我们在 `LoadBalancerProperties` 中引入了 `callGetWithRequestOnDelegates` 标志。如果此标志设置为 `true` ，则将实现 `ServiceInstanceListSupplier#get(Request request)` 方法来调用可从 `DelegatingServiceInstanceListSupplier` 分配的类中的 `delegate.get(request)` ，而该类尚未分配实现该方法，排除 `CachingServiceInstanceListSupplier` 和 `HealthCheckServiceInstanceListSupplier` ，它们应该在供应商通过网络执行实例检索之后、在任何基于请求的过滤之前直接放置在实例供应商层次结构中已经完成了。对于 `4.0.x` ，该标志默认设置为 `false` ，但是，由于 `4.1.0` 它将默认设置为 `true` 。

### 3.18. AOT 和本机映像支持


从 `4.0.0` 开始，Spring Cloud LoadBalancer 支持 Spring AOT 转换和原生镜像。但是，要使用此功能，您需要显式定义您的 `LoadBalancerClient` 服务 ID。您可以通过使用 `@LoadBalancerClient` 注释的 `value` 或 `name` 属性或作为 `spring.cloud.loadbalancer.eager-load.clients` 属性的值来执行此操作。

## 4. Spring Cloud 断路器

### 4.1.介绍


Spring Cloud Circuit Breaker 提供了跨不同断路器实现的抽象。它提供了在您的应用程序中使用的一致 API，让您（开发人员）可以选择最适合您的应用程序需求的断路器实现。

#### 4.1.1.支持的实施


Spring Cloud 支持以下断路器实现：

- [Resilience4J](https://github.com/resilience4j/resilience4j)
- [Sentinel](https://github.com/alibaba/Sentinel)
- [Spring Retry](https://github.com/spring-projects/spring-retry)

### 4.2.核心概念


要在代码中创建断路器，您可以使用 `CircuitBreakerFactory` API。当您在类路径中包含 Spring Cloud Circuit Breaker 启动器时，会自动为您创建一个实现此 API 的 bean。以下示例显示了如何使用此 API 的简单示例：

```java
@Service
public static class DemoControllerService {
    private RestTemplate rest;
    private CircuitBreakerFactory cbFactory;

    public DemoControllerService(RestTemplate rest, CircuitBreakerFactory cbFactory) {
        this.rest = rest;
        this.cbFactory = cbFactory;
    }

    public String slow() {
        return cbFactory.create("slow").run(() -> rest.getForObject("/slow", String.class), throwable -> "fallback");
    }

}
```


`CircuitBreakerFactory.create` API 创建名为 `CircuitBreaker` 的类的实例。 `run` 方法采用 `Supplier` 和 `Function` 。 `Supplier` 是您要包装在断路器中的代码。 `Function` 是断路器跳闸时运行的后备措施。该函数传递了导致触发回退的 `Throwable` 。如果您不想提供后备方案，则可以选择排除后备方案。

#### 4.2.1.反应式代码中的断路器


如果 Project Reactor 位于类路径上，您还可以使用 `ReactiveCircuitBreakerFactory` 作为反应式代码。以下示例展示了如何执行此操作：

```java
@Service
public static class DemoControllerService {
    private ReactiveCircuitBreakerFactory cbFactory;
    private WebClient webClient;


    public DemoControllerService(WebClient webClient, ReactiveCircuitBreakerFactory cbFactory) {
        this.webClient = webClient;
        this.cbFactory = cbFactory;
    }

    public Mono<String> slow() {
        return webClient.get().uri("/slow").retrieve().bodyToMono(String.class).transform(
        it -> cbFactory.create("slow").run(it, throwable -> return Mono.just("fallback")));
    }
}
```


`ReactiveCircuitBreakerFactory.create` API 创建名为 `ReactiveCircuitBreaker` 的类的实例。 `run` 方法采用 `Mono` 或 `Flux` 并将其包装在断路器中。您可以选择分析回退 `Function` ，如果断路器跳闸并传递导致故障的 `Throwable` ，则会调用该回退 `Function` 。

### 4.3.配置


您可以通过创建 `Customizer` 类型的 bean 来配置断路器。 `Customizer` 接口有一个方法（称为 `customize` ），它采用 `Object` 进行自定义。


有关如何自定义给定实现的详细信息，请参阅以下文档：

- [Resilience4J](https://docs.spring.io/spring-cloud-commons/spring-cloud-circuitbreaker/current/reference/html/spring-cloud-circuitbreaker.html#configuring-resilience4j-circuit-breakers)
- [Sentinel](https://github.com/alibaba/spring-cloud-alibaba/blob/master/spring-cloud-alibaba-docs/src/main/asciidoc/circuitbreaker-sentinel.adoc#circuit-breaker-spring-cloud-circuit-breaker-with-sentinel—configuring-sentinel-circuit-breakers)
- [Spring Retry](https://docs.spring.io/spring-cloud-circuitbreaker/docs/current/reference/html/spring-cloud-circuitbreaker.html#configuring-spring-retry-circuit-breakers)


某些 `CircuitBreaker` 实现（例如 `Resilience4JCircuitBreaker` ）每次调用 `CircuitBreaker#run` 时都会调用 `customize` 方法。它可能效率低下。在这种情况下，您可以使用 `CircuitBreaker#once` 方法。当多次调用 `customize` 没有意义时（例如，在使用 Resilience4j 的事件的情况下），它非常有用。


下面的例子展示了每个 `io.github.resilience4j.circuitbreaker.CircuitBreaker` 消费事件的方式。

```java
Customizer.once(circuitBreaker -> {
  circuitBreaker.getEventPublisher()
    .onStateTransition(event -> log.info("{}: {}", event.getCircuitBreakerName(), event.getStateTransition()));
}, CircuitBreaker::getName)
```

## 5. CachedRandomPropertySource


Spring Cloud Context 提供了一个 `PropertySource` 来缓存基于键的随机值。除了缓存功能之外，它的工作方式与 Spring Boot 的 `RandomValuePropertySource` 相同。如果您想要一个即使在 Spring 应用程序上下文重新启动后也保持一致的随机值，则此随机值可能很有用。属性值采用 `cachedrandom.[yourkey].[type]` 的形式，其中 `yourkey` 是缓存中的键。 `type` 值可以是 Spring Boot 的 `RandomValuePropertySource` 支持的任何类型。

```properties
myrandom=${cachedrandom.appname.value}
```

## 6. 安全

### 6.1.单点登录

> 在 1.3 版本中，所有 OAuth2 SSO 和资源服务器功能都转移到了 Spring Boot。您可以在 Spring Boot 用户指南中找到文档。

#### 6.1.1. Client Token Relay


如果您的应用程序是面向 OAuth2 客户端的用户（即已声明 `@EnableOAuth2Sso` 或 `@EnableOAuth2Client` ），那么它在 Spring Boot 的请求范围内有一个 `OAuth2ClientContext` 。您可以从此上下文和自动装配的 `OAuth2ProtectedResourceDetails` 创建您自己的 `OAuth2RestTemplate` ，然后上下文将始终将访问令牌转发到下游，并且如果访问令牌过期，也会自动刷新访问令牌。 （这些是 Spring Security 和 Spring Boot 的功能。）

#### 6.1.2. Resource Server Token Relay


如果您的应用有 `@EnableResourceServer` ，您可能希望将传入的令牌下游中继到其他服务。如果您使用 `RestTemplate` 联系下游服务，那么这只是如何使用正确的上下文创建模板的问题。


如果您的服务使用 `UserInfoTokenServices` 来验证传入令牌（即它使用 `security.oauth2.user-info-uri` 配置），那么您可以简单地使用自动装配的 `OAuth2ClientContext` （它将在访问后端代码之前由身份验证过程填充）。同样地（使用 Spring Boot 1.4），您可以注入 `UserInfoRestTemplateFactory` 并在配置中获取其 `OAuth2RestTemplate` 。例如：

MyConfiguration.java

```java
@Bean
public OAuth2RestTemplate restTemplate(UserInfoRestTemplateFactory factory) {
    return factory.getUserInfoRestTemplate();
}
```


然后，此其余模板将具有与身份验证过滤器使用的相同的 `OAuth2ClientContext` （请求范围），因此您可以使用它来发送具有相同访问令牌的请求。


如果您的应用程序不使用 `UserInfoTokenServices` 但仍然是客户端（即它声明 `@EnableOAuth2Client` 或 `@EnableOAuth2Sso` ），则使用 Spring Security Cloud 任何 `OAuth2RestOperations` 创建的 b3> 也将转发令牌。该功能默认作为 MVC 处理程序拦截器实现，因此仅适用于 Spring MVC。如果您不使用 MVC，则可以使用自定义过滤器或包装 `AccessTokenContextRelay` 的 AOP 拦截器来提供相同的功能。


这是一个基本示例，显示了在其他地方创建的自动连线休息模板的使用（“foo.com”是一个资源服务器，接受与周围应用程序相同的令牌）：

MyController.java

```java
@Autowired
private OAuth2RestOperations restTemplate;

@RequestMapping("/relay")
public String relay() {
    ResponseEntity<String> response =
      restTemplate.getForEntity("https://foo.com/bar", String.class);
    return "Success! (" + response.getBody() + ")";
}
```


如果您不想转发令牌（这是一个有效的选择，因为您可能想充当自己，而不是向您发送令牌的客户端），那么您只需创建自己的 `OAuth2Context`


Feign 客户端还会选择一个使用 `OAuth2ClientContext` 的拦截器（如果可用），因此它们还应该在 `RestTemplate` 所在的任何地方进行令牌中继。

## 7. 配置属性


要查看所有 Spring Cloud Commons 相关配置属性的列表，请查看[附录页面](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/appendix.html)。