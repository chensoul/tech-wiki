

## 介绍

Azkaban 是由 Linkedin 开源的一个批量工作流任务调度器。用于在一个工作流内以一个特定的顺序运行一组工作和流程。Azkaban 定义了一种 KV 文件格式来建立任务之间的依赖关系，并提供一个易于使用的 web 用户界面维护和跟踪你的工作流。

- Azkaban 官网地址：<http://azkaban.github.io/>
- Azkaban 的下载地址：http://azkaban.github.io/downloads.html

Azkaban 包括三个关键组件：

- `关系数据库`：使用 Mysql数据库，主要用于保存流程、权限、任务状态、任务计划等信息。
- `AzkabanWebServer`：为用户提供管理留存、任务计划、权限等功能。
- `AzkabanExecutorServer`：执行任务，并把任务执行的输出日志保存到 Mysql；可以同时启动多个 AzkabanExecutorServer ，他们通过 mysql 获取流程状态来协调工作。



## 二进制文件安装



### 编译源码

在 2.5 版本之后，Azkaban 提供了两种模式来安装：

- 一种是 standalone 的 “solo-server” 模式；
- 另一种是两个 server 的模式，分别为 AzkabanWebServer 和 AzkabanExecutorServer。

这里主要介绍第二种模式的安装方法。

    wget https://github.com/azkaban/azkaban/archive/3.84.8.zip
    unzip 3.84.8.zip
    cd azkaban-3.84.8
    ./gradlew distTar

编译后的文件：

    azkaban-exec-server/build/distributions/azkaban-exec-server-0.1.0-SNAPSHOT.zip
    azkaban-web-server/build/distributions/azkaban-web-server-0.1.0-SNAPSHOT.zip



### 安装 MySql

目前 Azkaban 只支持 MySql ，故需安装 MySql 服务器，安装 MySql 的过程这里不作介绍。


安装之后，创建 azkaban 数据库，并创建 azkaban 用户，密码为 azkaban，并设置权限。

    mysql> CREATE DATABASE azkaban;
    mysql> CREATE USER 'azkaban'@'%' IDENTIFIED BY 'azkaban';
    mysql> GRANT all ON azkaban.* to 'azkaban'@'%' WITH GRANT OPTION;
    mysql> flush privileges;

修改 `/etc/my.cnf` 文件，设置 `max_allowed_packet` 值：

    [mysqld]
    ...
    max_allowed_packet=1024M

然后重启 MySql。 

### 安装 azkaban-web-server

解压缩 azkaban-web-server-0.1.0-SNAPSHOT.zip。

    unzip azkaban-web-server-0.1.0-SNAPSHOT.zip
    mv azkaban-web-server-0.1.0-SNAPSHOT /opt/azkaban-web-server

创建 SSL 配置，命令：

    keytool -keystore keystore -alias jetty -genkey -keyalg RSA

密码均输入azkaban，最后在当前目录生成 keystore 证书文件。将 keystore 考贝到 azkaban web 目录中

    mv keystore /opt/azkaban-web-server

修改 azkaban web 服务器配置conf/azkaban.properties，主要包括：


a. 修改时区和首页名称:

    # Azkaban Personalization Settings
    azkaban.name=ETL Task
    azkaban.label=By BI
    azkaban.color=#FF3601
    azkaban.default.servlet.path=/index
    web.resource.dir=web/
    default.timezone.id=Asia/Shanghai

b. 修改 MySql 数据库配置

    database.type=mysql
    mysql.port=3306
    mysql.host=localhost
    mysql.database=azkaban
    mysql.user=azkaban
    mysql.password=azkaban
    mysql.numconnections=100

c. 修改 Jetty 服务器属性，包括 keystore 的相关配置

    # Azkaban Jetty server properties.
    jetty.hostname=0.0.0.0
    jetty.maxThreads=25
    jetty.port=8081
    jetty.keystore=keystore
    jetty.password=azkaban
    jetty.keypassword=azkaban
    jetty.truststore=keystore
    jetty.trustpassword=azkaban

d. 开启SSL

    jetty.use.ssl=true
    jetty.ssl.port=8443

e.修改邮件设置（可选）

    # mail settings
    mail.sender=admin@javachen.space
    mail.host=javachen.space
    mail.user=admin
    mail.password=admin

f. 解压缩azkaban-db-0.1.0-SNAPSHOT.zip，然后进入 mysql 命令行模式：

    unzip azkaban-db-0.1.0-SNAPSHOT.zip
    $ mysql -uazkaban -pazkaban
    mysql> use azkaban
    mysql> source azkaban-db-0.1.0-SNAPSHOT/create-all-sql-0.1.0-SNAPSHOT.sql

g. 设置Executor：

    #Multiple Executor
    azkaban.use.multiple.executors=true
    #azkaban.executorselector.filters=StaticRemainingFlowSize,MinimumFreeMemory,CpuStatus
    azkaban.executorselector.filters=StaticRemainingFlowSize,CpuStatus
    azkaban.executorselector.comparator.NumberOfAssignedFlowComparator=1
    azkaban.executorselector.comparator.Memory=1
    azkaban.executorselector.comparator.LastDispatched=1
    azkaban.executorselector.comparator.CpuUsage=1
    executor.port=12321



