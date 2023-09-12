## 1. 简介和概述

### 1.1 什么是LoadBalance?

LoadBalancer（负载均衡器）是分布式系统中的关键组件之一，其作用是在多个服务实例之间均衡地分配请求负载，以提高系统的可用性、性能和扩展性。它可以确保每个服务实例都能够平均地处理请求，避免某些实例过载而导致性能下降或服务不可用的情况。

负载均衡器的主要目标是实现请求的智能分发，将请求合理地分配给可用的服务实例。它可以通过多种策略来决定将请求发送给哪个实例，如轮询、随机、加权轮询等。这些策略可以根据实际需求进行配置和调整。

LoadBalancer的作用如下：

1. 提高系统的可用性：通过将请求分发到多个服务实例，负载均衡器可以确保即使某个实例出现故障或不可用，系统仍能继续提供服务，从而提高系统的可用性。
2. 提升系统的性能和吞吐量：负载均衡器可以根据实时的负载情况，将请求分发到负载较轻的实例上，从而避免单个实例过载，提高系统的性能和吞吐量。
3. 实现水平扩展：通过将请求均匀地分发到多个实例上，负载均衡器可以支持系统的水平扩展。当负载增加时，可以通过增加实例数量来处理更多的请求负载，从而实现系统的弹性扩展能力。
4. 实现负载分担：负载均衡器可以根据实例的处理能力和负载情况，合理地分配请求负载，确保每个实例都能够平均地处理请求，避免某些实例过载而导致性能下降或服务不可用。
5. 故障转移和容错处理：当某个实例发生故障或不可用时，负载均衡器可以自动将请求重新路由到其他可用的实例，实现故障转移和容错处理，保证服务的连续性和可靠性。

### 1.2 Spring Cloud LoadBalance背景简介

Spring Cloud LoadBalance是Spring Cloud生态系统中负责实现负载均衡的组件之一。它构建在Netflix Ribbon之上，提供了一种简单且灵活的方式来实现负载均衡。

从 Spring Cloud 2020 版本开始，官方确实移除了对 Netflix 的依赖，包括 Ribbon 负载均衡组件。相反，官方推荐使用 Spring Cloud LoadBalancer 来替代 Ribbon，并成为 Spring Cloud 负载均衡器的唯一实现。

在微服务架构中，服务通常会以多个实例的形式运行，每个实例都提供相同的服务。为了实现负载均衡，需要一个组件来决定将请求发送给哪个实例。Spring Cloud LoadBalance正是为了解决这个问题而设计的。

Spring Cloud LoadBalance提供了一个集中式的客户端负载均衡器，它可以与服务发现组件（如Eureka、Consul等）集成，自动地从注册中心获取可用的服务实例列表，并根据预定义的负载均衡策略将请求分发给这些实例。

与传统的硬件负载均衡器不同，Spring Cloud LoadBalance运行在服务消费者端，而不是网络层面。这意味着它可以与不同的服务发现机制集成，并与Spring Cloud的其他组件紧密配合，提供更灵活和可扩展的负载均衡解决方案。

### 1.3 Spring Cloud LoadBalance特点和优势

Spring Cloud LoadBalancer 是一个基于 Reactive 编程模型的负载均衡器，它提供了与 Spring WebFlux 和 WebClient 的集成。它具有以下特点和优势：

1. 高度集成化：Spring Cloud LoadBalancer 直接与 Spring Cloud 体系结合，与 Spring Cloud Gateway、Spring Cloud OpenFeign 等组件无缝集成，可以轻松实现负载均衡的功能。
2. 基于 Reactor：Spring Cloud LoadBalancer 是基于 Reactor 编程模型构建的，可以充分利用响应式编程的优势，提供非阻塞的、高效的负载均衡能力。
3. 支持多种负载均衡策略：Spring Cloud LoadBalancer 提供了多种负载均衡策略，包括轮询、随机、加权随机等，在使用时可以根据需求进行配置。
4. 与服务发现整合：Spring Cloud LoadBalancer 与服务发现组件（如 Eureka、Consul 等）无缝整合，可以自动从服务注册中心获取可用的服务实例列表，并根据负载均衡策略进行请求路由。

## 2. 基本使用

