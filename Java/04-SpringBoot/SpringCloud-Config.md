# Spring Cloud Config

Spring Cloud Config 为分布式系统中的外部化配置提供服务器端和客户端支持。借助配置服务器，您可以在一个中心位置来管理所有环境中应用程序的外部属性。客户端和服务器上的概念与 Spring `Environment` 和 `PropertySource` 抽象映射相同，因此它们非常适合 Spring 应用程序，但可以与以任何语言运行的任何应用程序一起使用。当应用程序通过部署管道从开发到测试再进入生产时，您可以管理这些环境之间的配置，并确保应用程序拥有迁移时运行所需的一切。服务器存储后端的默认实现使用 git，因此它可以轻松支持配置环境的标记版本，并且可以访问各种用于管理内容的工具。添加替代实现并使用 Spring 配置插入它们很容易。

## 快速开始


本快速入门将逐步介绍如何使用 Spring Cloud Config Server 的服务器和客户端。


首先，启动服务器，如下：

```
$ cd spring-cloud-config-server
$ ../mvnw spring-boot:run
```


服务器是一个 Spring Boot 应用程序，因此如果您愿意，可以从 IDE 运行它（主类是 `ConfigServerApplication` ）。


接下来尝试一个客户端，如下：

```
$ curl localhost:8888/foo/development
{
  "name": "foo",
  "profiles": [
    "development"
  ]
  ....
  "propertySources": [
    {
      "name": "https://github.com/spring-cloud-samples/config-repo/foo-development.properties",
      "source": {
        "bar": "spam",
        "foo": "from foo development"
      }
    },
    {
      "name": "https://github.com/spring-cloud-samples/config-repo/foo.properties",
      "source": {
        "foo": "from foo props",
        "democonfigclient.message": "hello spring io"
      }
    },
    ....
```


定位属性源的默认策略是克隆 git 存储库（位于 `spring.cloud.config.server.git.uri` ）并使用它来初始化迷你 `SpringApplication` 。迷你应用程序的 `Environment` 用于枚举属性源并将其发布到 JSON 端点。


HTTP 服务具有以下形式的资源：

```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

例如：

```
curl localhost:8888/foo/development
curl localhost:8888/foo/development/master
curl localhost:8888/foo/development,db/master
curl localhost:8888/foo-development.yml
curl localhost:8888/foo-db.properties
curl localhost:8888/master/foo-db.properties
```


其中 `application` 作为 `SpringApplication` 中的 `spring.config.name` 注入（在常规 Spring Boot 应用程序中通常是 `application` ）， `profile` 是活动配置文件（或逗号分隔的属性列表）， `label` 是可选的 git 标签（默认为 `master` 。）


Spring Cloud Config Server 从各种来源提取远程客户端的配置。以下示例从 git 存储库（必须提供）获取配置，如下例所示：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
```


其他来源包括任何 JDBC 兼容数据库、Subversion、Hashicorp Vault、Credhub 和本地文件系统。

### 客户端使用


要在应用程序中使用这些功能，您可以将其构建为依赖于 spring-cloud-config-client 的 Spring Boot 应用程序（例如，请参阅 config-client 的测试用例或示例应用程序）。添加依赖项最方便的方法是使用 Spring Boot starter `org.springframework.cloud:spring-cloud-starter-config` 。还有一个用于 Maven 用户的父 pom 和 BOM ( `spring-cloud-starter-parent` ) 以及一个用于 Gradle 和 Spring CLI 用户的 Spring IO 版本管理属性文件。以下示例显示了典型的 Maven 配置：

pom.xml

```xml
   <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>{spring-boot-docs-version}</version>
       <relativePath /> <!-- lookup parent from repository -->
   </parent>

<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>{spring-cloud-version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>

<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>

<build>
	<plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
	</plugins>
</build>

   <!-- repositories also needed for snapshots and milestones -->
```


现在您可以创建一个标准的 Spring Boot 应用程序，例如以下 HTTP 服务器：

```
@SpringBootApplication
@RestController
public class Application {

    @RequestMapping("/")
    public String home() {
        return "Hello World!";
    }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```


当此 HTTP 服务器运行时，它会从端口 8888 上的默认本地配置服务器（如果正在运行）获取外部配置。要修改启动行为，您可以使用 `application.properties` 更改配置服务器的位置b0> 如下例所示：

```
spring.config.import=optional:configserver:http://myconfigserver.com
```


默认情况下，如果未设置应用程序名称，将使用 `application` 。要修改名称，可以将以下属性添加到 `application.properties` 文件中：

```
spring.application.name: myapp
```

> 设置属性 `${spring.application.name}` 时，请勿使用保留字 `application-` 作为应用程序名称的前缀，以防止解析正确的属性源时出现问题。


配置服务器属性在 `/env` 端点中显示为高优先级属性源，如以下示例所示。

```
$ curl localhost:8080/env
{
  "activeProfiles": [],
  {
    "name": "servletContextInitParams",
    "properties": {}
  },
  {
    "name": "configserver:https://github.com/spring-cloud-samples/config-repo/foo.properties",
    "properties": {
      "foo": {
        "value": "bar",
        "origin": "Config Server https://github.com/spring-cloud-samples/config-repo/foo.properties:2:12"
      }
    }
  },
  ...
}
```


名为 `configserver:<URL of remote repository>/<file name>` 的属性源包含值为 `bar` 的 `foo` 属性。

> 属性源名称中的 URL 是 git 存储库，而不是配置服务器 URL。
>
> 如果您使用 Spring Cloud Config Client，则需要设置 `spring.config.import` 属性才能绑定到 Config Server。您可以在 Spring Cloud Config 参考指南中阅读更多相关信息。

## Spring Cloud 配置服务器


Spring Cloud Config Server 为外部配置（名称-值对或等效的 YAML 内容）提供基于 HTTP 资源的 API。通过使用 `@EnableConfigServer` 注释，服务器可嵌入到 Spring Boot 应用程序中。因此，以下应用程序是一个配置服务器：

ConfigServer.java

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {
  public static void main(String[] args) {
    SpringApplication.run(ConfigServer.class, args);
  }
}
```


与所有 Spring Boot 应用程序一样，它默认在端口 8080 上运行，但您可以通过各种方式将其切换到更常规的端口 8888。最简单的方法是使用 `spring.config.name=configserver` 启动它（配置服务器 jar 中有一个 `configserver.yml` ），它还设置默认配置存储库。另一种方法是使用您自己的 `application.properties` ，如以下示例所示：

application.properties

```properties
server.port: 8888
spring.cloud.config.server.git.uri: file://${user.home}/config-repo
```


其中 `${user.home}/config-repo` 是包含 YAML 和属性文件的 git 存储库。

> 在 Windows 上，如果文件 URL 是带有驱动器前缀的绝对路径（例如 `file:///${user.home}/config-repo` ），则需要在文件 URL 中添加一个额外的“/”。
>
> 以下清单显示了在前面的示例中创建 git 存储库的方法：`$ cd $HOME $ mkdir config-repo $ cd config-repo $ git init . $ echo info.foo: bar > application.properties $ git add -A . $ git commit -m "Add application.properties"`
>
> 将本地文件系统用于 git 存储库仅用于测试。您应该使用服务器来托管生产中的配置存储库。
>
> 如果您仅在其中保留文本文件，则配置存储库的初始克隆可以快速高效。如果您存储二进制文件，尤其是大文件，则首次配置请求时可能会遇到延迟，或者在服务器中遇到内存不足错误。

### 环境存储库


您应该在哪里存储配置服务器的配置数据？控制此行为的策略是 `EnvironmentRepository` ，服务于 `Environment` 对象。这个 `Environment` 是 Spring `Environment` 域的浅拷贝（包括 `propertySources` 作为主要功能）。 `Environment` 资源由三个变量参数化：

- `{application}` ，映射到客户端的 `spring.application.name` 。
- `{profile}` ，映射到客户端上的 `spring.profiles.active` （逗号分隔列表）。
- `{label}` ，这是一个服务器端功能，标记一组“版本化”配置文件。


存储库实现通常表现得像 Spring Boot 应用程序，从等于 `{application}` 参数的 `spring.config.name` 加载配置文件，以及等于 `{profiles}` 加载配置文件。 b3>参数。配置文件的优先级规则也与常规 Spring Boot 应用程序相同：活动配置文件优先于默认配置文件，如果有多个配置文件，则最后一个获胜（类似于向 `Map` 添加条目） 。


以下示例客户端应用程序具有此引导程序配置：

```yaml
spring:
  application:
    name: foo
  profiles:
    active: dev,mysql
```


（与 Spring Boot 应用程序一样，这些属性也可以通过环境变量或命令行参数设置）。


如果存储库是基于文件的，服务器会从 `application.yml` （在所有客户端之间共享）和 `foo.yml` （与 `foo.yml` 优先）。如果 YAML 文件中包含指向 Spring 配置文件的文档，则这些文件将以更高的优先级应用（按照列出的配置文件的顺序）。如果存在特定于配置文件的 YAML（或属性）文件，这些文件也会以比默认值更高的优先级应用。更高的优先级转化为 `Environment` 中较早列出的 `PropertySource` 。 （这些相同的规则适用于独立的 Spring Boot 应用程序。）


您可以将 `spring.cloud.config.server.accept-empty` 设置为 `false` ，以便服务器在未找到应用程序时返回 HTTP 404 状态。默认情况下，此标志设置为 `true` 。

> 您不能将 `spring.main.*` 属性放置在远程 `EnvironmentRepository` 中。这些属性用作应用程序初始化的一部分。

#### Git 后端


`EnvironmentRepository` 的默认实现使用Git后端，这对于管理升级和物理环境以及审核更改非常方便。要更改存储库的位置，您可以在配置服务器中设置 `spring.cloud.config.server.git.uri` 配置属性（例如在 `application.yml` 中）。如果您使用 `file:` 前缀设置它，它应该在本地存储库中工作，以便您可以在没有服务器的情况下快速轻松地开始。

然而，在这种情况下，服务器直接在本地存储库上操作，而不克隆它（如果它不是裸露的也没关系，因为配置服务器永远不会对“远程”存储库进行更改）。要扩展配置服务器并使其高度可用，您需要让服务器的所有实例都指向同一存储库，因此只有共享文件系统才可以工作。即使在这种情况下，最好对共享文件系统存储库使用 `ssh:` 协议，以便服务器可以克隆它并使用本地工作副本作为缓存。


此存储库实现将 HTTP 资源的 `{label}` 参数映射到 git 标签（提交 ID、分支名称或标记）。如果 git 分支或标记名称包含斜杠 ( `/` )，则应使用特殊字符串 `(_)` 指定 HTTP URL 中的标签（以避免与其他 URL 路径产生歧义） ）。例如，如果标签为 `foo/bar` ，则替换斜杠将产生以下标签： `foo(_)bar` 。特殊字符串 `(_)` 的包含也可以应用于 `{application}` 参数。如果您使用命令行客户端（例如curl），请小心URL 中的括号——您应该使用单引号（''）将它们从shell 中转义。

##### 跳过 SSL 证书验证


可以通过将 `git.skipSslValidation` 属性设置为 `true` （默认为 `false` ）来禁用配置服务器对 Git 服务器 SSL 证书的验证。

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://example.com/my/repo
          skipSslValidation: true
```

##### 设置 HTTP 连接超时


您可以配置配置服务器等待获取 HTTP 连接的时间（以秒为单位）。使用 `git.timeout` 属性（默认为 `5` ）。

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://example.com/my/repo
          timeout: 4
```

##### Git URI 中的占位符


Spring Cloud Config Server 支持 git 存储库 URL，其中包含 `{application}` 和 `{profile}` 占位符（如果需要，还可以使用 `{label}` ，但请记住，标签应用为无论如何，一个 git 标签）。因此，您可以使用类似于以下的结构来支持“每个应用程序一个存储库”策略：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/myorg/{application}
```


您还可以使用类似的模式但使用 `{profile}` 来支持“每个配置文件一个存储库”策略。


此外，在 `{application}` 参数中使用特殊字符串“(_)”可以启用对多个组织的支持，如以下示例所示：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/{application}
```