### 安装 azkaban-exec-server

解压缩 azkaban-exec-server-0.1.0-SNAPSHOT.zip

    unzip azkaban-exec-server-0.1.0-SNAPSHOT.zip
    mv azkaban-exec-server-0.1.0-SNAPSHOT /opt/azkaban-exec-server

然后修改配置文件conf/azkaban.properties，包括：


a. 修改时区为：`default.timezone.id=Asia/Shanghai`


b. 修改 MySql 数据库配置

    database.type=mysql
    mysql.port=3306
    mysql.host=localhost
    mysql.database=azkaban
    mysql.user=azkaban
    mysql.password=azkaban
    mysql.numconnections=100

c. 设置executor端口为12321

    executor.maxThreads=50
    executor.port=12321
    executor.flow.threads=30



### 用户设置

进入 azkaban web 服务器 conf 目录，修改 azkaban-users.xml ，增加管理员用户：

    <azkaban-users>
      <user groups="azkaban" password="admin027" roles="admin" username="azkaban"/>
      <user password="metrics" roles="metrics" username="metrics"/>
      <user groups="test" username="test"  password="123456" roles="read"/>
      <role name="admin" permissions="ADMIN"/>
      <role name="metrics" permissions="METRICS"/>
      <role name="read" permissions="READ"/>
      <role name="write" permissions="WRITE"/>
      <role name="execute" permissions="EXECUTE"/>
      <role name="schedule" permissions="SCHEDULE"/>
      <role name="createprojects" permissions="CREATEPROJECTS"/>
    </azkaban-users>



### 启动服务

azkaban-exec-server，需要在 azkaban-exec-server 目录下执行下面命令：

    sh bin/start-exec.sh

激活当前executor：

    $ curl -G "localhost:$(<./executor.port)/executor?action=activate" && echo
    {"status":"success"}

azkaban-web-server，需要在 azkaban-web-server 目录下执行下面命令：

    sh bin/start-web.sh



## Docker安装

关于azkaban的docker镜像，可以参考：

- <https://gitee.com/datatech/docker-azkaban>
- <https://github.com/jfim/data-platform-demo/blob/master/dockerfiles/Dockerfile-azkaban>
- <https://github.com/wilson-lauw/azkaban>