1. 添加 Spring Cloud LoadBalancer 依赖：

   在项目的构建配置文件（如 Maven 的 pom.xml）中添加 Spring Cloud LoadBalancer 的依赖。你可以使用以下依赖坐标：

   ```xml
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-loadbalancer</artifactId>
   </dependency>
   ```

   这将引入 Spring Cloud LoadBalancer 相关的库和组件。

2. 配置服务发现组件和参数：

   Spring Cloud LoadBalancer 通常与服务发现组件（如 Eureka、Consul）一起使用，以获取服务实例的信息和状态。你需要在配置文件中指定服务发现组件的相关参数，例如注册中心的地址、端口等。这些参数可以根据你使用的服务发现组件进行相应的配置。

   例如，如果你使用 Eureka 作为服务发现组件，可以在配置文件中添加以下配置：

   ```properties
   eureka.client.service-url.defaultZone=http://eureka-server:8761/eureka/
   ```

   这将指定默认的 Eureka 注册中心的地址。

3. 完成安装和配置：

   在基于 Spring Boot 的应用中，你可以使用 `@LoadBalanced` 注解来配置 RestTemplate 或 WebClient，以启用负载均衡功能。

   - 对于 RestTemplate，你可以使用以下方式进行配置：

   ```java
   @Bean
   @LoadBalanced
   public RestTemplate restTemplate() {
       return new RestTemplate();
   }
   ```

   - 对于 WebClient，你可以使用以下方式进行配置：

   ```java
   @Bean
   @LoadBalanced
   public WebClient.Builder webClientBuilder() {
       return WebClient.builder();
   }
   ```

   上述代码片段创建了一个带有负载均衡功能的 RestTemplate 或 WebClient 实例。RestTemplate是Spring提供的用于访问Rest服务的客户端，RestTemplate提供了多种便捷访问远程Http服务的方法，能够大大提高客户端的编写效率。默认情况下，RestTemplate默认依赖jdk的HTTP连接工具。创建RestTemplateConfig配置类，标注 @LoadBalanced注解，默认使用的ReactiveLoadBalancer实现是RoundRobinLoadBalancer。

4. 发送带有负载均衡的 HTTP 请求：

   现在你可以使用带有负载均衡功能的 RestTemplate 或 WebClient 来发送 HTTP 请求，并自动实现负载均衡。

   - 对于 RestTemplate，你可以使用以下方式发送请求：

     ```java
     ResponseEntity<String> response = restTemplate.getForEntity("http://my-service/my-endpoint", String.class);
     ```

   - 对于 WebClient，你可以使用以下方式发送请求：

     ```java
     WebClient webClient = webClientBuilder.build();
     Mono<String> response = webClient.get()
         .uri("http://my-service/my-endpoint")
         .retrieve()
         .bodyToMono(String.class);
     ```

   在上述代码中，"[http://my-service](http://my-service/)" 是服务的逻辑名称，它将根据负载均衡策略自动选择一个可用的服务实例进行请求转发。

## 3. 工作原理

Spring Cloud LoadBalancer的工作原理主要基于以下几个关键步骤：

1. 服务注册和发现：Spring Cloud LoadBalancer依赖于服务注册和发现组件（如Eureka、Consul等）来获取可用的服务实例列表。在启动时，LoadBalancer会从注册中心获取服务的元数据信息，包括实例的网络地址和健康状态等。
2. 负载均衡策略选择：一旦LoadBalancer获取到可用的服务实例列表，它会根据预定义的负载均衡策略来选择要发送请求的实例。负载均衡策略可以是轮询、随机、加权轮询等，也可以是自定义的策略。
3. 请求转发：LoadBalancer根据选择的负载均衡策略，将请求转发给选定的服务实例。它会将请求重新路由到实例的网络地址，确保请求能够达到正确的服务实例。
4. 容错处理：如果请求发送的实例不可用（如故障或超过预设的故障阈值），LoadBalancer会自动将请求重新路由到其他可用的实例。这种容错处理确保了服务的连续性和可靠性。
5. 动态更新：Spring Cloud LoadBalancer会定期从注册中心获取服务实例的最新信息，以便动态地调整负载均衡策略和实例列表。这样可以适应服务实例的动态变化，保持负载均衡的有效性。