其中 `{application}` 在请求时以以下格式提供： `organization(_)application` 。

##### 模式匹配和多个存储库


Spring Cloud Config 还通过应用程序和配置文件名称的模式匹配来支持更复杂的需求。模式格式是带有通配符的 `{application}/{profile}` 名称的逗号分隔列表（请注意，以通配符开头的模式可能需要加引号），如以下示例所示：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          repos:
            simple: https://github.com/simple/config-repo
            special:
              pattern: special*/dev*,*special*/dev*
              uri: https://github.com/special/config-repo
            local:
              pattern: local*
              uri: file:/home/configsvc/config-repo
```


如果 `{application}/{profile}` 与任何模式都不匹配，它将使用 `spring.cloud.config.server.git.uri` 下定义的默认 URI。在上面的示例中，对于“简单”存储库，模式为 `simple/*` （它仅匹配所有配置文件中名为 `simple` 的一个应用程序）。 “本地”存储库匹配所有配置文件中以 `local` 开头的所有应用程序名称（ `/*` 后缀会自动添加到没有配置文件匹配器的任何模式）。

> 仅当要设置的唯一属性是 URI 时，才能使用“简单”示例中使用的“单行”快捷方式。如果您需要设置其他任何内容（凭据、模式等），则需要使用完整表单。


存储库中的 `pattern` 属性实际上是一个数组，因此您可以使用 YAML 数组（或属性文件中的 `[0]` 、 `[1]` 等后缀）来绑定到多个模式。如果您要运行具有多个配置文件的应用程序，则可能需要这样做，如下例所示：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          repos:
            development:
              pattern:
                - '*/development'
                - '*/staging'
              uri: https://github.com/development/config-repo
            staging:
              pattern:
                - '*/qa'
                - '*/production'
              uri: https://github.com/staging/config-repo

```

> Spring Cloud 猜测包含不以 `*` 结尾的配置文件的模式意味着您实际上想要匹配以此模式开头的配置文件列表（因此 `*/staging` 是 < b2> 等）。这很常见，例如，您需要在本地“开发”配置文件中运行应用程序，但也需要远程运行“云”配置文件中的应用程序。


每个存储库还可以选择将配置文件存储在子目录中，并且可以将搜索这些目录的模式指定为 `search-paths` 。以下示例显示了顶层的配置文件：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          search-paths:
            - foo
            - bar*
```


在前面的示例中，服务器在顶层和 `foo/` 子目录以及名称以 `bar` 开头的任何子目录中搜索配置文件。


默认情况下，服务器会在首次请求配置时克隆远程存储库。可以将服务器配置为在启动时克隆存储库，如以下顶级示例所示：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://git/common/config-repo.git
          repos:
            team-a:
                pattern: team-a-*
                cloneOnStart: true
                uri: https://git/team-a/config-repo.git
            team-b:
                pattern: team-b-*
                cloneOnStart: false
                uri: https://git/team-b/config-repo.git
            team-c:
                pattern: team-c-*
                uri: https://git/team-a/config-repo.git
```


在前面的示例中，服务器在启动时克隆 team-a 的 config-repo，然后再接受任何请求。在请求存储库的配置之前，不会克隆所有其他存储库。

> 设置在 Config Server 启动时克隆存储库有助于在 Config Server 启动时快速识别配置错误的配置源（例如无效的存储库 URI）。如果未为配置源启用 `cloneOnStart` ，则配置服务器可能会使用错误配置或无效的配置源成功启动，并且在应用程序从该配置源请求配置之前不会检测到错误。

##### Authentication 验证


要在远程存储库上使用 HTTP 基本身份验证，请分别添加 `username` 和 `password` 属性（不在 URL 中），如以下示例所示：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          username: trolley
          password: strongpassword
```


如果您不使用 HTTPS 和用户凭据，则当您将密钥存储在默认目录 ( `~/.ssh` ) 中并且 URI 指向 SSH 位置（例如 `git@github.com:configuration/cloud-configuration` 文件中存在 Git 服务器的条目并且采用 `ssh-rsa` 格式，这一点很重要。不支持其他格式（例如 `ecdsa-sha2-nistp256` ）。为了避免意外，您应该确保 Git 服务器的 `known_hosts` 文件中只存在一个条目，并且它与您提供给配置服务器的 URL 相匹配。如果您在 URL 中使用主机名，则您希望在 `known_hosts` 文件中使用该主机名（而不是 IP）。该存储库是通过使用 JGit 访问的，因此您找到的任何文档都应该适用。 HTTPS 代理设置可以在 `~/.git/config` 中设置，或者（与任何其他 JVM 进程的方式相同）使用系统属性（ `-Dhttps.proxyHost` 和 `-Dhttps.proxyPort` ）进行设置。

> 如果您不知道 `~/.git` 目录在哪里，请使用 `git config --global` 来操作设置（例如 `git config --global http.sslVerify false` ）。


JGit 需要 PEM 格式的 RSA 密钥。下面是一个示例 ssh-keygen （来自 openssh）命令，它将生成正确格式的密钥：

```bash
ssh-keygen -m PEM -t rsa -b 4096 -f ~/config_server_deploy_key.rsa
```


警告：使用 SSH 密钥时，预期的 ssh 私钥必须以 `-----BEGIN RSA PRIVATE KEY-----` 开头。如果密钥以 `-----BEGIN OPENSSH PRIVATE KEY-----` 开头，那么当 spring-cloud-config 服务器启动时，RSA 密钥将不会加载。错误看起来像：

```
- Error in object 'spring.cloud.config.server.git': codes [PrivateKeyIsValid.spring.cloud.config.server.git,PrivateKeyIsValid]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [spring.cloud.config.server.git.,]; arguments []; default message []]; default message [Property 'spring.cloud.config.server.git.privateKey' is not a valid private key]
```


要纠正上述错误，必须将 RSA 密钥转换为 PEM 格式。上面提供了使用 openssh 生成适当格式的新密钥的示例。

##### 使用 AWS CodeCommit 进行身份验证


Spring Cloud Config Server 还支持 AWS CodeCommit 身份验证。从命令行使用 Git 时，AWS CodeCommit 使用身份验证帮助程序。此帮助程序不与 JGit 库一起使用，因此如果 Git URI 与 AWS CodeCommit 模式匹配，则会创建 AWS CodeCommit 的 JGit CredentialProvider。 AWS CodeCommit URI 遵循以下模式：

```bash
https://git-codecommit.${AWS_REGION}.amazonaws.com/v1/repos/${repo}
```


如果您通过 AWS CodeCommit URI 提供用户名和密码，则它们必须是提供存储库访问权限的 AWS accessKeyId 和 SecretAccessKey。如果您不指定用户名和密码，则将使用默认凭证提供程序链检索 accessKeyId 和 SecretAccessKey。


如果您的 Git URI 与 CodeCommit URI 模式（如前文所示）匹配，您必须在用户名和密码中或在默认凭证提供程序链支持的位置之一提供有效的 AWS 凭证。 AWS EC2 实例可以使用 EC2 实例的 IAM 角色。

> `software.amazon.awssdk:auth` jar 是一个可选的依赖项。如果 `software.amazon.awssdk:auth` jar 不在您的类路径中，则无论 git 服务器 URI 是什么，都不会创建 AWS Code Commit 凭证提供程序。

##### 使用 Google Cloud 源进行身份验证


Spring Cloud Config Server 还支持针对 Google Cloud Source 存储库进行身份验证。


如果您的 Git URI 使用 `http` 或 `https` 协议并且域名为 `source.developers.google.com` ，则将使用 Google Cloud Source 凭据提供程序。 Google Cloud Source 存储库 URI 的格式为 `https://source.developers.google.com/p/${GCP_PROJECT}/r/${REPO}` 。要获取存储库的 URI，请单击 Google Cloud Source UI 中的“克隆”，然后选择“手动生成的凭据”。不生成任何凭据，只需复制显示的 URI。


Google Cloud Source 凭据提供程序将使用 Google Cloud Platform 应用程序默认凭据。请参阅 Google Cloud SDK 文档，了解如何为系统创建应用程序默认凭据。此方法适用于开发环境中的用户帐户和生产环境中的服务帐户。

> `com.google.auth:google-auth-library-oauth2-http` 是一个可选的依赖项。如果 `google-auth-library-oauth2-http` jar 不在您的类路径中，则无论 git 服务器 URI 为何，都不会创建 Google Cloud Source 凭据提供程序。

##### 使用属性进行 Git SSH 配置


默认情况下，Spring Cloud Config Server 使用的 JGit 库在使用 SSH URI 连接到 Git 存储库时使用 SSH 配置文件，例如 `~/.ssh/known_hosts` 和 `/etc/ssh/ssh_config` 。在 Cloud Foundry 等云环境中，本地文件系统可能是短暂的或不易访问。对于这些情况，可以使用 Java 属性来设置 SSH 配置。为了激活基于属性的 SSH 配置，必须将 `spring.cloud.config.server.git.ignoreLocalSshSettings` 属性设置为 `true` ，如以下示例所示：

```yaml
  spring:
    cloud:
      config:
        server:
          git:
            uri: git@gitserver.com:team/repo1.git
            ignoreLocalSshSettings: true
            hostKey: someHostKey
            hostKeyAlgorithm: ssh-rsa
            privateKey: |
                         -----BEGIN RSA PRIVATE KEY-----
                         MIIEpgIBAAKCAQEAx4UbaDzY5xjW6hc9jwN0mX33XpTDVW9WqHp5AKaRbtAC3DqX
                         IXFMPgw3K45jxRb93f8tv9vL3rD9CUG1Gv4FM+o7ds7FRES5RTjv2RT/JVNJCoqF
                         ol8+ngLqRZCyBtQN7zYByWMRirPGoDUqdPYrj2yq+ObBBNhg5N+hOwKjjpzdj2Ud
                         1l7R+wxIqmJo1IYyy16xS8WsjyQuyC0lL456qkd5BDZ0Ag8j2X9H9D5220Ln7s9i
                         oezTipXipS7p7Jekf3Ywx6abJwOmB0rX79dV4qiNcGgzATnG1PkXxqt76VhcGa0W
                         DDVHEEYGbSQ6hIGSh0I7BQun0aLRZojfE3gqHQIDAQABAoIBAQCZmGrk8BK6tXCd
                         fY6yTiKxFzwb38IQP0ojIUWNrq0+9Xt+NsypviLHkXfXXCKKU4zUHeIGVRq5MN9b
                         BO56/RrcQHHOoJdUWuOV2qMqJvPUtC0CpGkD+valhfD75MxoXU7s3FK7yjxy3rsG
                         EmfA6tHV8/4a5umo5TqSd2YTm5B19AhRqiuUVI1wTB41DjULUGiMYrnYrhzQlVvj
                         5MjnKTlYu3V8PoYDfv1GmxPPh6vlpafXEeEYN8VB97e5x3DGHjZ5UrurAmTLTdO8
                         +AahyoKsIY612TkkQthJlt7FJAwnCGMgY6podzzvzICLFmmTXYiZ/28I4BX/mOSe
                         pZVnfRixAoGBAO6Uiwt40/PKs53mCEWngslSCsh9oGAaLTf/XdvMns5VmuyyAyKG
                         ti8Ol5wqBMi4GIUzjbgUvSUt+IowIrG3f5tN85wpjQ1UGVcpTnl5Qo9xaS1PFScQ
                         xrtWZ9eNj2TsIAMp/svJsyGG3OibxfnuAIpSXNQiJPwRlW3irzpGgVx/AoGBANYW
                         dnhshUcEHMJi3aXwR12OTDnaLoanVGLwLnkqLSYUZA7ZegpKq90UAuBdcEfgdpyi
                         PhKpeaeIiAaNnFo8m9aoTKr+7I6/uMTlwrVnfrsVTZv3orxjwQV20YIBCVRKD1uX
                         VhE0ozPZxwwKSPAFocpyWpGHGreGF1AIYBE9UBtjAoGBAI8bfPgJpyFyMiGBjO6z
                         FwlJc/xlFqDusrcHL7abW5qq0L4v3R+FrJw3ZYufzLTVcKfdj6GelwJJO+8wBm+R
                         gTKYJItEhT48duLIfTDyIpHGVm9+I1MGhh5zKuCqIhxIYr9jHloBB7kRm0rPvYY4
                         VAykcNgyDvtAVODP+4m6JvhjAoGBALbtTqErKN47V0+JJpapLnF0KxGrqeGIjIRV
                         cYA6V4WYGr7NeIfesecfOC356PyhgPfpcVyEztwlvwTKb3RzIT1TZN8fH4YBr6Ee
                         KTbTjefRFhVUjQqnucAvfGi29f+9oE3Ei9f7wA+H35ocF6JvTYUsHNMIO/3gZ38N
                         CPjyCMa9AoGBAMhsITNe3QcbsXAbdUR00dDsIFVROzyFJ2m40i4KCRM35bC/BIBs
                         q0TY3we+ERB40U8Z2BvU61QuwaunJ2+uGadHo58VSVdggqAo0BSkH58innKKt96J
                         69pcVH/4rmLbXdcmNYGm6iu+MlPQk4BUZknHSmVHIFdJ0EPupVaQ8RHT
                         -----END RSA PRIVATE KEY-----
```


