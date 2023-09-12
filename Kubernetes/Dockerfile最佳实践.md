本附录是笔者对 Docker 官方文档中 [Best practices for writing Dockerfiles](https://docs.docker.com/engine/userguide/eng-image/dockerfile_best-practices/) 的理解与翻译。 

# 一般性的指南和建议



## 减少构建时间



### 构建顺序影响缓存的利用率

![](../assets/1ed37d912a1bae4749007e0abb655698/008i3skNly1gtm2mxi7dhj60m809tt9a02.jpg)


镜像的构建顺序很重要，当你向 Dockerfile 中添加文件，或者修改其中的某一行时，那一部分的缓存就会失效，该缓存的后续步骤都会中断，需要重新构建。**所以优化缓存的最佳方法是把不需要经常更改的行放到最前面，更改最频繁的行放到最后面。** 

### 最小化可缓存的执行层

![](../assets/1ed37d912a1bae4749007e0abb655698/008i3skNly1gtm2n07xesj60m00acaas02.jpg)


每一个 RUN 指令都会被看作是可缓存的执行单元。太多的 RUN 指令会增加镜像的层数，增大镜像体积，而将所有的命令都放到同一个 RUN 指令中又会破坏缓存，从而延缓开发周期。当使用包管理器安装软件时，一般都会先更新软件索引信息，然后再安装软件。推荐将更新索引和安装软件放在同一个 RUN 指令中，这样可以形成一个可缓存的执行单元，否则你可能会安装旧的软件包。 

### 构建缓存

在镜像的构建过程中，Docker 会遍历 `Dockerfile` 文件中的指令，然后按顺序执行。在执行每条指令之前，Docker 都会在缓存中查找是否已经存在可重用的镜像，如果有就使用现存的镜像，不再重复创建。如果你不想在构建过程中使用缓存，你可以在 `docker build` 命令中使用 `--no-cache=true` 选项。


但是，如果你想在构建的过程中使用缓存，你得明白什么时候会，什么时候不会找到匹配的镜像，遵循的基本规则如下：

- 从一个基础镜像开始（`FROM` 指令指定），下一条指令将和该基础镜像的所有子镜像进行匹配，检查这些子镜像被创建时使用的指令是否和被检查的指令完全一样。如果不是，则缓存失效。



- 在大多数情况下，只需要简单地对比 `Dockerfile` 中的指令和子镜像。然而，有些指令需要更多的检查和解释。



- 对于 `ADD` 和 `COPY` 指令，镜像中对应文件的内容也会被检查，每个文件都会计算出一个校验和。文件的最后修改时间和最后访问时间不会纳入校验。在缓存的查找过程中，会将这些校验和和已存在镜像中的文件校验和进行对比。如果文件有任何改变，比如内容和元数据，则缓存失效。



- 除了 `ADD` 和 `COPY` 指令，缓存匹配过程不会查看临时容器中的文件来决定缓存是否匹配。例如，当执行完 `RUN apt-get -y update` 指令后，容器中一些文件被更新，但 Docker 不会检查这些文件。这种情况下，只有指令字符串本身被用来匹配缓存。




一旦缓存失效，所有后续的 `Dockerfile` 指令都将产生新的镜像，缓存不会被使用。 

## 减小镜像体积



### 只拷贝需要的文件，防止缓存溢出

![](../assets/1ed37d912a1bae4749007e0abb655698/008i3skNly1gtm2n1oqanj60m40a6aaq02.jpg)


当拷贝文件到镜像中时，尽量只拷贝需要的文件，切忌使用 COPY . 指令拷贝整个目录。如果被拷贝的文件内容发生了更改，缓存就会被破坏。在上面的示例中，镜像中只需要构建好的 jar 包，因此只需要拷贝这个文件就行了，这样即使其他不相关的文件发生了更改也不会影响缓存。 

### 使用 `.dockerignore` 文件

使用 `Dockerfile` 构建镜像时最好是将 `Dockerfile` 放置在一个新建的空目录下。然后将构建镜像所需要的文件添加到该目录中。为了提高构建镜像的效率，你可以在目录下新建一个 `.dockerignore` 文件来指定要忽略的文件和目录。`.dockerignore` 文件的排除模式语法和 Git 的 `.gitignore` 文件相似。 

### 删除不必要依赖

![](../assets/1ed37d912a1bae4749007e0abb655698/008i3skNly1gtm2mwvidbj60m407mgm002.jpg)


删除不必要的依赖，不要安装调试工具。如果实在需要调试工具，可以在容器运行之后再安装。某些包管理工具（如 apt）除了安装用户指定的包之外，还会安装推荐的包，这会无缘无故增加镜像的体积。apt 可以通过添加参数 -–no-install-recommends 来确保不会安装不需要的依赖项。如果确实需要某些依赖项，请在后面手动添加。 

### 删除包管理工具的缓存

![](../assets/1ed37d912a1bae4749007e0abb655698/008i3skNly1gtm2mzauimj60m40a4gm302.jpg)


包管理工具会维护自己的缓存，这些缓存会保留在镜像文件中，推荐的处理方法是在每一个 RUN 指令的末尾删除缓存。如果你在下一条指令中删除缓存，不会减小镜像的体积。 

### 一个容器只运行一个进程

应该保证在一个容器中只运行一个进程。将多个应用解耦到不同容器中，保证了容器的横向扩展和复用。例如 web 应用应该包含三个容器：web应用、数据库、缓存。


如果容器互相依赖，你可以使用 [Docker 自定义网络]() 来把这些容器连接起来。 

## 可维护性



### 尽量使用官方镜像

![](../assets/1ed37d912a1bae4749007e0abb655698/008i3skNly1gtm2mybc1gj60m407zmxq02.jpg)


使用官方镜像可以节省大量的维护时间，因为官方镜像的所有安装步骤都使用了最佳实践。如果你有多个项目，可以共享这些镜像层，因为他们都可以使用相同的基础镜像。 

### 使用更具体的标签

![](../assets/1ed37d912a1bae4749007e0abb655698/008i3skNly1gtm2n0p12lj60m809wwey02.jpg)


基础镜像尽量不要使用 latest 标签。虽然这很方便，但随着时间的推移，latest 镜像可能会发生重大变化。因此在 Dockerfile 中最好指定基础镜像的具体标签。我们使用 openjdk 作为示例，指定标签为 8。其他更多标签请查看官方仓库。 

### 使用体积最小的基础镜像

![](../assets/1ed37d912a1bae4749007e0abb655698/008i3skNly1gtm2mwgn1oj60m409s0t402.jpg)


基础镜像的标签风格不同，镜像体积就会不同。slim 风格的镜像是基于 Debian 发行版制作的，而 alpine 风格的镜像是基于体积更小的 Alpine Linux 发行版制作的。其中一个明显的区别是：Debian 使用的是 GNU 项目所实现的 C 语言标准库，而 Alpine 使用的是 Musl C 标准库，它被设计用来替代 GNU C 标准库（glibc）的替代品，用于嵌入式操作系统和移动设备。因此使用 Alpine 在某些情况下会遇到兼容性问题。 以 openjdk 为例，jre 风格的镜像只包含 Java 运行时，不包含 SDK，这么做也可以大大减少镜像体积。 

## 重复利用



### 在一致的环境中从源代码构建

源代码是你构建 Docker 镜像的最终来源，Dockerfile 里面只提供了构建步骤。


![](../assets/1ed37d912a1bae4749007e0abb655698/008i3skNly1gtm2n15wfzj60m409n3z102.jpg)


首先应该确定构建应用所需的所有依赖，本文的示例 Java 应用很简单，只需要 Maven 和 JDK，所以基础镜像应该选择官方的体积最小的 maven 镜像，该镜像也包含了 JDK。如果你需要安装更多依赖，可以在 RUN 指令中添加。pom.xml文件和 src 文件夹需要被复制到镜像中，因为最后执行 mvn package 命令（-e 参数用来显示错误，-B 参数表示以非交互式的“批处理”模式运行）打包的时候会用到这些依赖文件。


虽然现在我们解决了环境不一致的问题，但还有另外一个问题：**每次代码更改之后，都要重新获取一遍 pom.xml 中描述的所有依赖项。**下面我们来解决这个问题。 

### 在单独的步骤中获取依赖项

![](../assets/1ed37d912a1bae4749007e0abb655698/008i3skNly1gtm2mxu31aj60m4073t9402.jpg)


结合前面提到的缓存机制，我们可以让获取依赖项这一步变成可缓存单元，只要 pom.xml 文件的内容没有变化，无论代码如何更改，都不会破坏这一层的缓存。上图中两个 COPY 指令中间的 RUN 指令用来告诉 Maven 只获取依赖项。


现在又遇到了一个新问题：跟之前直接拷贝 jar 包相比，镜像体积变得更大了，因为它包含了很多运行应用时不需要的构建依赖项。 

### 使用多阶段构建来删除构建时的依赖项

![](../assets/1ed37d912a1bae4749007e0abb655698/008i3skNly1gtm2n23b0rj60m409u0tj02.jpg)


多阶段构建可以由多个 FROM 指令识别，每一个 FROM 语句表示一个新的构建阶段，阶段名称可以用 AS 参数指定。本例中指定第一阶段的名称为 builder，它可以被第二阶段直接引用。两个阶段环境一致，并且第一阶段包含所有构建依赖项。


第二阶段是构建最终镜像的最后阶段，它将包括应用运行时的所有必要条件，本例是基于 Alpine 的最小 JRE 镜像。上一个构建阶段虽然会有大量的缓存，但不会出现在第二阶段中。为了将构建好的 jar 包添加到最终的镜像中，可以使用 COPY --from=STAGE\_NAME 指令，其中 STAGE\_NAME 是上一构建阶段的名称。


![](../assets/1ed37d912a1bae4749007e0abb655698/008i3skNly1gtm2mw3ptvj60m808yt9c02.jpg)


多阶段构建是删除构建依赖的首选方案。 

# Dockerfile 指令

下面针对 `Dockerfile` 中各种指令的最佳编写方式给出建议。 

### FROM

尽可能使用当前官方仓库作为你构建镜像的基础。推荐使用 [Alpine](https://hub.docker.com/_/alpine/) 镜像，因为它被严格控制并保持最小尺寸（目前小于 5 MB），但它仍然是一个完整的发行版。 

### LABEL

你可以给镜像添加标签来帮助组织镜像、记录许可信息、辅助自动化构建等。每个标签一行，由 `LABEL` 开头加上一个或多个标签对。下面的示例展示了各种不同的可能格式。`#` 开头的行是注释内容。


注意：如果你的字符串中包含空格，必须将字符串放入引号中或者对空格使用转义。如果字符串内容本身就包含引号，必须对引号使用转义。

    # Set one or more individual labels
    LABEL com.example.version="0.0.1-beta"
    LABEL vendor="ACME Incorporated"
    LABEL com.example.release-date="2015-02-12"
    LABEL com.example.version.is-production=""

一个镜像可以包含多个标签，但建议将多个标签放入到一个 `LABEL` 指令中。

    # Set multiple labels at once, using line-continuation characters to break long lines
    LABEL vendor=ACME\ Incorporated \
          com.example.is-beta= \
          com.example.is-production="" \
          com.example.version="0.0.1-beta" \
          com.example.release-date="2015-02-12"

关于标签可以接受的键值对，参考 [Understanding object labels](https://docs.docker.com/engine/userguide/labels-custom-metadata/)。关于查询标签信息，参考 [Managing labels on objects](https://docs.docker.com/engine/userguide/labels-custom-metadata/#managing-labels-on-objects)。 

### RUN

为了保持 `Dockerfile` 文件的可读性，可理解性，以及可维护性，建议将长的或复杂的 `RUN` 指令用反斜杠 `\` 分割成多行。 

#### apt-get

`RUN` 指令最常见的用法是安装包用的 `apt-get`。因为 `RUN apt-get` 指令会安装包，所以有几个问题需要注意。


不要使用 `RUN apt-get upgrade` 或 `dist-upgrade`，因为许多基础镜像中的「必须」包不会在一个非特权容器中升级。如果基础镜像中的某个包过时了，你应该联系它的维护者。如果你确定某个特定的包，比如 `foo`，需要升级，使用 `apt-get install -y foo` 就行，该指令会自动升级 `foo` 包。


永远将 `RUN apt-get update` 和 `apt-get install` 组合成一条 `RUN` 声明，例如：

    RUN apt-get update && apt-get install -y \
            package-bar \
            package-baz \
            package-foo

将 `apt-get update` 放在一条单独的 `RUN` 声明中会导致缓存问题以及后续的 `apt-get install` 失败。比如，假设你有一个 `Dockerfile` 文件：

    FROM ubuntu:14.04
    RUN apt-get update
    RUN apt-get install -y curl

构建镜像后，所有的层都在 Docker 的缓存中。假设你后来又修改了其中的 `apt-get install` 添加了一个包：

    FROM ubuntu:14.04
    RUN apt-get update
    RUN apt-get install -y curl nginx

Docker 发现修改后的 `RUN apt-get update` 指令和之前的完全一样。所以，`apt-get update` 不会执行，而是使用之前的缓存镜像。因为 `apt-get update` 没有运行，后面的 `apt-get install` 可能安装的是过时的 `curl` 和 `nginx` 版本。


使用 `RUN apt-get update && apt-get install -y` 可以确保你的 Dockerfiles 每次安装的都是包的最新的版本，而且这个过程不需要进一步的编码或额外干预。这项技术叫作 `cache busting`。你也可以显示指定一个包的版本号来达到 `cache-busting`，这就是所谓的固定版本，例如：

    RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo=1.3.*

固定版本会迫使构建过程检索特定的版本，而不管缓存中有什么。这项技术也可以减少因所需包中未预料到的变化而导致的失败。


下面是一个 `RUN` 指令的示例模板，展示了所有关于 `apt-get` 的建议。

    RUN apt-get update && apt-get install -y \
        aufs-tools \
        automake \
        build-essential \
        curl \
        dpkg-sig \
        libcap-dev \
        libsqlite3-dev \
        mercurial \
        reprepro \
        ruby1.9.1 \
        ruby1.9.1-dev \
        s3cmd=1.1.* \
     && rm -rf /var/lib/apt/lists/*

其中 `s3cmd` 指令指定了一个版本号 `1.1.*`。如果之前的镜像使用的是更旧的版本，指定新的版本会导致 `apt-get udpate` 缓存失效并确保安装的是新版本。


另外，清理掉 apt 缓存 `var/lib/apt/lists` 可以减小镜像大小。因为 `RUN` 指令的开头为 `apt-get udpate`，包缓存总是会在 `apt-get install` 之前刷新。


注意：官方的 Debian 和 Ubuntu 镜像会自动运行 apt-get clean，所以不需要显式的调用 apt-get clean。 

### CMD

`CMD` 指令用于执行目标镜像中包含的软件，可以包含参数。`CMD` 大多数情况下都应该以 `CMD ["executable", "param1", "param2"...]` 的形式使用。因此，如果创建镜像的目的是为了部署某个服务(比如 `Apache`)，你可能会执行类似于 `CMD ["apache2", "-DFOREGROUND"]` 形式的命令。我们建议任何服务镜像都使用这种形式的命令。


多数情况下，`CMD` 都需要一个交互式的 `shell` (bash, Python, perl 等)，例如 `CMD ["perl", "-de0"]`，或者 `CMD ["PHP", "-a"]`。使用这种形式意味着，当你执行类似 `docker run -it python` 时，你会进入一个准备好的 `shell` 中。`CMD` 应该在极少的情况下才能以 `CMD ["param", "param"]` 的形式与 `ENTRYPOINT` 协同使用，除非你和你的镜像使用者都对 `ENTRYPOINT` 的工作方式十分熟悉。 

### EXPOSE

`EXPOSE` 指令用于指定容器将要监听的端口。因此，你应该为你的应用程序使用常见的端口。例如，提供 `Apache` web 服务的镜像应该使用 `EXPOSE 80`，而提供 `MongoDB` 服务的镜像使用 `EXPOSE 27017`。


对于外部访问，用户可以在执行 `docker run` 时使用一个标志来指示如何将指定的端口映射到所选择的端口。 

### ENV

为了方便新程序运行，你可以使用 `ENV` 来为容器中安装的程序更新 `PATH` 环境变量。例如使用 `ENV PATH /usr/local/nginx/bin:$PATH` 来确保 `CMD ["nginx"]` 能正确运行。


`ENV` 指令也可用于为你想要容器化的服务提供必要的环境变量，比如 Postgres 需要的 `PGDATA`。


最后，`ENV` 也能用于设置常见的版本号，比如下面的示例：

    ENV PG_MAJOR 9.3
    ENV PG_VERSION 9.3.4
    RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
    ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH

类似于程序中的常量，这种方法可以让你只需改变 `ENV` 指令来自动的改变容器中的软件版本。 

### ADD 和 COPY

虽然 `ADD` 和 `COPY` 功能类似，但一般优先使用 `COPY`。因为它比 `ADD` 更透明。`COPY` 只支持简单将本地文件拷贝到容器中，而 `ADD` 有一些并不明显的功能（比如本地 tar 提取和远程 URL 支持）。因此，`ADD` 的最佳用例是将本地 tar 文件自动提取到镜像中，例如 `ADD rootfs.tar.xz`。


如果你的 `Dockerfile` 有多个步骤需要使用上下文中不同的文件。单独 `COPY` 每个文件，而不是一次性的 `COPY` 所有文件，这将保证每个步骤的构建缓存只在特定的文件变化时失效。例如：

    COPY requirements.txt /tmp/
    RUN pip install --requirement /tmp/requirements.txt
    COPY . /tmp/

如果将 `COPY . /tmp/` 放置在 `RUN` 指令之前，只要 `.` 目录中任何一个文件变化，都会导致后续指令的缓存失效。


为了让镜像尽量小，最好不要使用 `ADD` 指令从远程 URL 获取包，而是使用 `curl` 和 `wget`。这样你可以在文件提取完之后删掉不再需要的文件来避免在镜像中额外添加一层。比如尽量避免下面的用法：

    ADD http://example.com/big.tar.xz /usr/src/things/
    RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
    RUN make -C /usr/src/things all

而是应该使用下面这种方法：

    RUN mkdir -p /usr/src/things \
        && curl -SL http://example.com/big.tar.xz \
        | tar -xJC /usr/src/things \
        && make -C /usr/src/things all

上面使用的管道操作，所以没有中间文件需要删除。


对于其他不需要 `ADD` 的自动提取功能的文件或目录，你应该使用 `COPY`。 

### ENTRYPOINT

`ENTRYPOINT` 的最佳用处是设置镜像的主命令，允许将镜像当成命令本身来运行（用 `CMD` 提供默认选项）。


例如，下面的示例镜像提供了命令行工具 `s3cmd`:

    ENTRYPOINT ["s3cmd"]
    CMD ["--help"]

现在直接运行该镜像创建的容器会显示命令帮助：

    $ docker run s3cmd

或者提供正确的参数来执行某个命令：

    $ docker run s3cmd ls s3://mybucket

这样镜像名可以当成命令行的参考。


`ENTRYPOINT` 指令也可以结合一个辅助脚本使用，和前面命令行风格类似，即使启动工具需要不止一个步骤。


例如，`Postgres` 官方镜像使用下面的脚本作为 `ENTRYPOINT`：

    #!/bin/bash
    set -e
    if [ "$1" = 'postgres' ]; then
        chown -R postgres "$PGDATA"
        if [ -z "$(ls -A "$PGDATA")" ]; then
            gosu postgres initdb
        fi
        exec gosu postgres "$@"
    fi
    exec "$@"

注意：该脚本使用了 Bash 的内置命令 exec，所以最后运行的进程就是容器的 PID 为 1 的进程。这样，进程就可以接收到任何发送给容器的 Unix 信号了。


该辅助脚本被拷贝到容器，并在容器启动时通过 `ENTRYPOINT` 执行：

    COPY ./docker-entrypoint.sh /
    ENTRYPOINT ["/docker-entrypoint.sh"]

该脚本可以让用户用几种不同的方式和 `Postgres` 交互。


你可以很简单地启动 `Postgres`：

    $ docker run postgres

也可以执行 `Postgres` 并传递参数：

    $ docker run postgres postgres --help

最后，你还可以启动另外一个完全不同的工具，比如 `Bash`：

    $ docker run --rm -it postgres bash



### VOLUME

`VOLUME` 指令用于暴露任何数据库存储文件，配置文件，或容器创建的文件和目录。强烈建议使用 `VOLUME` 来管理镜像中的可变部分和用户可以改变的部分。 

### USER

如果某个服务不需要特权执行，建议使用 `USER` 指令切换到非 root 用户。先在 `Dockerfile` 中使用类似 `RUN groupadd -r postgres && useradd -r -g postgres postgres` 的指令创建用户和用户组。


注意：在镜像中，用户和用户组每次被分配的 UID/GID 都是不确定的，下次重新构建镜像时被分配到的 UID/GID 可能会不一样。如果要依赖确定的 UID/GID，你应该显示的指定一个 UID/GID。


你应该避免使用 `sudo`，因为它不可预期的 TTY 和信号转发行为可能造成的问题比它能解决的问题还多。如果你真的需要和 `sudo` 类似的功能（例如，以 root 权限初始化某个守护进程，以非 root 权限执行它），你可以使用 [gosu](https://github.com/tianon/gosu)。


最后，为了减少层数和复杂度，避免频繁地使用 `USER` 来回切换用户。 

### WORKDIR

为了清晰性和可靠性，你应该总是在 `WORKDIR` 中使用绝对路径。另外，你应该使用 `WORKDIR` 来替代类似于 `RUN cd ... && do-something` 的指令，后者难以阅读、排错和维护。 

## 官方仓库示例

这些官方仓库的 Dockerfile 都是参考典范：<https://github.com/docker-library/docs>