值得注意的是，Spring Cloud LoadBalancer本身并不实现负载均衡算法，而是通过集成具体的负载均衡器实现。默认情况下，它使用Netflix Ribbon作为负载均衡器，但也支持其他负载均衡器的集成，如Nacos、Consul等。

总结起来，Spring Cloud LoadBalancer通过与服务注册和发现组件集成，根据负载均衡策略选择合适的服务实例，并将请求转发给选定的实例。它还具备容错处理和动态更新的能力，以确保服务的连续性和可靠性。这样，开发人员可以专注于业务逻辑，而无需过多关注负载均衡的实现细节。



## 4. 负载均衡策略

### 4.1 常见的负载均衡策略

在负载均衡中，有几种常见的策略可用于决定将请求发送到哪个服务实例。以下是几种常见的负载均衡策略：

1. 轮询（Round Robin）：轮询策略按照顺序依次将请求分发给每个服务实例。每个请求依次循环地发送到不同的实例，以实现请求的均匀分配。这是一种简单且公平的策略，适用于服务实例之间的负载相对均衡的情况。
2. 随机（Random）：随机策略在可用的服务实例中随机选择一个实例来处理每个请求。每个请求的目的地是随机决定的，因此请求的分发是不可预测的。这种策略适用于负载均衡要求不太严格的场景。
3. 加权轮询（Weighted Round Robin）：加权轮询策略根据每个服务实例的权重来分配请求。每个实例被分配的请求量与其权重成比例。具有较高权重的实例将接收到更多的请求，从而处理更多的负载。这种策略适用于希望根据实例的性能或资源配置来分配负载的情况。
4. 加权随机（Weighted Random）：加权随机策略根据每个服务实例的权重进行随机选择。每个实例被选择的概率与其权重成比例。具有较高权重的实例将有更高的概率被选择，从而处理更多的负载。这种策略也适用于根据实例的性能或资源配置来分配负载的情况。
5. 最少连接（Least Connections）：最少连接策略将请求发送到当前连接数最少的服务实例。通过跟踪每个实例的连接数，负载均衡器可以将请求发送到负载较轻的实例，以实现更均衡的负载分配。这种策略适用于处理长连接的场景。
6. IP哈希（IP Hash）：IP哈希策略基于请求的源IP地址来选择服务实例。根据请求的源IP地址进行哈希计算，然后将请求发送到特定的实例。这样，相同IP地址的请求将始终被路由到相同的服务实例上，适用于需要保持会话一致性的场景。

除了上述策略外，还有其他一些高级的负载均衡策略，如基于负载监测的动态权重调整、最短响应时间等。选择适当的负载均衡策略应基于应用程序的需求和特定场景的考虑，以实现最佳的负载均衡效果。

### 4.2 自定义负载均衡策略

1. 实现 `ILoadBalancer` 接口定义自定义负载均衡器：

   在 Spring Cloud LoadBalancer 中，你可以实现 `ILoadBalancer` 接口来定义自己的负载均衡器。该接口提供了一组方法，用于管理服务实例列表、选择下一个要处理请求的实例等。

   你可以根据自己的需求实现 `ILoadBalancer` 接口，并根据具体的负载均衡算法来选择服务实例。

   ```java
   public interface ILoadBalancer {
       void addServers(List<Server> newServers);
       Server chooseServer(Object key);
       void markServerDown(Server server);
       List<Server> getServerList(boolean availableOnly);
   }
   ```