下表描述了 SSH 配置属性。

| Property Name                 | Remarks 评论                                                 |
| :---------------------------- | :----------------------------------------------------------- |
| **ignoreLocalSshSettings**    | 如果 `true` ，则使用基于属性的 SSH 配置，而不是基于文件的 SSH 配置。必须设置为 `spring.cloud.config.server.git.ignoreLocalSshSettings` ，而不是在存储库定义内。 |
| **privateKey**                |                                                              |
| **hostKey **                  | 有效的 SSH 主机密钥。如果还设置了 `hostKeyAlgorithm` ，则必须设置。 |
| **hostKeyAlgorithm **         | `ssh-dss, ssh-rsa, ssh-ed25519, ecdsa-sha2-nistp256, ecdsa-sha2-nistp384, or ecdsa-sha2-nistp521` 之一。如果还设置了 `hostKey` ，则必须设置。 |
| **strictHostKeyChecking **    | 如果为 false，则忽略主机密钥错误。                           |
| **knownHostsFile **           | 自定义 `.known_hosts` 文件的位置。                           |
| **preferredAuthentications ** | 覆盖服务器身份验证方法顺序。如果服务器在 `publickey` 方法之前进行键盘交互式身份验证，这应该允许逃避登录提示。 |

##### Git 搜索路径中的占位符


Spring Cloud Config Server 还支持带有 `{application}` 和 `{profile}` 占位符的搜索路径（如果需要，还支持 `{label}` ），如以下示例所示：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          search-paths: '{application}'
```


前面的列表会导致在存储库中搜索与目录（以及顶层）同名的文件。通配符在带有占位符的搜索路径中也有效（任何匹配的目录都包含在搜索中）。

##### 强制拉入 Git 存储库


如前所述，Spring Cloud Config Server 会克隆远程 git 存储库，以防本地副本变脏（例如，操作系统进程更改文件夹内容），从而导致 Spring Cloud Config Server 无法从远程存储库更新本地副本。


为了解决这个问题，有一个 `force-pull` 属性，如果本地副本脏了，Spring Cloud Config Server 会强制从远程存储库拉取，如下例所示：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          force-pull: true
```


如果您有多个存储库配置，则可以为每个存储库配置 `force-pull` 属性，如以下示例所示：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://git/common/config-repo.git
          force-pull: true
          repos:
            team-a:
                pattern: team-a-*
                uri: https://git/team-a/config-repo.git
                force-pull: true
            team-b:
                pattern: team-b-*
                uri: https://git/team-b/config-repo.git
                force-pull: true
            team-c:
                pattern: team-c-*
                uri: https://git/team-a/config-repo.git
```

>  `force-pull` 属性的默认值为 `false` 。

##### 删除 Git 存储库中未跟踪的分支


由于 Spring Cloud Config Server 在检出分支到本地存储库（例如按标签获取属性）后具有远程 git 存储库的克隆，因此它将永远保留该分支或直到下一次服务器重新启动（这将创建新的本地存储库）。因此，可能会出现远程分支被删除但其本地副本仍可用于获取的情况。如果 Spring Cloud Config Server 客户端服务以 `--spring.cloud.config.label=deletedRemoteBranch,master` 开头，它将从 `deletedRemoteBranch` 本地分支获取属性，而不是从 `master` 获取属性。


为了保持本地存储库分支干净并达到远程 - 可以设置 `deleteUntrackedBranches` 属性。它将使 Spring Cloud Config Server 强制从本地存储库中删除未跟踪的分支。例子：

```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          deleteUntrackedBranches: true
```

|      | The default value for `deleteUntrackedBranches` property is `false`. `deleteUntrackedBranches` 属性的默认值为 `false` 。 |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

##### Git 刷新率


您可以使用 `spring.cloud.config.server.git.refreshRate` 控制配置服务器从 Git 后端获取更新配置数据的频率。该属性的值以秒为单位指定。默认情况下，该值为 0，这意味着配置服务器每次请求时都会从 Git 存储库获取更新的配置。

##### 默认标签


Git 使用的默认标签是 `main` 。如果您未设置 `spring.cloud.config.server.git.defaultLabel` 并且名为 `main` 的分支不存在，则配置服务器默认情况下也会尝试签出名为 `master` 的分支。如果您想禁用后备分支行为，可以将 `spring.cloud.config.server.git.tryMasterBranch` 设置为 `false` 。

#### 版本控制后端文件系统使用

使用基于 VCS 的后端（git、svn），文件可以检出或克隆到本地文件系统。默认情况下，它们放置在系统临时目录中，前缀为 `config-repo-` 。例如，在 Linux 上，它可能是 `/tmp/config-repo-<randomid>` 。某些操作系统会定期清理临时目录。这可能会导致意外行为，例如丢失属性。要避免此问题，请通过将 `spring.cloud.config.server.git.basedir` 或 `spring.cloud.config.server.svn.basedir` 设置为不驻留在系统临时结构中的目录来更改 Config Server 使用的目录。

#### 文件系统后端


配置服务器中还有一个“本机”配置文件，它不使用 Git，而是从本地类路径或文件系统（您想要使用 `spring.cloud.config.server.native.searchLocations` 指向的任何静态 URL）加载配置文件。要使用本机配置文件，请使用 `spring.profiles.active=native` 启动配置服务器。

请记住对文件资源使用 `file:` 前缀（没有前缀的默认值通常是类路径）。与任何 Spring Boot 配置一样，您可以嵌入 `${}` 样式的环境占位符，但请记住 Windows 中的绝对路径需要额外的 `/` （例如 `file:///${user.home}/config-repo` ）。

`searchLocations` 的默认值与本地 Spring Boot 应用程序相同（即 `[classpath:/, classpath:/config, file:./, file:./config]` ）。这不会将服务器中的 `application.properties` 公开给所有客户端，因为服务器中存在的任何属性源在发送到客户端之前都会被删除。

文件系统后端非常适合快速入门和测试。要在生产中使用它，您需要确保文件系统可靠并在配置服务器的所有实例之间共享。


搜索位置可以包含 `{application}` 、 `{profile}` 和 `{label}` 占位符。通过这种方式，您可以隔离路径中的目录并选择对您有意义的策略（例如每个应用程序的子目录或每个配置文件的子目录）。


如果您不在搜索位置中使用占位符，此存储库还会将 HTTP 资源的 `{label}` 参数附加到搜索路径上的后缀，以便从每个搜索位置和具有以下名称的子目录加载属性文件：与标签同名（在 Spring 环境中，带标签的属性优先）。因此，没有占位符的默认行为与添加以 `/{label}/` 结尾的搜索位置相同。例如， `file:/tmp/config` 与 `file:/tmp/config,file:/tmp/config/{label}` 相同。可以通过设置 `spring.cloud.config.server.native.addLabelLocations=false` 禁用此行为。

#### Vault后端


Spring Cloud Config Server 还支持 Vault 作为后端。


Vault 是一种安全访问秘密的工具。秘密是您想要严格控制访问的任何内容，例如 API 密钥、密码、证书和其他敏感信息。 Vault 为任何秘密提供统一的接口，同时提供严格的访问控制并记录详细的审核日志。


有关 Vault 的更多信息，请参阅 Vault 快速入门指南。


要使配置服务器能够使用 Vault 后端，您可以使用 `vault` 配置文件运行配置服务器。例如，在配置服务器的 `application.properties` 中，您可以添加 `spring.profiles.active=vault` 。


默认情况下，Spring Cloud Config Server 使用基于令牌的身份验证从 Vault 获取配置。 Vault 还支持其他身份验证方法，例如 AppRole、LDAP、JWT、CloudFoundry、Kubernetes Auth。为了使用除 TOKEN 或 X-Config-Token 标头之外的任何身份验证方法，我们需要在类路径上放置 Spring Vault Core，以便 Config Server 可以将身份验证委托给该库。请将以下依赖项添加到您的配置服务器应用程序中。

```
Maven (pom.xml)
<dependencies>
	<dependency>
		<groupId>org.springframework.vault</groupId>
		<artifactId>spring-vault-core</artifactId>
	</dependency>
</dependencies>
Gradle (build.gradle)
dependencies {
    implementation "org.springframework.vault:spring-vault-core"
}
```


默认情况下，配置服务器假定您的 Vault 服务器在 `http://127.0.0.1:8200` 运行。它还假设后端的名称是 `secret` ，键是 `application` 。所有这些默认值都可以在配置服务器的 `application.properties` 中配置。下表描述了可配置的 Vault 属性：

| Name              | Default Value 默认值 |
| :---------------- | :------------------- |
| host              | 127.0.0.1            |
| port              | 8200                 |
| scheme            | http                 |
| backend           | secret               |
| defaultKey        | application          |
| profileSeparator  | ,                    |
| kvVersion         | 1                    |
| skipSslValidation | false                |
| timeout           | 5                    |
| namespace         | null                 |

> 上表中的所有属性都必须以 `spring.cloud.config.server.vault` 为前缀，或者放置在复合配置的正确 Vault 部分中。


所有可配置的属性都可以在 `org.springframework.cloud.config.server.environment.VaultEnvironmentProperties` 中找到。

>  Vault 0.10.0 引入了版本化的键值后端（k/v 后端版本 2），它公开了与早期版本不同的 API，它现在需要在挂载路径和实际上下文路径之间有一个 `data/` 并换行 `data` 对象中的秘密。设置 `spring.cloud.config.server.vault.kv-version=2` 将考虑到这一点。


（可选）支持 Vault Enterprise `X-Vault-Namespace` 标头。要将其发送到 Vault，请设置 `namespace` 属性。


配置服务器运行时，您可以向服务器发出 HTTP 请求以从 Vault 后端检索值。为此，您需要 Vault 服务器的令牌。


首先，将一些数据放入 Vault 中，如下例所示：

```sh
$ vault kv put secret/application foo=bar baz=bam
$ vault kv put secret/myapp foo=myappsbar
```


其次，向配置服务器发出 HTTP 请求以检索值，如以下示例所示：

```
$ curl -X "GET" "http://localhost:8888/myapp/default" -H "X-Config-Token: yourtoken"
```


您应该看到类似于以下内容的响应：

```json
{
   "name":"myapp",
   "profiles":[
      "default"
   ],
   "label":null,
   "version":null,
   "state":null,
   "propertySources":[
      {
         "name":"vault:myapp",
         "source":{
            "foo":"myappsbar"
         }
      },
      {
         "name":"vault:application",
         "source":{
            "baz":"bam",
            "foo":"bar"
         }
      }
   ]
}
```


客户端提供必要的身份验证以让 Config Server 与 Vault 通信的默认方法是设置 X-Config-Token 标头。但是，您可以通过设置与 Spring Cloud Vault 相同的配置属性来省略标头并在服务器中配置身份验证。要设置的属性是 `spring.cloud.config.server.vault.authentication` 。应将其设置为支持的身份验证方法之一。您可能还需要设置特定于您使用的身份验证方法的其他属性，方法是使用 `spring.cloud.vault` 中记录的相同属性名称，但使用 `spring.cloud.config.server.vault` 前缀。有关更多详细信息，请参阅 Spring Cloud Vault 参考指南。