这里，我参考上面文章自己制作了azkaban的相关镜像，地址：[azkaban-web-server](https://github.com/javachen/dockerfiles/azkaban-web-server) 和 [azkaban-exec-server](https://github.com/javachen/dockerfiles/azkaban-exec-server) 

### azkaban-web-server镜像

azkaban-web-server镜像制作过程：


1、首先从源码编译得到打包后的文件 azkaban-web-server-0.1.0-SNAPSHOT.tar.gz

    mkdir azkaban-exec-server azkaban-web-server
    wget https://github.com/azkaban/azkaban/archive/3.84.8.zip
    unzip 3.84.8.zip
    cd azkaban-3.84.8
    ./gradlew distTar
    cp azkaban-exec-server/build/distributions/azkaban-exec-server-0.1.0-SNAPSHOT.tar.gz ../azkaban-exec-server/
    cp azkaban-web-server/build/distributions/azkaban-web-server-0.1.0-SNAPSHOT.tar.gz ../azkaban-web-server/

2、创建Dockerfile：

    FROM javachen/alpine-jdk:8
    # Azkaban web server port
    EXPOSE 8443    
    COPY azkaban-web-server-0.1.0-SNAPSHOT.tar.gz .
    RUN set -eux \
        && tar zxf azkaban-web-server-0.1.0-SNAPSHOT.tar.gz \
        && mv azkaban-web-server-0.1.0-SNAPSHOT azkaban-web-server \
        && rm -rf azkaban-web-server-0.1.0-SNAPSHOT.tar.gz
    # Define default workdir
    WORKDIR azkaban-web-server
    COPY conf/* conf/
    COPY run.sh run.sh
    RUN chmod +x run.sh
    CMD ./run.sh && tail -f /dev/null

3、创建conf目录，存储配置文件：

- azkaban-users.xml
- azkaban.properties
- keystore

4、创建run.sh文件，主要用于设置等待mysql启动之后再启动服务并修改启动服务的命令

    #!/bin/bash
    DB_LOOPS="20"
    MYSQL_HOST="mysql"
    MYSQL_PORT="3306"
    START_CMD="bin/start-web.sh"
    #wait for mysql
    i=0
    while ! nc $MYSQL_HOST $MYSQL_PORT >/dev/null 2>&1 < /dev/null; do
      i=`expr $i + 1`
      if [ $i -ge $DB_LOOPS ]; then
        echo "$(date) - ${MYSQL_HOST}:${MYSQL_PORT} still not reachable, giving up"
        exit 1
      fi
      echo "$(date) - waiting for ${MYSQL_HOST}:${MYSQL_PORT}..."
      sleep 1
    done
    # Work around to run container as a daemon
    sed -i "s/ &//" $START_CMD
    curl -G "executor:12321/executor?action=activate" && echo
    # mysql -h mysql -uazkaban -pazkaban -e "update executors set active=1 where host='executor';"
    # Work around to run container as a daemon
    exec $START_CMD

注意：

- 从run.sh可以看到 azkaban-web-server 依赖 mysql 数据库，其主机名为 mysql ，所以需要部署一个mysql容器，容器名称设置为mysql，当然，你如果想用外置的sql，则可以设置为ip或者域名。

5、编译上传镜像：

    docker build -t javachen/azkaban-web-server .
    docker push javachen/azkaban-web-server



### azkaban-exec-server镜像

同理，azkaban-exec-server镜像制作过程：


1、首先从源码编译得到打包后的文件 azkaban-exec-server-0.1.0-SNAPSHOT.tar.gz 以及sql文件 create-all-sql-0.1.0-SNAPSHOT.sql


2、创建Dockerfile：

    FROM javachen/alpine-jdk:8
    # Azkaban executor port
    EXPOSE 12321
    COPY azkaban-exec-server-0.1.0-SNAPSHOT.tar.gz .
    # coreutils https://www.cnblogs.com/shansongxian/p/10531439.html
    RUN  apk add --no-cache git coreutils mysql-client mongodb-tools postgresql-client redis \
        && tar zxf azkaban-exec-server-0.1.0-SNAPSHOT.tar.gz \
        && mv azkaban-exec-server-0.1.0-SNAPSHOT azkaban-exec-server \
        && rm -rf azkaban-exec-server-0.1.0-SNAPSHOT.tar.gz
    # Define default workdir
    WORKDIR azkaban-exec-server
    COPY conf/* conf/
    COPY run.sh run.sh
    RUN chmod +x run.sh
    CMD ./run.sh && tail -f /dev/null

3、创建conf目录，存储配置文件：

- azkaban.properties

4、创建run.sh文件

    #!/bin/bash
    DB_LOOPS="20"
    MYSQL_HOST="mysql"
    MYSQL_PORT="3306"
    START_CMD="bin/start-exec.sh"
    i=0
    while ! nc $MYSQL_HOST $MYSQL_PORT >/dev/null 2>&1 < /dev/null; do
      i=`expr $i + 1`
      if [ $i -ge $DB_LOOPS ]; then
        echo "$(date) - ${MYSQL_HOST}:${MYSQL_PORT} still not reachable, giving up"
        exit 1
      fi
      echo "$(date) - waiting for ${MYSQL_HOST}:${MYSQL_PORT}..."
      sleep 1
    done
    sed -i "s/ &//" $START_CMD
    # echo "import azkaban create-all-sql.sql to $MYSQL_HOST"
    # mysql -h $MYSQL_HOST -uazkaban -pazkaban azkaban < create-all-sql-0.1.0-SNAPSHOT.sql
    # Work around to run container as a daemon
    exec $START_CMD

注意：

- 从run.sh可以看到 azkaban-web-server 依赖 mysql 数据库，其主机名为 mysql ，所以需要部署一个mysql容器，容器名称设置为mysql，当然，你如果想用外置的sql，则可以设置为ip或者域名。

5、编译上传镜像：

    docker build -t javachen/azkaban-exec-server .
    docker push javachen/azkaban-exec-server



### docker-compose

创建一个网络：

    docker create network traefik

Docker-compose.yaml编排文件：

    version: '3'
    services:
      mysql:
        command: --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
        image: mysql:5.7
        restart: always
        environment:
          MYSQL_DATABASE: azkaban
          MYSQL_USER: azkaban
          MYSQL_PASSWORD: azkaban
          MYSQL_ROOT_PASSWORD: 123456
        ports:
          - "3306:3306"  
        volumes:
          - mysql:/var/lib/mysql
      executor:
        image: javachen/azkaban-exec-server
        restart: always
        # 和服务名称保持一致
        hostname: executor
        environment:
          AZKABAN_OPTS: "-Xmx2048m"
        ports:
          - "12321:12321"  
        volumes:
          - /data/bi_etl:/data/bi_etl
          - /data/ods:/data/ods #自定义的数据目录
        networks:
          - traefik
      webserver:
        image: javachen/azkaban-web-server
        restart: always
        environment:
          AZKABAN_OPTS: "-Xmx512m"
        depends_on:
          - executor
        networks:
          - traefik
        ports:
          - "8081:8081"
    networks:
      traefik:
        external: true

可以先启动mysql：

    docker-compose up -d mysql

然后需要初始化数据库，执行脚本：

    docker exec -it mysql_1 bash
    mysql -h localhost -uazkaban -pazkaban azkaban < create-all-sql-0.1.0-SNAPSHOT.sql

然后，启动azkaban：

    docker-compose up -d