2. 实现自定义的负载均衡器：

   下面是一个简单的示例，展示了如何实现自定义的负载均衡器 `MyLoadBalancer` 的完整代码：

   ```java
   import com.netflix.loadbalancer.Server;
   import org.springframework.cloud.client.ServiceInstance;
   import org.springframework.cloud.client.discovery.DiscoveryClient;
   import org.springframework.cloud.loadbalancer.core.ServiceInstanceListSupplier;
   import org.springframework.cloud.loadbalancer.support.LoadBalancerClientFactory;
   import reactor.core.publisher.Flux;
   
   import java.util.List;
   
   public class MyLoadBalancer implements ILoadBalancer {
   
       private final DiscoveryClient discoveryClient;
       private final String serviceName;
   
       public MyLoadBalancer(LoadBalancerClientFactory loadBalancerClientFactory, String serviceName) {
           this.discoveryClient = loadBalancerClientFactory.getLazyProvider(serviceName, DiscoveryClient.class);
           this.serviceName = serviceName;
       }
   
       @Override
       public void addServers(List<Server> newServers) {
           // 添加服务列表
           // 这里可以根据需要进行服务实例的添加操作
       }
   
       @Override
       public Server chooseServer(Object key) {
           // 根据负载均衡算法选择服务实例
           // 这里可以根据自定义的负载均衡算法选择一个服务实例
           ServiceInstanceListSupplier supplier = new DiscoveryClientServiceInstanceListSupplier(discoveryClient, serviceName);
           List<ServiceInstance> instances = supplier.get().blockFirst();
           if (instances != null && !instances.isEmpty()) {
               ServiceInstance instance = instances.get(0);
               return new Server(instance.getHost(), instance.getPort());
           }
           return null;
       }
   
       @Override
       public void markServerDown(Server server) {
           // 标记服务实例不可用
           // 这里可以根据需要进行服务实例的下线标记操作
       }
   
       @Override
       public List<Server> getServerList(boolean availableOnly) {
           // 获取服务实例列表
           // 这里可以根据需要获取可用或所有的服务实例列表
           ServiceInstanceListSupplier supplier = new DiscoveryClientServiceInstanceListSupplier(discoveryClient, serviceName);
           List<ServiceInstance> instances = supplier.get().blockFirst();
           List<Server> servers = new ArrayList<>();
           if (instances != null && !instances.isEmpty()) {
               for (ServiceInstance instance : instances) {
                   servers.add(new Server(instance.getHost(), instance.getPort()));
               }
           }
           return servers;
       }
   }
   ```

   在上述代码中，`MyLoadBalancer` 类实现了 `ILoadBalancer` 接口，并使用了 Spring Cloud 的 `DiscoveryClient` 来获取服务实例列表。根据自定义的负载均衡算法，可以在 `chooseServer()` 方法中选择一个服务实例。你可以根据自己的需求，修改和扩展该负载均衡器的逻辑。

   请注意，在使用 `MyLoadBalancer` 类时，你需要传入一个 `LoadBalancerClientFactory` 对象和服务名称。这样可以确保在负载均衡器中使用正确的服务发现客户端。

   这只是一个简单的示例，演示了如何实现自定义的负载均衡器。在实际应用中，你可能需要根据具体的负载均衡算法和服务发现组件进行更复杂的实现。

3. 配置自定义负载均衡策略：

   在 Spring Cloud LoadBalancer 中，你可以通过配置文件来指定使用自定义的负载均衡策略。你需要设置 `spring.cloud.loadbalancer.ribbon.NFLoadBalancerRuleClassName` 属性，将其值设置为你自定义负载均衡器的类名。

   ```properties
   spring.cloud.loadbalancer.ribbon.NFLoadBalancerRuleClassName=com.example.MyLoadBalancer
   ```

   上述配置将使用名为 `com.example.MyLoadBalancer` 的自定义负载均衡器。

4. 使用自定义负载均衡策略进行服务调用：

   当配置了自定义负载均衡策略后，Spring Cloud LoadBalancer 将使用你实现的负载均衡器进行服务实例的选择和负载均衡。

   在使用 RestTemplate 或 WebClient 发送请求时，Spring Cloud LoadBalancer 将根据自定义负载均衡策略来选择服务实例。



## 5. 高级配置和扩展

### 5.1 配置服务实例的权重和优先级

在 Spring Cloud LoadBalancer 中，权重和优先级是通过自定义的负载均衡策略实现的。你可以扩展 `AbstractLoadBalancerRule` 类，并在自定义的负载均衡策略中实现权重和优先级的逻辑。

下面是一个示例，展示如何在负载均衡策略中配置服务实例的权重和优先级：