> 如果省略 X-Config-Token 标头并使用服务器属性来设置身份验证，则 Config Server 应用程序需要额外依赖 Spring Vault 才能启用其他身份验证选项。有关如何添加该依赖项的信息，请参阅 Spring Vault 参考指南。

##### 多种属性来源


使用 Vault 时，您可以为应用程序提供多个属性源。例如，假设您已将数据写入 Vault 中的以下路径：

```sh
secret/myApp,dev
secret/myApp
secret/application,dev
secret/application
```


写入 `secret/application` 的属性可供所有使用配置服务器的应用程序使用。名称为 `myApp` 的应用程序将具有写入 `secret/myApp` 和 `secret/application` 的可用属性。当 `myApp` 启用 `dev` 配置文件时，写入上述所有路径的属性都可用，列表中第一个路径中的属性优先于其他路径。

#### 通过代理访问后端


配置服务器可以通过 HTTP 或 HTTPS 代理访问 Git 或 Vault 后端。对于 Git 或 Vault，此行为由 `proxy.http` 和 `proxy.https` 下的设置控制。这些设置针对每个存储库，因此如果您使用复合环境存储库，则必须为复合环境中的每个后端单独配置代理设置。如果使用的网络需要单独的 HTTP 和 HTTPS URL 代理服务器，您可以为单个后端配置 HTTP 和 HTTPS 代理设置：在这种情况下 `http` 访问将使用 `http` 代理和 `https` 访问 `https` 之一。此外，您还可以使用应用程序和代理之间的代理定义协议指定一个单独的代理，该代理将用于这两种协议。


下表描述了 HTTP 和 HTTPS 代理的代理配置属性。所有这些属性都必须以 `proxy.http` 或 `proxy.https` 为前缀。

| Property Name      | Remarks                                                      |
| :----------------- | :----------------------------------------------------------- |
| **host **          | 代理的主机。                                                 |
| **port **          | 用于访问代理的端口。                                         |
| **nonProxyHosts ** | 配置服务器应在代理外部访问的任何主机。如果为 `proxy.http.nonProxyHosts` 和 `proxy.https.nonProxyHosts` 都提供了值，则将使用 `proxy.http` 值。 |
| **username **      | 用于向代理进行身份验证的用户名。如果为 `proxy.http.username` 和 `proxy.https.username` 都提供了值，则将使用 `proxy.http` 值。 |
| **password **      | 用于向代理进行身份验证的密码。如果为 `proxy.http.password` 和 `proxy.https.password` 都提供了值，则将使用 `proxy.http` 值。 |


以下配置使用 HTTPS 代理来访问 Git 存储库。

```yaml
spring:
  profiles:
    active: git
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-cloud-samples/config-repo
          proxy:
            https:
              host: my-proxy.host.io
              password: myproxypassword
              port: '3128'
              username: myproxyusername
              nonProxyHosts: example.com
```

#### 与所有应用程序共享配置


在所有应用程序之间共享配置根据您采用的方法而有所不同，如以下主题中所述：