```
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.Server;
import org.springframework.util.CollectionUtils;

import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.atomic.AtomicInteger;

public class WeightedRoundRobinRule extends AbstractLoadBalancerRule {

    private Map<Server, AtomicInteger> serverWeights = new ConcurrentHashMap<>();

    @Override
    public Server choose(Object key) {
        List<Server> allServers = getLoadBalancer().getAllServers();
        if (CollectionUtils.isEmpty(allServers)) {
            return null;
        }

        // 计算总权重
        int totalWeight = 0;
        for (Server server : allServers) {
            int weight = getServerWeight(server);
            totalWeight += weight;
        }

        // 根据权重选择服务实例
        int position = 0;
        for (int i = 0; i < allServers.size(); i++) {
            Server server = allServers.get(i);
            int weight = getServerWeight(server);

            // 使用原子计数器保证线程安全
            AtomicInteger currentWeight = serverWeights.computeIfAbsent(server, k -> new AtomicInteger(0));
            position += weight;

            // 根据权重判断是否选中该服务实例
            if (currentWeight.get() >= position - weight) {
                return server;
            }
        }

        // 如果没有合适的服务实例，则返回第一个服务实例
        return allServers.get(0);
    }

    // 获取服务实例的权重
    private int getServerWeight(Server server) {
        // 这里可以根据具体的逻辑获取服务实例的权重
        // 可以从配置文件、数据库或其他外部系统中获取权重信息
        // 返回默认权重值 1
        return 1;
    }
}
```

在上述示例中，我们自定义了一个负载均衡策略 `WeightedRoundRobinRule`，其中 `getServerWeight()` 方法用于获取服务实例的权重。你可以根据实际需求修改 `getServerWeight()` 方法的实现，例如从配置文件、数据库或其他外部系统中读取服务实例的权重信息。

请注意，以上示例仅展示了如何实现权重的逻辑，实际上，你可以在负载均衡策略中扩展更多的逻辑，例如根据优先级进行服务实例的选择。通过自定义负载均衡策略，你可以根据具体需求灵活配置服务实例的权重和优先级，以实现更精细的负载均衡控制。

### 5.2 使用自定义的服务发现组件

要在 Spring Cloud 中使用自定义的服务发现组件，你需要进行以下步骤：

1. 实现自定义的服务发现客户端：首先，你需要实现一个服务发现客户端，该客户端负责与你的自定义服务发现组件进行通信，获取服务实例的信息。你可以使用 Spring Cloud 的 `ServiceInstance` 类作为服务实例的表示，并将其封装在你的自定义客户端中。
2. 配置自定义的服务发现客户端：在 Spring Cloud 应用程序的配置文件中，你需要指定使用自定义的服务发现客户端。这可以通过设置 `spring.cloud.discovery.client.simple.enabled=false` 来禁用默认的服务发现客户端，然后配置你的自定义客户端的相关属性。
3. 注册自定义的服务发现客户端：将你的自定义服务发现客户端注册为 Spring Bean，以便在应用程序中进行使用。你可以使用 `@Configuration` 注解创建一个配置类，并在该类中定义一个方法，返回你的自定义客户端的实例。

下面是一个简单的示例，展示了如何使用自定义的服务发现组件：

```java
@Configuration
public class CustomDiscoveryConfig {

    @Bean
    public MyCustomDiscoveryClient myCustomDiscoveryClient() {
        // 创建并返回你的自定义服务发现客户端的实例
        return new MyCustomDiscoveryClient();
    }
}
```

然后，在应用程序的配置文件中，禁用默认的服务发现客户端，并指定使用你的自定义客户端：

```properties
spring.cloud.discovery.client.simple.enabled=false
spring.cloud.discovery.client.myCustomDiscovery.enabled=true
```

这样，Spring Cloud 将使用你的自定义服务发现客户端来进行服务发现。

需要注意的是，具体实现和配置可能因你使用的自定义服务发现组件而有所不同。你需要根据你的服务发现组件的文档和要求，进行相应的实现和配置。确保你的自定义服务发现客户端能够与 Spring Cloud 的其他组件（如负载均衡、断路器等）进行集成。

请记住，以上示例仅提供了一个简单的概念性示例，并不代表完整的实现。根据你使用的自定义服务发现组件的要求和特性，你可能需要进行更复杂的实现和配置。

### 5.3 结合其他 Spring Cloud 组件进行高级用法和集成



## 6. 实战示例

- 构建一个微服务架构的示例应用
- 使用 Spring Cloud LoadBalancer 实现服务消费方的负载均衡
- 演示高可用性和容错处理的实际应用



## 7. 性能优化和最佳实践

### 7.1 提供性能优化的建议和最佳实践

当涉及到性能优化和最佳实践时，下面是一些适用于 Spring Cloud 应用程序的建议：

1. 使用适当的负载均衡策略：选择适合你应用程序需求的负载均衡策略。Spring Cloud 中的 Ribbon 提供了多种负载均衡策略，例如轮询、随机、权重等。根据你的应用场景，选择合适的策略来均衡请求负载，并确保各个服务实例能够平衡地处理请求。
2. 合理设置超时和重试：在调用远程服务时，合理设置超时和重试机制。根据你对服务的响应时间和可用性的要求，设置适当的超时时间，并配置重试策略。这将帮助你处理网络延迟、故障和负载过高等情况，提高应用程序的可靠性和性能。
3. 使用服务缓存：对于频繁访问的数据或计算密集型操作，考虑使用服务缓存。通过缓存服务的响应结果，可以减少对远程服务的请求次数，提高响应速度和性能。Spring Cloud 中的 Redis、Ehcache 等缓存组件可以与服务发现和负载均衡组件集成，方便地实现服务缓存。
4. 合理设置连接池大小：对于使用连接池的组件（如 Feign、RestTemplate），根据应用程序的并发需求和服务实例的容量，合理设置连接池的大小。确保连接池具有足够的并发连接数，以满足高并发环境下的请求需求，避免连接池耗尽和性能下降。
5. 监控和度量：使用适当的监控和度量工具来监视和分析你的应用程序的性能。Spring Cloud 提供了 Actuator 组件，它可以集成各种监控工具和指标系统，例如 Prometheus、Grafana、Micrometer 等。通过监控和度量，你可以及时发现性能瓶颈和问题，进行优化和调整。
6. 避免服务调用的过度链式调用：在微服务架构中，服务之间的调用通常会形成调用链。尽量避免过度的链式调用，减少不必要的网络开销和延迟。合理设计服务间的依赖关系，避免出现过于复杂的调用链。
7. 优化数据库访问：对于与数据库交互的操作，合理设计和优化数据库查询语句、索引和连接池等。遵循数据库最佳实践，减少不必要的查询和数据传输，提高数据库访问的效率。
8. 水平扩展和容器化：考虑将应用程序设计为可水平扩展的架构，并部署到容器化平台（如 Kubernetes）。通过水平扩展，可以根据负载情况动态地增加或减少应用实例，以满足不同的并发需求。容器化平台提供了弹性和自动化的部署方式，能够更好地支持应用程序的弹性伸缩和高可用性。

### 7.2 介绍一些常见的负载均衡场景和应对策略

以下是一些常见的负载均衡场景以及相应的应对策略：

1. 高并发请求：当系统面临大量并发请求时，负载均衡可以将请求分发到多个服务实例上，以平衡负载并提高系统的吞吐量。针对这种场景，合适的负载均衡策略可以是基于轮询、随机选择或加权选择等。这样可以确保每个服务实例都能平均地处理请求，避免单个实例过载而导致性能下降。
2. 故障转移和容错：当系统中的某个服务实例出现故障或不可用时，负载均衡可以自动将请求转发到其他可用的实例上，以实现故障转移和容错。常见的应对策略包括使用健康检查机制来监测服务实例的状态，并将请求路由到健康的实例上。这可以提高系统的可用性，并使用户无感知地处理故障情况。
3. 弹性伸缩：当系统的负载增加时，负载均衡可以自动地增加服务实例的数量，以应对增加的请求负载。这种场景通常与自动化的部署和弹性伸缩机制相结合，例如在云平台上使用自动扩展组件。负载均衡可以根据负载情况动态地添加或移除实例，并根据负载均衡策略将请求分发到新增的实例上。
4. 多地域部署：当应用程序在多个地理位置或数据中心部署时，负载均衡可以根据用户的位置或请求来源选择最近的服务实例来处理请求。这称为地理位置负载均衡。这样可以减少延迟，并改善用户体验。常见的应对策略包括使用全局负载均衡器或 CDN（内容分发网络）来路由请求到最近的服务实例。
5. 服务级别的负载均衡：在微服务架构中，不同服务可能具有不同的负载模式和需求。一些服务可能是计算密集型的，而其他服务可能是 I/O 密集型的。针对不同的服务类型，可以采用不同的负载均衡策略或算法。例如，可以为计算密集型服务使用加权轮询策略，而为 I/O 密集型服务使用随机策略。