- [File Based Repositories](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#spring-cloud-config-server-file-based-repositories)
- [Vault Server](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#spring-cloud-config-server-vault-server)

##### 基于文件的存储库


对于基于文件（git、svn 和本机）的存储库，文件名位于 `application*` 中的资源（ `application.properties` 、 `application.yml` 、 `application-*.properties` 等）在所有客户端应用程序之间共享。您可以使用具有这些文件名的资源来配置全局默认值，并根据需要让它们被特定于应用程序的文件覆盖。


属性覆盖功能还可用于设置全局默认值，占位符应用程序允许在本地覆盖它们。

> 对于“本机”配置文件（本地文件系统后端），您应该使用不属于服务器自身配置的显式搜索位置。否则，默认搜索位置中的 `application*` 资源将被删除，因为它们是服务器的一部分。

##### Vault Server


使用 Vault 作为后端时，您可以通过将配置放在 `secret/application` 中与所有应用程序共享配置。例如，如果您运行以下 Vault 命令，则所有使用配置服务器的应用程序都将具有可用的属性 `foo` 和 `baz` ：

```sh
$ vault write secret/application foo=bar baz=bam
```

##### CredHub服务器


使用 CredHub 作为后端时，您可以通过将配置放置在 `/application/` 中或将其放置在应用程序的 `default` 配置文件中来与所有应用程序共享配置。例如，如果您运行以下 CredHub 命令，则使用配置服务器的所有应用程序都将具有可用的属性 `shared.color1` 和 `shared.color2` ：

```sh
credhub set --name "/application/profile/master/shared" --type=json
value: {"shared.color1": "blue", "shared.color": "red"}
credhub set --name "/my-app/default/master/more-shared" --type=json
value: {"shared.word1": "hello", "shared.word2": "world"}
```

#### AWS 秘密管理器


使用 AWS Secrets Manager 作为后端时，您可以通过将配置放置在 `/application/` 中或将其放置在应用程序的 `default` 配置文件中来与所有应用程序共享配置。例如，如果您使用以下密钥添加机密，则所有使用配置服务器的应用程序都将具有可用的属性 `shared.foo` 和 `shared.bar` ：

```
secret name = /secret/application-default/
secret value =
{
 shared.foo: foo,
 shared.bar: bar
}
```

or

```
secret name = /secret/application/
secret value =
{
 shared.foo: foo,
 shared.bar: bar
}
```

##### 标记版本


AWS Secrets Manager 存储库允许像 Git 后端一样保留配置环境的标记版本。


存储库实现将 HTTP 资源的 `{label}` 参数映射到 AWS Secrets Manager 密钥的暂存标签。要创建带标签的密钥，请创建密钥或更新其内容并为其定义暂存标签（有时在 AWS 文档中称为版本阶段）。例如：

```sh
$ aws secretsmanager create-secret \
      --name /secret/test/ \
      --secret-string '{"version":"1"}'
{
    "ARN": "arn:aws:secretsmanager:us-east-1:123456789012:secret:/secret/test/-a1b2c3",
    "Name": "/secret/test/",
    "VersionId": "cd291674-de2f-41de-8f3b-37dbf4880d69"
}

$ aws secretsmanager update-secret-version-stage \
      --secret-id /secret/test/ \
      --version-stage 1.0.0 \
      --move-to-version-id cd291674-de2f-41de-8f3b-37dbf4880d69

{
    "ARN": "arn:aws:secretsmanager:us-east-1:123456789012:secret:/secret/test/-a1b2c3",
    "Name": "/secret/test/",
}
```


使用 `spring.cloud.config.server.aws-secretsmanager.default-label` 属性设置默认标签。如果未定义该属性，后端将使用 AWSCURRENT 作为暂存标签。

```yaml
spring:
  profiles:
    active: aws-secretsmanager
  cloud:
    config:
      server:
        aws-secretsmanager:
          region: us-east-1
          default-label: 1.0.0
```


请注意，如果未设置默认标签并且请求未定义标签，则存储库将使用机密，就像禁用了标签版本支持一样。此外，仅当启用标签支持时才会使用默认标签。否则，定义这个属性是没有意义的。


请注意，如果暂存标签包含斜杠 ( `/` )，则应使用特殊字符串 `(_)` 指定 HTTP URL 中的标签（以避免与其他 URL 路径产生歧义）与 Git 后端部分描述的方式相同。

#### AWS 参数存储


使用 AWS Parameter Store 作为后端时，您可以通过将属性放置在 `/application` 层次结构中来与所有应用程序共享配置。


例如，如果您添加具有以下名称的参数，则所有使用配置服务器的应用程序都将具有可用的属性 `foo.bar` 和 `fred.baz` ：

```
/config/application/foo.bar
/config/application-default/fred.baz
```

#### JDBC后端


Spring Cloud Config Server 支持 JDBC（关系数据库）作为配置属性的后端。您可以通过将 `spring-boot-starter-data-jdbc` 添加到类路径并使用 `jdbc` 配置文件或添加 `JdbcEnvironmentRepository` 类型的 bean 来启用此功能。如果您在类路径中包含正确的依赖项（有关更多详细信息，请参阅用户指南），Spring Boot 会配置数据源。


您可以通过将 `spring.cloud.config.server.jdbc.enabled` 属性设置为 `false` 来禁用 `JdbcEnvironmentRepository` 的自动配置。


数据库需要有一个名为 `PROPERTIES` 的表，其中包含名为 `APPLICATION` 、 `PROFILE` 和 `LABEL` 的列（通常使用 `Environment` 和 `VALUE` 用于 `Properties` 样式的键和值对。在 Java 中，所有字段都是 String 类型，因此您可以将它们设置为您需要的任何长度 `VARCHAR` 。属性值的行为方式与来自名为 `{application}-{profile}.properties` 的 Spring Boot 属性文件的行为相同，包括所有加密和解密，这些将作为后处理步骤应用（即，不在直接存储库实现）。

>  JDBC 使用的默认标签是 `master` 。您可以通过设置 `spring.cloud.config.server.jdbc.defaultLabel` 来更改它。

#### Redis后端


Spring Cloud Config Server 支持 Redis 作为配置属性的后端。您可以通过添加对 Spring Data Redis 的依赖来启用此功能。

pom.xml

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-data-redis</artifactId>
	</dependency>
</dependencies>
```


以下配置使用 Spring Data `RedisTemplate` 访问 Redis。我们可以使用 `spring.redis.*` 属性来覆盖默认连接设置。

```yaml
spring:
  profiles:
    active: redis
  redis:
    host: redis
    port: 16379
```


这些属性应作为字段存储在哈希中。哈希的名称应与 `spring.application.name` 属性或 `spring.application.name` 和 `spring.profiles.active[n]` 的连接相同。

```sh
HMSET sample-app server.port "8100" sample.topic.name "test" test.property1 "property1"
```


运行上面可见的命令后，哈希值应包含以下键和值：

```
HGETALL sample-app
{
  "server.port": "8100",
  "sample.topic.name": "test",
  "test.property1": "property1"
}
```

> 当未指定配置文件时，将使用 `default` 。

#### AWS S3 后端


Spring Cloud Config Server 支持 AWS S3 作为配置属性的后端。您可以通过添加对适用于 Amazon S3 的 AWS Java 开发工具包的依赖项来启用此功能。

pom.xml

```xml
<dependencies>
	<dependency>
		<groupId>software.amazon.awssdk</groupId>
		<artifactId>s3</artifactId>
	</dependency>
</dependencies>
```


以下配置使用 AWS S3 客户端访问配置文件。我们可以使用 `spring.cloud.config.server.awss3.*` 属性来选择存储配置的存储桶。

```yaml
spring:
  profiles:
    active: awss3
  cloud:
    config:
      server:
        awss3:
          region: us-east-1
          bucket: bucket1
```


还可以指定 AWS URL 以使用 `spring.cloud.config.server.awss3.endpoint` 覆盖 S3 服务的标准终端节点。这允许支持 S3 的 beta 区域以及其他 S3 兼容的存储 API。


凭证是使用默认凭证提供程序链找到的。支持版本化和加密的存储桶，无需进一步配置。


配置文件以 `{application}-{profile}.properties` 、 `{application}-{profile}.yml` 或 `{application}-{profile}.json` 形式存储在您的存储桶中。可以提供可选标签来指定文件的目录路径。

> 当未指定配置文件时，将使用 `default` 。

#### AWS 参数存储后端


Spring Cloud Config Server 支持 AWS Parameter Store 作为配置属性的后端。您可以通过添加对 AWS Java SDK for SSM 的依赖项来启用此功能。

pom.xml

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>ssm</artifactId>
</dependency>
```


以下配置使用 AWS SSM 客户端访问参数。

```yaml
spring:
  profiles:
    active: awsparamstore
  cloud:
    config:
      server:
        awsparamstore:
          region: eu-west-2
          endpoint: https://ssm.eu-west-2.amazonaws.com
          origin: aws:parameter:
          prefix: /config/service
          profile-separator: _
          recursive: true
          decrypt-values: true
          max-results: 5
```


下表描述了 AWS Parameter Store 配置属性。

| Property Name       | Required | Default Value        | Remarks                                                      |
| :------------------ | :------- | :------------------- | :----------------------------------------------------------- |
| **region **         | no       |                      | AWS Parameter Store 客户端要使用的区域。如果未明确设置，SDK 会尝试使用默认区域提供商链来确定要使用的区域。 |
| **endpoint **       | no       |                      | AWS SSM 客户端入口点的 URL。这可用于指定 API 请求的备用端点。 |
| **origin **         | no       | `aws:ssm:parameter:` | 添加到属性源名称以显示其出处的前缀。                         |
| **prefix **         | no       | `/config`            | 指示从 AWS Parameter Store 加载的每个属性的参数层次结构中的 L1 级别的前缀。 |
| **profile-separator | no       | `-`                  | 将附加配置文件与上下文名称分隔开的字符串。                   |
| **recursive **      | no       | `true`               | 用于指示检索层次结构中所有 AWS 参数的标志。                  |
| **decrypt-values**  | no       | `true`               | 指示检索所有 AWS 参数及其值已解密的标志。                    |
| **max-results **    | no       | `10`                 | AWS Parameter Store API 调用返回的最大项目数。               |


AWS Parameter Store API 凭证是使用默认凭证提供程序链确定的。返回最新版本的默认行为已支持版本化参数。

#### AWS Secrets Manager 后端


Spring Cloud Config Server 支持 AWS Secrets Manager 作为配置属性的后端。您可以通过添加对 AWS Java SDK for Secrets Manager 的依赖项来启用此功能。

pom.xml

```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>secretsmanager</artifactId>
</dependency>
```


以下配置使用 AWS Secrets Manager 客户端访问密钥。

```yaml
spring:
  profiles:
  	active: awssecretsmanager
  cloud:
    config:
      server:
        aws-secretsmanager:
          region: us-east-1
          endpoint: https://us-east-1.console.aws.amazon.com/
          origin: aws:secrets:
          prefix: /secret/foo
          profileSeparator: _
```


AWS Secrets Manager API 凭证是使用默认凭证提供商链确定的。

> 当未指定应用程序时， `application` 为默认值，当未指定配置文件时，将使用 `default` 。

#### CredHub 后端


Spring Cloud Config Server 支持 CredHub 作为配置属性的后端。您可以通过向 Spring CredHub 添加依赖项来启用此功能。

pom.xml

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.credhub</groupId>
		<artifactId>spring-credhub-starter</artifactId>
	</dependency>
</dependencies>
```


以下配置使用双向 TLS 访问 CredHub：

```yaml
spring:
  profiles:
    active: credhub
  cloud:
    config:
      server:
        credhub:
          url: https://credhub:8844
```


属性应存储为 JSON，例如：

```sh
credhub set --name "/demo-app/default/master/toggles" --type=json
value: {"toggle.button": "blue", "toggle.link": "red"}
credhub set --name "/demo-app/default/master/abs" --type=json
value: {"marketing.enabled": true, "external.enabled": false}
```


所有名为 `spring.cloud.config.name=demo-app` 的客户端应用程序都将具有以下可用属性：

```
{
    toggle.button: "blue",
    toggle.link: "red",
    marketing.enabled: true,
    external.enabled: false
}
```

>  当未指定配置文件时，将使用 `default` ；当未指定标签时，将使用 `master` 作为默认值。注意：添加到 `application` 的值将由所有应用程序共享。

##### OAuth 2.0


您可以使用 UAA 作为提供者通过 OAuth 2.0 进行身份验证。

pom.xml

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.security</groupId>
		<artifactId>spring-security-config</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.security</groupId>
		<artifactId>spring-security-oauth2-client</artifactId>
	</dependency>
</dependencies>
```


以下配置使用 OAuth 2.0 和 UAA 访问 CredHub：

```yaml
spring:
  profiles:
    active: credhub
  cloud:
    config:
      server:
        credhub:
          url: https://credhub:8844
          oauth2:
            registration-id: credhub-client
  security:
    oauth2:
      client:
        registration:
          credhub-client:
            provider: uaa
            client-id: credhub_config_server
            client-secret: asecret
            authorization-grant-type: client_credentials
        provider:
          uaa:
            token-uri: https://uaa:8443/oauth/token
```

> 使用的 UAA 客户端 ID 应以 `credhub.read` 作为范围。

#### 复合环境存储库


在某些情况下，您可能希望从多个环境存储库中提取配置数据。为此，您可以在配置服务器的应用程序属性或 YAML 文件中启用 `composite` 配置文件。例如，如果您想要从 Subversion 存储库以及两个 Git 存储库中提取配置数据，您可以为配置服务器设置以下属性：

```yaml
spring:
  profiles:
    active: composite
  cloud:
    config:
      server:
        composite:
        -
          type: svn
          uri: file:///path/to/svn/repo
        -
          type: git
          uri: file:///path/to/rex/git/repo
        -
          type: git
          uri: file:///path/to/walter/git/repo
```


使用此配置，优先级由存储库在 `composite` 键下列出的顺序确定。在上面的示例中，Subversion 存储库首先列出，因此在 Subversion 存储库中找到的值将覆盖在 Git 存储库之一中找到的相同属性的值。在 `rex` Git 存储库中找到的值将先于在 `walter` Git 存储库中找到的相同属性的值使用。


如果您只想从不同类型的存储库中提取配置数据，则可以在配置服务器的应用程序属性或 YAML 文件中启用相应的配置文件，而不是 `composite` 配置文件。例如，如果您想要从单个 Git 存储库和单个 HashiCorp Vault 服务器中提取配置数据，您可以为配置服务器设置以下属性：

```yaml
spring:
  profiles:
    active: git, vault
  cloud:
    config:
      server:
        git:
          uri: file:///path/to/git/repo
          order: 2
        vault:
          host: 127.0.0.1
          port: 8200
          order: 1
```


使用此配置，可以通过 `order` 属性确定优先级。您可以使用 `order` 属性指定所有存储库的优先级顺序。 `order` 属性的数值越低，其优先级越高。存储库的优先级顺序有助于解决包含相同属性值的存储库之间的任何潜在冲突。

> 如果您的复合环境包含 Vault 服务器（如上例所示），则必须在向配置服务器发出的每个请求中包含 Vault 令牌。请参阅保管库后端。
>
> 从环境存储库检索值时的任何类型的失败都会导致整个复合环境失败。如果您希望复合在存储库失败时也能继续，您可以将 `spring.cloud.config.server.failOnCompositeError` 设置为 `false` 。
>
>  使用复合环境时，所有存储库都包含相同的标签非常重要。如果您的环境与前面的示例类似，并且您使用 `master` 标签请求配置数据，但 Subversion 存储库不包含名为 `master` 的分支，则整个请求将失败。

##### 自定义复合环境存储库


除了使用 Spring Cloud 中的环境存储库之一之外，您还可以提供自己的 `EnvironmentRepository` bean 作为复合环境的一部分。为此，您的 bean 必须实现 `EnvironmentRepository` 接口。如果您想在复合环境中控制自定义 `EnvironmentRepository` 的优先级，您还应该实现 `Ordered` 接口并重写 `getOrdered` 方法。如果您不实现 `Ordered` 接口，则您的 `EnvironmentRepository` 将获得最低优先级。

#### 属性覆盖


配置服务器具有“覆盖”功能，允许操作员向所有应用程序提供配置属性。使用普通 Spring Boot 挂钩的应用程序不会意外更改覆盖的属性。要声明覆盖，请将名称-值对的映射添加到 `spring.cloud.config.server.overrides` ，如以下示例所示：

```yaml
spring:
  cloud:
    config:
      server:
        overrides:
          foo: bar
```


前面的示例导致作为配置客户端的所有应用程序读取 `foo=bar` ，而与它们自己的配置无关。

> 配置系统不能强制应用程序以任何特定方式使用配置数据。因此，覆盖是不可强制执行的。但是，它们确实为 Spring Cloud Config 客户端提供了有用的默认行为。
>
> 通常，带有 `${}` 的 Spring 环境占位符可以通过使用反斜杠 ( `\` ) 转义 `$` 或 `{` 。例如， `\${app.foo:bar}` 解析为 `bar` ，除非应用程序提供自己的 `app.foo` 。
>
>  在 YAML 中，您不需要转义反斜杠本身。但是，在属性文件中，当您在服务器上配置覆盖时，确实需要转义反斜杠。


您可以将客户端中所有覆盖的优先级更改为更像默认值，让应用程序在环境变量或系统属性中提供自己的值，方法是在远程存储库。

#### 使用 Bootstrap 覆盖属性


如果启用配置优先引导，则可以通过在来自配置服务器的应用程序配置中放置两个属性来允许客户端应用程序覆盖来自配置服务器的配置。

```properties
spring.cloud.config.allowOverride=true
spring.cloud.config.overrideNone=true
```


启用 Bootstrap 并将这两个属性设置为 true 后，您将能够在客户端应用程序配置中覆盖来自配置服务器的配置。

#### 使用占位符覆盖属性


在不启用配置优先引导的情况下覆盖属性的一种更简洁的方法是在来自配置服务器的配置中使用属性占位符。


例如，如果来自配置服务器的配置包含以下属性

```properties
hello=${app.hello:Hello From Config Server!}
```


您可以通过在本地应用程序配置中设置 `app.hello` 来覆盖来自配置服务器的 `hello` 值

```properties
app.hello=Hello From Application!
```

#### 使用配置文件覆盖属性


覆盖来自配置服务器的属性的最后一种方法是在客户端应用程序内的配置文件特定配置文件中指定它们。


例如，如果您从配置服务器具有以下配置

```properties
hello="Hello From Config Server!"
```


您可以通过在配置文件特定的配置文件中设置 `hello` 然后启用该配置文件来覆盖客户端应用程序中 `hello` 的值。

application-overrides.properties

```properties
hello="Hello From Application!"
```


在上面的示例中，您必须启用 `overrides` 配置文件。

### 健康指标


配置服务器附带一个健康指示器，用于检查配置的 `EnvironmentRepository` 是否正常工作。默认情况下，它会向 `EnvironmentRepository` 请求名为 `app` 的应用程序、 `default` 配置文件以及 `EnvironmentRepository` 实现提供的默认标签。


您可以配置运行状况指示器以检查更多应用程序以及自定义配置文件和自定义标签，如下例所示：

```yaml
spring:
  cloud:
    config:
      server:
        health:
          repositories:
            myservice:
              label: mylabel
            myservice-dev:
              name: myservice
              profiles: development
```


您可以通过设置 `management.health.config.enabled=false` 禁用健康指示器。


此外，您还可以通过设置属性 `spring.cloud.config.server.health.down-health-status` （默认值为 `"DOWN'` ）来提供您自己的自定义 `down` 状态。

### 安全


您可以通过任何对您有意义的方式来保护您的配置服务器（从物理网络安全到 OAuth2 不记名令牌），因为 Spring Security 和 Spring Boot 为许多安全安排提供支持。


要使用默认的 Spring Boot 配置的 HTTP Basic 安全性，请在类路径中包含 Spring Security（例如，通过 `spring-boot-starter-security` ）。默认用户名是 `user` 和随机生成的密码。随机密码在实践中没有用，因此我们建议您配置密码（通过设置 `spring.security.user.password` ）并对其进行加密（有关如何执行此操作的说明，请参阅下文）。

### 执行器和安全

某些平台配置运行状况检查或类似的内容并指向 `/actuator/health` 或其他执行器端点。如果执行器不是配置服务器的依赖项，则对 `/actuator/` 的请求将与配置服务器 API `/{application}/{label}` 匹配，可能会泄漏安全信息。请记住在这种情况下添加 `spring-boot-starter-actuator` 依赖项并配置用户，以便调用 `/actuator/` 的用户无法访问 `/{application}/{label}` .

### 加密与解密

要使用加密和解密功能，您需要在 JVM 中安装全强度 JCE（默认情况下不包括）。您可以从 Oracle 下载“Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files”并按照安装说明进行操作（本质上，您需要将 JRE lib/security 目录中的两个策略文件替换为您下载的文件）。


如果远程属性源包含加密内容（以 `{cipher}` 开头的值），则它们会在通过 HTTP 发送到客户端之前被解密。此设置的主要优点是属性值在“静止”时（例如，在 git 存储库中）不需要采用纯文本形式。如果某个值无法解密，则会将其从属性源中删除，并使用相同的密钥添加一个附加属性，但前缀为 `invalid` 和一个表示“不适用”的值（通常为 `<n/a>`


如果您为配置客户端应用程序设置远程配置存储库，它可能包含类似于以下内容的 `application.yml` ：

application.yml

```yaml
spring:
  datasource:
    username: dbuser
    password: '{cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ'
```


`application.properties` 文件中的加密值不得用引号引起来。否则，该值不会被解密。以下示例显示了可行的值：

application.properties

```
spring.datasource.username: dbuser
spring.datasource.password: {cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ
```


您可以安全地将此纯文本推送到共享 git 存储库，并且秘密密码仍然受到保护。


服务器还公开 `/encrypt` 和 `/decrypt` 端点（假设这些端点是安全的并且只能由授权代理访问）。如果编辑远程配置文件，则可以使用配置服务器通过 POST 到 `/encrypt` 端点来加密值，如以下示例所示：

```
$ curl localhost:8888/encrypt -s -d mysecret
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
```

如果您使用curl进行测试，则使用 `--data-urlencode` （而不是 `-d` ）并在要加密的值前面加上 `=` （curl需要此）或设置显式 `Content-Type: text/plain` 确保当存在特殊字符时，curl 能够正确编码数据（“+”特别棘手）。

请确保加密值中不包含任何curl 命令统计信息，这就是示例使用 `-s` 选项来静默它们的原因。将值输出到文件可以帮助避免此问题。


也可以通过 `/decrypt` 进行逆向操作（前提是服务器配置了对称密钥或完整密钥对），如下例所示：

```
$ curl localhost:8888/decrypt -s -d 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
mysecret
```


在将加密值放入 YAML 或属性文件之前以及提交并将其推送到远程（可能不安全）存储之前，请获取加密值并添加 `{cipher}` 前缀。

`/encrypt` 和 `/decrypt` 端点也都接受 `/*/{application}/{profiles}` 形式的路径，它可用于控制每个应用程序（名称）和每个应用程序的加密。 - 客户端调用主环境资源时的配置文件基础。

> 要以这种精细的方式控制加密，您还必须提供 `TextEncryptorLocator` 类型的 `@Bean` ，它为每个名称和配置文件创建不同的加密器。默认提供的密钥不会这样做（所有加密都使用相同的密钥）。


`spring` 命令行客户端（安装了 Spring Cloud CLI 扩展）也可以用于加密和解密，如下例所示：

```
$ spring encrypt mysecret --key foo
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
$ spring decrypt --key foo 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
mysecret
```


要使用文件中的密钥（例如用于加密的 RSA 公钥），请在密钥值前面添加“@”并提供文件路径，如以下示例所示：

```
$ spring encrypt mysecret --key @${HOME}/.ssh/id_rsa.pub
AQAjPgt3eFZQXwt8tsHAVv/QHiY5sI2dRcR+...
```

>  `--key` 参数是强制性的（尽管有 `--` 前缀）。

### 密钥管理


配置服务器可以使用对称（共享）密钥或非对称密钥（RSA 密钥对）。非对称选择在安全性方面更为优越，但使用对称密钥通常更方便，因为它是在 `application.properties` 中配置的单个属性值。


要配置对称密钥，您需要将 `encrypt.key` 设置为秘密字符串（或使用 `ENCRYPT_KEY` 环境变量以使其远离纯文本配置文件）。

 如果您在类路径中包含 `spring-cloud-starter-bootstrap` 或将 `spring.cloud.bootstrap.enabled=true` 设置为系统属性，则需要在 `bootstrap.properties` 中设置 `encrypt.key` 。

> 您无法使用 `encrypt.key` 配置非对称密钥。


要配置非对称密钥，请使用密钥库（例如，由 JDK 附带的 `keytool` 实用程序创建）。密钥库属性为 `encrypt.keyStore.*` ，其中 `*` 等于

|          Property           |              Description              |
| :-------------------------: | :-----------------------------------: |
| `encrypt.keyStore.location` |         包含 `Resource` 位置          |
| `encrypt.keyStore.password` |         保存解锁密钥库的密码          |
|  `encrypt.keyStore.alias`   |      标识要使用商店中的哪个密钥       |
|   `encrypt.keyStore.type`   | 要创建的密钥库的类型。默认为 `jks` 。 |


加密是用公钥完成的，解密时需要私钥。因此，原则上，如果您只想加密（并准备使用私钥在本地自行解密这些值），则可以仅在服务器中配置公钥。实际上，您可能不想在本地进行解密，因为它将密钥管理过程分散在所有客户端上，而不是集中在服务器中。另一方面，如果您的配置服务器相对不安全并且只有少数客户端需要加密的属性，那么它可能是一个有用的选项。

### 创建用于测试的密钥库


要创建用于测试的密钥库，您可以使用类似于以下内容的命令：

```
$ keytool -genkeypair -alias mytestkey -keyalg RSA \
  -dname "CN=Web Server,OU=Unit,O=Organization,L=City,S=State,C=US" \
  -keypass changeme -keystore server.jks -storepass letmein
```

> 当使用 JDK 11 或更高版本时，使用上述命令时可能会收到以下警告。在这种情况下，您可能需要确保 `keypass` 和 `storepass` 值匹配。


将 `server.jks` 文件放入类路径中（例如），然后在 `bootstrap.yml` 中为配置服务器创建以下设置：

```yaml
encrypt:
  keyStore:
    location: classpath:/server.jks
    password: letmein
    alias: mytestkey
    secret: changeme
```

### 使用多个密钥和密钥轮换


除了加密属性值中的 `{cipher}` 前缀之外，配置服务器还会在（Base64 编码的）密文开始之前查找零个或多个 `{name:value}` 前缀。密钥被传递给 `TextEncryptorLocator` ，它可以执行为密码查找 `TextEncryptor` 所需的任何逻辑。如果您已配置密钥库 ( `encrypt.keystore.location` )，则默认定位器会查找具有由 `key` 前缀提供的别名的密钥，其密文类似于以下内容：

```yaml
foo:
  bar: `{cipher}{key:testkey}...`
```


定位器查找名为“testkey”的键。还可以通过在前缀中使用 `{secret:…​}` 值来提供机密。但是，如果未提供，则默认情况下使用密钥库密码（这是在构建密钥库且不指定机密时获得的密码）。如果您确实提供了机密，您还应该使用自定义 `SecretLocator` 来加密该机密。


当密钥仅用于加密几个字节的配置数据（即它们不在其他地方使用）时，从加密角度来看，几乎不需要密钥轮换。但是，您有时可能需要更改密钥（例如，在发生安全漏洞时）。在这种情况下，所有客户端都需要更改其源配置文件（例如，在 git 中）并在所有密码中使用新的 `{key:…​}` 前缀。请注意，客户端需要首先检查配置服务器密钥库中的密钥别名是否可用。

> 如果您想让配置服务器处理所有加密和解密，还可以将 `{name:value}` 前缀添加为发布到 `/encrypt` 端点的纯文本。

### 提供加密属性


有时您希望客户端在本地解密配置，而不是在服务器中解密。在这种情况下，如果您提供 `encrypt.*` 配置来定位密钥，您仍然可以拥有 `/encrypt` 和 `/decrypt` 端点，但您需要显式关闭通过将 `spring.cloud.config.server.encrypt.enabled=false` 放入 `bootstrap.[yml|properties]` 中来解密传出属性。如果您不关心端点，那么如果您不配置密钥或启用标志，它应该可以工作。

### 提供替代格式


环境端点的默认 JSON 格式非常适合 Spring 应用程序使用，因为它直接映射到 `Environment` 抽象。如果您愿意，可以通过向资源路径添加后缀（“.yml”、“.yaml”或“.properties”）来使用与 YAML 或 Java 属性相同的数据。这对于不关心 JSON 端点结构或它们提供的额外元数据的应用程序的使用非常有用（例如，不使用 Spring 的应用程序可能会受益于这种方法的简单性）。


YAML 和属性表示有一个附加标志（作为名为 `resolvePlaceholders` 的布尔查询参数提供）来指示应解析源文档中的占位符（采用标准 Spring `${…​}` 形式）如果可能的话，在渲染之前的输出中。对于不了解 Spring 占位符约定的消费者来说，这是一个有用的功能。

> 使用 YAML 或属性格式存在限制，主要与元数据的丢失有关。例如，JSON 的结构为属性源的有序列表，其名称与源相关。即使值的来源有多个源，YAML 和属性表单也会合并到单个映射中，并且原始源文件的名称会丢失。此外，YAML 表示也不一定是后备存储库中 YAML 源的忠实表示。 It is constructed from a list of flat property sources, and assumptions have to be made about the form of the keys. 它是根据扁平属性源列表构建的，并且必须对密钥的形式做出假设。

### 提供纯文本服务


您的应用程序可能需要针对其环境定制的通用纯文本配置文件，而不是使用 `Environment` 抽象（或其 YAML 或属性格式的替代表示形式之一）。配置服务器通过 `/{application}/{profile}/{label}/{path}` 处的附加端点提供这些信息，其中 `application` 、 `profile` 和 `label` 与常规端点具有相同的含义环境端点，但 `path` 是文件名的路径（例如 `log.xml` ）。此端点的源文件的定位方式与环境端点的源文件的定位方式相同。属性和 YAML 文件使用相同的搜索路径。但是，不会聚合所有匹配的资源，而是仅返回第一个匹配的资源。


找到资源后，将通过使用提供的应用程序名称、配置文件和标签的有效 `Environment` 解析正常格式的占位符 ( `${…​}` )。这样，资源端点与环境端点紧密集成。

> 与环境配置的源文件一样， `profile` 用于解析文件名。因此，如果您想要一个特定于配置文件的文件， `/*/development/*/logback.xml` 可以通过名为 `logback-development.xml` 的文件来解析（优先于 `logback.xml` ）。
>
> 如果您不想提供 `label` 并让服务器使用默认标签，则可以提供 `useDefaultLabel` 请求参数。因此，前面的 `default` 配置文件示例可能是 `/sample/default/nginx.conf?useDefaultLabel` 。


目前，Spring Cloud Config 可以为 git、SVN、本机后端和 AWS S3 提供明文服务。对 git、SVN 和本机后端的支持是相同的。 AWS S3 的工作方式有点不同。以下部分展示了每个部分的工作原理：

- [Git, SVN, and Native Backends
  Git、SVN 和本机后端](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#spring-cloud-config-serving-plain-text-git-svn-native-backends)
- [AWS S3 AWS S3](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/#spring-cloud-config-serving-plain-text-aws-s3)

### 提供二进制文件


为了从配置服务器提供二进制文件，您需要发送 `application/octet-stream` 的 `Accept` 标头。

#### Git、SVN 和本机后端


考虑以下 GIT 或 SVN 存储库或本机后端的示例：

```
application.yml
nginx.conf
```


`nginx.conf` 可能类似于以下清单：

```
server {
    listen              80;
    server_name         ${nginx.server.name};
}
```


`application.yml` 可能类似于以下清单：

```yaml
nginx:
  server:
    name: example.com
---
spring:
  profiles: development
nginx:
  server:
    name: develop.com
```


`/sample/default/master/nginx.conf` 资源可能如下：

```
server {
    listen              80;
    server_name         example.com;
}
```


`/sample/development/master/nginx.conf` 可能如下：

```
server {
    listen              80;
    server_name         develop.com;
}
```

#### AWS S3


要为 AWS s3 提供纯文本服务，Config Server 应用程序需要包含对 `io.awspring.cloud:spring-cloud-aws-context` 的依赖项。有关如何设置该依赖项的详细信息，请参阅 Spring Cloud AWS 参考指南。此外，当将 Spring Cloud AWS 与 Spring Boot 结合使用时，包含自动配置依赖项也很有用。然后，您需要配置 Spring Cloud AWS，如 Spring Cloud AWS 参考指南中所述。

#### 解密纯文本


默认情况下，纯文本文件中的加密值不会被解密。为了启用纯文本文件的解密，请在 `bootstrap.[yml|properties]` 中设置 `spring.cloud.config.server.encrypt.enabled=true` 和 `spring.cloud.config.server.encrypt.plainTextEncrypt=true`

> 仅 YAML、JSON 和属性文件扩展名支持解密纯文本文件。


如果启用此功能，并且请求不受支持的文件扩展名，则文件中的任何加密值都不会被解密。

### 嵌入配置服务器


配置服务器作为独立应用程序运行效果最佳。但是，如果需要，您可以将其嵌入到另一个应用程序中。为此，请使用 `@EnableConfigServer` 注释。在这种情况下，名为 `spring.cloud.config.server.bootstrap` 的可选属性可能很有用。它是一个标志，指示服务器是否应该从自己的远程存储库进行自身配置。默认情况下，该标志处于关闭状态，因为它可以延迟启动。然而，当嵌入到另一个应用程序中时，以与任何其他应用程序相同的方式初始化是有意义的。将 `spring.cloud.config.server.bootstrap` 设置为 `true` 时，您还必须使用复合环境存储库配置。例如

```yaml
spring:
  application:
    name: configserver
  profiles:
    active: composite
  cloud:
    config:
      server:
        composite:
          - type: native
            search-locations: ${HOME}/Desktop/config
        bootstrap: true
```

>  如果您使用 bootstrap 标志，则配置服务器需要在 `bootstrap.yml` 中配置其名称和存储库 URI。


要更改服务器端点的位置，您可以（可选）设置 `spring.cloud.config.server.prefix` （例如 `/config` ），以在前缀下提供资源。前缀应以 `/` 开头但不以 `/` 结尾。它应用于配置服务器中的 `@RequestMappings` （即在 Spring Boot `server.servletPath` 和 `server.contextPath` 前缀下面）。


如果您想直接从后端存储库（而不是从配置服务器）读取应用程序的配置，那么您基本上需要一个没有端点的嵌入式配置服务器。您可以通过不使用 `@EnableConfigServer` 注释（设置 `spring.cloud.config.server.bootstrap=true` ）来完全关闭端点。

### 推送通知和 Spring Cloud Bus


许多源代码存储库提供商（例如 Github、Gitlab、Gitea、Gitee、Gogs 或 Bitbucket）通过 Webhook 通知您存储库中的更改。您可以通过提供商的用户界面将 Webhook 配置为 URL 和您感兴趣的一组事件。例如，Github 使用 POST 发送到 webhook，其 JSON 主体包含提交列表，标头 ( `X-Github-Event` ) 设置为 `push` 。如果您添加对 `spring-cloud-config-monitor` 库的依赖项并在配置服务器中激活 Spring Cloud Bus，则会启用 `/monitor` 端点。


当 webhook 被激活时，配置服务器会发送一个 `RefreshRemoteApplicationEvent` ，针对它认为可能已更改的应用程序。变化检测可以制定策略。但是，默认情况下，它会查找与应用程序名称匹配的文件中的更改（例如， `foo.properties` 的目标是 `foo` 应用程序，而 `application.properties` 的目标是在所有应用程序中）。当您想要覆盖行为时使用的策略是 `PropertyPathNotificationExtractor` ，它接受请求标头和正文作为参数并返回已更改的文件路径列表。


默认配置可直接用于 Github、Gitlab、Gitea、Gitee、Gogs 或 Bitbucket。除了来自 Github、Gitlab、Gitee 或 Bitbucket 的 JSON 通知之外，您还可以通过使用 `path={application}` 模式中的表单编码正文参数 POST 到 `/monitor` 来触发更改通知。这样做会广播到与 `{application}` 模式（可以包含通配符）匹配的应用程序。

> 仅当 `spring-cloud-bus` 在配置服务器和客户端应用程序中均被激活时，才会传输 `RefreshRemoteApplicationEvent` 。
>
>  默认配置还检测本地 git 存储库中的文件系统更改。在这种情况下，不会使用 webhook。但是，一旦您编辑配置文件，就会广播刷新。

### AOT 和本机映像支持


从 `4.0.0` 开始，Spring Cloud Config Server 支持 Spring AOT 转换。不过，暂时不支持 GraalVM 原生镜像。 graal#5134 阻止了实现本机图像支持，并且可能需要完成 https://github.com/graalvm/taming-build-time-initialization 上的工作才能修复。

## Spring Cloud 配置客户端


Spring Boot 应用程序可以立即利用 Spring Config Server（或应用程序开发人员提供的其他外部属性源）。它还获取了一些与 `Environment` 更改事件相关的其他有用功能。

### Spring Boot 配置数据导入


Spring Boot 2.4 引入了一种通过 `spring.config.import` 属性导入配置数据的新方法。现在这是绑定到配置服务器的默认方式。


要选择连接到配置服务器，请在 application.properties 中设置以下内容：

application.properties

```properties
spring.config.import=optional:configserver:
```


这将连接到默认位置“http://localhost:8888”的配置服务器。删除 `optional:` 前缀将导致配置客户端在无法连接到配置服务器时失败。要更改配置服务器的位置，请设置 `spring.cloud.config.uri` 或将 url 添加到 `spring.config.import` 语句，例如 `spring.config.import=optional:configserver:http://myhost:8888` 。 import 属性中的位置优先于 uri 属性。


Spring Boot Config Data 通过两步过程解析配置。首先，它使用 `default` 配置文件加载所有配置。这允许 Spring Boot 收集可能激活任何其他配置文件的所有配置。收集所有激活的配置文件后，它将加载活动配置文件的任何附加配置。因此，您可能会看到向 Spring Cloud Config Server 发出多个请求来获取配置。这是正常现象，是 Spring Boot 在使用 `spring.config.import` 时加载配置的副作用。在以前版本的 Spring Cloud Config 中，仅发出一个请求，但这意味着您无法从来自配置服务器的配置激活配置文件。现在，仅使用“默认”配置文件的附加请求使这成为可能。

> 通过 `spring.config.import` 导入的 Spring Boot 配置数据方法不需要 `bootstrap` 文件（属性或 yaml）。

### 配置第一个引导程序


要使用连接到配置服务器的旧引导程序方式，必须通过属性或 `spring-cloud-starter-bootstrap` 启动器启用引导程序。该属性是 `spring.cloud.bootstrap.enabled=true` 。它必须设置为系统属性或环境变量。启用引导程序后，类路径上具有 Spring Cloud Config Client 的任何应用程序都将连接到配置服务器，如下所示：当配置客户端启动时，它会绑定到配置服务器（通过 `spring.cloud.config.uri` 引导程序配置属性）并使用远程属性源初始化 Spring `Environment` 。


此行为的最终结果是，所有想要使用配置服务器的客户端应用程序都需要一个 `bootstrap.yml` （或环境变量），其服务器地址设置在 `spring.cloud.config.uri` 中（默认为“http://localhost:8888”）。

#### 发现首次查找

除非您使用配置优先引导程序，否则您的配置属性中需要有一个带有 `optional:` 前缀的 `spring.config.import` 属性。例如， `spring.config.import=optional:configserver:` 。


如果您使用 `DiscoveryClient` 实现，例如 Spring Cloud Netflix 和 Eureka Service Discovery 或 Spring Cloud Consul，您可以让 Config Server 向 Discovery Service 注册。


如果您喜欢使用 `DiscoveryClient` 来定位配置服务器，可以通过设置 `spring.cloud.config.discovery.enabled=true` （默认为 `false` ）来实现。例如，使用 Spring Cloud Netflix，您需要定义 Eureka 服务器地址（例如在 `eureka.client.serviceUrl.defaultZone` 中）。使用此选项的代价是启动时需要额外的网络往返，以查找服务注册。好处是，只要发现服务是一个固定点，配置服务器就可以改变它的坐标。默认服务 ID 为 `configserver` ，但您可以通过设置 `spring.cloud.config.discovery.serviceId` 在客户端上更改它（在服务器上，以服务的常用方式，例如通过设置 `spring.application.name` ）。


发现客户端实现都支持某种元数据映射（例如，我们为 Eureka 有 `eureka.instance.metadataMap` ）。可能需要在其服务注册元数据中配置配置服务器的一些附加属性，以便客户端可以正确连接。如果配置服务器使用 HTTP Basic 进行保护，您可以将凭据配置为 `user` 和 `password` 。另外，如果配置服务器有上下文路径，您可以设置 `configPath` 。例如，以下 YAML 文件适用于作为 Eureka 客户端的配置服务器：

```yaml
eureka:
  instance:
    ...
    metadataMap:
      user: osufhalskjrtl
      password: lviuhlszvaorhvlo5847
      configPath: /config
```

#### 使用 Eureka 和 WebClient 发现第一个 Bootstrap


如果您使用 Spring Cloud Netflix 中的 Eureka `DiscoveryClient` 并且还想使用 `WebClient` 而不是 Jersey 或 `RestTemplate` ，则需要包含 `WebClient` 在你的类路径上并设置 `eureka.client.webclient.enabled=true` 。

### 配置客户端快速失败


在某些情况下，如果某个服务无法连接到配置服务器，您可能希望该服务启动失败。如果这是所需的行为，请设置引导配置属性 `spring.cloud.config.fail-fast=true` 以使客户端因异常而停止。

>  要使用 `spring.config.import` 获得类似的功能，只需省略 `optional:` 前缀即可。

### 配置客户端重试


如果您预计应用程序启动时配置服务器可能偶尔不可用，则可以使其在失败后继续尝试。首先，您需要设置 `spring.cloud.config.fail-fast=true` 。然后您需要将 `spring-retry` 和 `spring-boot-starter-aop` 添加到您的类路径中。默认行为是重试六次，初始退避间隔为 1000 毫秒，后续退避的指数乘数为 1.1。您可以通过设置 `spring.cloud.config.retry.*` 配置属性来配置这些属性（以及其他属性）。

> 要完全控制重试行为并使用旧引导程序，请添加 ID 为 `configServerRetryInterceptor` 的 `RetryOperationsInterceptor` 类型的 `@Bean` 。 Spring Retry 有一个 `RetryInterceptorBuilder` 支持创建一个。

### 使用 spring.config.import 配置客户端重试


Retry 适用于 Spring Boot `spring.config.import` 语句，并且正常属性适用。但是，如果导入语句位于配置文件中，例如 `application-prod.properties` ，那么您需要采用不同的方式来配置重试。配置需要作为 url 参数放置在 import 语句中。

application-prod.properties

```properties
spring.config.import=configserver:http://configserver.example.com?fail-fast=true&max-attempts=10&max-interval=1500&multiplier=1.2&initial-interval=1100"
```


这会设置 `spring.cloud.config.fail-fast=true` （注意上面缺少的前缀）和所有可用的 `spring.cloud.config.retry.*` 配置属性。

### 查找远程配置资源


配置服务从 `/{application}/{profile}/{label}` 提供属性源，其中客户端应用程序中的默认绑定如下：

- "application" = `${spring.application.name}`
- "profile" = `${spring.profiles.active}` (actually `Environment.getActiveProfiles()`)
- "label" = "master" 

>  设置属性 `${spring.application.name}` 时，请勿使用保留字 `application-` 作为应用程序名称的前缀，以防止解析正确的属性源时出现问题。


您可以通过设置 `spring.cloud.config.*` 覆盖所有它们（其中 `*` 是 `name` 、 `profile` 或 `label` ）。 `label` 对于回滚到以前版本的配置很有用。使用默认的 Config Server 实现，它可以是 git 标签、分支名称或提交 ID。标签也可以作为逗号分隔的列表提供。在这种情况下，将一项一项地尝试列表中的项目，直到其中一项成功为止。在处理功能分支时，此行为非常有用。例如，您可能希望将配置标签与您的分支对齐，但将其设为可选（在这种情况下，请使用 `spring.cloud.config.label=myfeature,develop` ）。

### 为配置服务器指定多个 URL


当您部署了多个 Config Server 实例并预计一个或多个实例不可用或无法满足请求（例如 Git 服务器关闭）时，为了确保高可用性，您可以指定多个 URL（例如 `spring.cloud.config.uri` 属性下的逗号分隔列表），或者让所有实例在 Eureka 等服务注册表中注册（如果使用 Discovery-First Bootstrap 模式）。


`spring.cloud.config.uri` 下列出的 URL 将按照列出的顺序进行尝试。默认情况下，配置客户端将尝试从每个 URL 获取属性，直到尝试成功以确保高可用性。


但是，如果您只想在 Config Server 未运行时（即应用程序退出时）或发生连接超时时确保高可用性，请将 `spring.cloud.config.multiple-uri-strategy` 设置为 `connection-timeout-only` . （ `spring.cloud.config.multiple-uri-strategy` 的默认值为 `always` 。）例如，如果配置服务器返回 500（内部服务器错误）响应，或者配置客户端从配置服务器接收到 401（由于凭据错误或其他原因），配置客户端不会尝试从其他 URL 获取属性。 400 错误（可能 404 除外）表示用户问题而不是可用性问题。需要注意的是，如果Config Server设置为使用Git服务器，并且调用Git服务器失败，可能会出现404错误。


可以在单个 `spring.config.import` 键而不是 `spring.cloud.config.uri` 下指定多个位置。位置将按照定义的顺序进行处理，稍后导入的优先。但是，如果 `spring.cloud.config.fail-fast` 为 `true` ，则如果第一个配置服务器调用因任何原因不成功，配置客户端将会失败。如果 `fail-fast` 是 `false` ，它将尝试所有 URL，直到一次调用成功，无论失败的原因是什么。 （在 `spring.config.import` 下指定 URL 时， `spring.cloud.config.multiple-uri-strategy` 不适用。）


如果您在配置服务器上使用 HTTP 基本安全性，则当前仅当您将凭据嵌入在 `spring.cloud.config.uri` 属性下指定的每个 URL 中时，才可以支持每个配置服务器身份验证凭据。如果您使用任何其他类型的安全机制，则（当前）无法支持每个配置服务器的身份验证和授权。

### 配置超时


如果要配置超时阈值：

- 可以使用属性 `spring.cloud.config.request-read-timeout` 配置读取超时。
- 可以使用属性 `spring.cloud.config.request-connect-timeout` 配置连接超时。

### 安全


如果您在服务器上使用 HTTP Basic 安全性，则客户端需要知道密码（如果不是默认值，还需要知道用户名）。您可以通过配置服务器 URI 或通过单独的用户名和密码属性指定用户名和密码，如以下示例所示：

```yaml
spring:
  cloud:
    config:
     uri: https://user:secret@myconfig.mycompany.com
```


以下示例显示了传递相同信息的另一种方法：

```yaml
spring:
  cloud:
    config:
     uri: https://myconfig.mycompany.com
     username: user
     password: secret
```


`spring.cloud.config.password` 和 `spring.cloud.config.username` 值会覆盖 URI 中提供的任何内容。


如果您在 Cloud Foundry 上部署应用程序，提供密码的最佳方法是通过服务凭据（例如在 URI 中，因为它不需要位于配置文件中）。以下示例在本地运行，适用于 Cloud Foundry 上名为 `configserver` 的用户提供的服务：

```yaml
spring:
  cloud:
    config:
     uri: ${vcap.services.configserver.credentials.uri:http://user:password@localhost:8888}
```


如果配置服务器需要客户端 TLS 证书，您可以通过属性配置客户端 TLS 证书和信任存储，如下例所示：

```yaml
spring:
  cloud:
    config:
      uri: https://myconfig.myconfig.com
      tls:
        enabled: true
        key-store: <path-of-key-store>
        key-store-type: PKCS12
        key-store-password: <key-store-password>
        key-password: <key-password>
        trust-store: <path-of-trust-store>
        trust-store-type: PKCS12
        trust-store-password: <trust-store-password>
```


`spring.cloud.config.tls.enabled` 需要为 true 才能启用配置客户端 TLS。当省略 `spring.cloud.config.tls.trust-store` 时，将使用 JVM 默认信任存储。 `spring.cloud.config.tls.key-store-type` 和 `spring.cloud.config.tls.trust-store-type` 的默认值为 PKCS12。当省略密码属性时，假定密码为空。


如果您使用另一种形式的安全性，则可能需要向 `ConfigServicePropertySourceLocator` 提供 `RestTemplate` （例如，通过在引导上下文中获取它并注入它）。

#### 健康指标


配置客户端提供了一个 Spring Boot 运行状况指示器，尝试从配置服务器加载配置。可以通过设置 `health.config.enabled=false` 禁用健康指示器。出于性能原因，响应也会被缓存。默认缓存生存时间为 5 分钟。要更改该值，请设置 `health.config.time-to-live` 属性（以毫秒为单位）。

#### 提供自定义 RestTemplate


在某些情况下，您可能需要自定义从客户端向配置服务器发出的请求。通常，这样做需要传递特殊的 `Authorization` 标头来验证对服务器的请求。

##### 使用配置数据提供自定义 RestTemplate


使用配置数据时提供自定义 `RestTemplate` ：

1. 创建一个实现 `BootstrapRegistryInitializer` 的类

   CustomBootstrapRegistryInitializer.java

   ```java
   public class CustomBootstrapRegistryInitializer implements BootstrapRegistryInitializer {
   
   	@Override
   	public void initialize(BootstrapRegistry registry) {
   		registry.register(RestTemplate.class, context -> {
   			RestTemplate restTemplate = new RestTemplate();
   			// Customize RestTemplate here
   			return restTemplate;
   		});
   	}
   
   }
   ```

2. 在 `resources/META-INF` 中，创建一个名为 `spring.factories` 的文件并指定您的自定义配置，如以下示例所示：

   spring.factories

   ```properties
   org.springframework.boot.BootstrapRegistryInitializer=com.my.config.client.CustomBootstrapRegistryInitializer
   ```

##### 使用 Bootstrap 提供自定义 RestTemplate


使用 Bootstrap 时提供自定义 `RestTemplate` ：

1. 创建一个具有 `PropertySourceLocator` 实现的新配置 bean，如以下示例所示：

   CustomConfigServiceBootstrapConfiguration.java

   ```java
   @Configuration
   public class CustomConfigServiceBootstrapConfiguration {
       @Bean
       public ConfigServicePropertySourceLocator configServicePropertySourceLocator() {
           ConfigClientProperties clientProperties = configClientProperties();
          ConfigServicePropertySourceLocator configServicePropertySourceLocator =  new ConfigServicePropertySourceLocator(clientProperties);
           configServicePropertySourceLocator.setRestTemplate(customRestTemplate(clientProperties));
           return configServicePropertySourceLocator;
       }
   }
   ```

   > 对于添加 `Authorization` 标头的简化方法，可以使用 `spring.cloud.config.headers.*` 属性。

2. 在 `resources/META-INF` 中，创建一个名为 `spring.factories` 的文件并指定您的自定义配置，如以下示例所示：

   spring.factories

   ```properties
   org.springframework.cloud.bootstrap.BootstrapConfiguration = com.my.config.client.CustomConfigServiceBootstrapConfiguration
   ```

#### Vault


当使用 Vault 作为配置服务器的后端时，客户端需要为服务器提供令牌以从 Vault 检索值。可以通过在 `bootstrap.yml` 中设置 `spring.cloud.config.token` 在客户端内提供此令牌，如以下示例所示：

```yaml
spring:
  cloud:
    config:
      token: YourVaultToken
```

### Vault 中的嵌套键


Vault 支持在 Vault 中存储的值中嵌套键的功能，如以下示例所示：

```
echo -n '{"appA": {"secret": "appAsecret"}, "bar": "baz"}' | vault write secret/myapp -
```


此命令将 JSON 对象写入您的 Vault。要在 Spring 中访问这些值，您可以使用传统的点 ( `.` ) 注释，如以下示例所示

```java
@Value("${appA.secret}")
String name = "World";
```


前面的代码会将 `name` 变量的值设置为 `appAsecret` 。

### AOT 和本机映像支持


从 `4.0.0` 开始，Spring Cloud Config Client 支持 Spring AOT 转换和 GraalVM 本机映像。

AOT 和本机映像支持不适用于配置首次引导（使用 `spring.config.use-legacy-processing=true` ）。

本机映像不支持刷新范围。如果您要将配置客户端应用程序作为本机映像运行，请确保将 `spring.cloud.refresh.enabled` 属性设置为 `false` 。

 在构建包含 Spring Cloud Config Client 的项目时，必须确保其连接的配置数据源（例如 Spring Cloud Config Server、Consul、Zookeeper、Vault 等）可用。例如，如果您从 Spring Cloud Config Server 检索配置数据，请确保其实例正在运行并且在 Config Client 设置中指示的端口可用。 This is necessary because the application context is being optimized at build time and requires the target environment to be resolved. 这是必要的，因为应用程序上下文正在构建时进行优化，并且需要解析目标环境。

由于在 AOT 和本机模式中，正在处理配置并在构建时优化上下文，因此任何会影响 Bean 创建的属性（例如引导上下文中使用的属性）都应在构建时和运行时设置为相同的值以避免意外行为。

由于 Config Client 在从本机映像启动时连接到正在运行的数据源（例如 Config Server），因此快速启动时间将因网络通信所需的时间而变慢。

# 附录

## 可观测性元数据

### 可观察性 - 指标


您可以在下面找到该项目声明的所有指标的列表。

#### 环境存储库

>
> 围绕环境存储库创建的观察。


指标名称 `spring.cloud.config.environment.find` （由约定类 `org.springframework.cloud.config.server.environment.ObservationEnvironmentRepositoryObservationConvention` 定义）。输入 `timer` 。


指标名称 `spring.cloud.config.environment.find.active` （由约定类 `org.springframework.cloud.config.server.environment.ObservationEnvironmentRepositoryObservationConvention` 定义）。输入 `long task timer` 。

> *.active 指标中可能会缺少启动观察后添加的键值。
>
> Micrometer 内部使用 `nanoseconds` 作为基本单位。然而，每个后端决定实际的基本单元。 （即普罗米修斯使用秒）


封闭类的完全限定名称 `org.springframework.cloud.config.server.environment.DocumentedConfigObservation` 。

> 所有标签必须以 `spring.cloud.config.environment` 前缀开头！

| Name                                                       | Description                    |
| ---------------------------------------------------------- | ------------------------------ |
| `spring.cloud.config.environment.application` *(required)* | 正在查询其属性的应用程序名称。 |
| `spring.cloud.config.environment.class` *(required)*       | 环境存储库的实施。             |
| `spring.cloud.config.environment.label` *(required)*       | 正在查询属性的标签。           |
| `spring.cloud.config.environment.profile` *(required)*     | 正在查询其属性的应用程序名称。 |

### 可观察性 - 跨度


您可以在下面找到该项目声明的所有跨度的列表。

#### 环境存储库跨度

>
> 围绕环境存储库创建的观察。


跨度名称 `spring.cloud.config.environment.find` （由约定类 `org.springframework.cloud.config.server.environment.ObservationEnvironmentRepositoryObservationConvention` 定义）。


封闭类的完全限定名称 `org.springframework.cloud.config.server.environment.DocumentedConfigObservation` 。

> 所有标签必须以 `spring.cloud.config.environment` 前缀开头！

| Name                                                       | Description                    |
| ---------------------------------------------------------- | ------------------------------ |
| `spring.cloud.config.environment.application` *(required)* | 正在查询其属性的应用程序名称。 |
| `spring.cloud.config.environment.class` *(required)*       | 环境存储库的实施。             |
| `spring.cloud.config.environment.label` *(required)*       | 正在查询属性的标签。           |
| `spring.cloud.config.environment.profile` *(required)*     | 正在查询其属性的应用程序名称。 |