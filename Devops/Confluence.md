Confluence docker镜像地址：<https://hub.docker.com/r/atlassian/confluence-server> ，当前最新tag为7.3.1 

# 安装Postgres

```bash
docker run -d --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=123456 --restart=always postgres
```

创建数据库：

    docker exec -it postgres psql -Upostgres -w -c "CREATE DATABASE confluence WITH OWNER postgres;"

如果是重装，则需要重建数据库：

    REVOKE CONNECT ON DATABASE confluence FROM public;
    SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname='confluence' AND pid<>pg_backend_pid(); 
    drop database confluence;



# Docker安装



## 官方镜像安装

直接使用官方镜像安装：

    docker run -d \
      --name confluence \
      -p 8090:8090 -p 8091:8091 \
      -v /data/docker/confluence:/var/atlassian/application-data/confluence \
      --restart=always \
      atlassian/confluence-server:7.3.1



## 参数配置

常用参数配置Confluece：

- JVM参数：
- JVM\_MINIMUM\_MEMORY：默认1024M
- JVM\_MAXIMUM\_MEMORY：默认1024M
- JVM\_RESERVED\_CODE\_CACHE\_SIZE：默认256M
- JVM\_SUPPORT\_RECOMMENDED\_ARGS：额外设置，例如设置证书：`-Djavax.net.ssl.trustStore=/var/atlassian/application-data/confluence/cacerts`
- 代理设置：
- ATL\_PROXY\_NAME
- ATL\_PROXY\_PORT
- ATL\_TOMCAT\_PORT
- ATL\_TOMCAT\_SCHEME
- ATL\_TOMCAT\_SECURE
- ATL\_TOMCAT\_CONTEXTPATH
- ATL\_TOMCAT\_ACCESS\_LOG 

## 破解

从 <https://gitee.com/pengzhile/atlassian-agent> 下载破解文件，然后将破解文件拷贝到容器：

    wget https://github.com/javachen/dockerfiles/raw/master/confluence/atlassian-agent.jar
    docker cp atlassian-agent.jar confluence:/opt/atlassian/confluence/
    docker exec -it confluence bash
    echo 'export CATALINA_OPTS="-Duser.timezone=GMT+08 -javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' \
    	>> /opt/atlassian/confluence/bin/setenv.sh
    docker restart confluence

或者直接编译到镜像，Dockerfile：

    FROM  atlassian/confluence-server:7.3.1
    #ADD  http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.30.tar.gz /opt/atlassian/jira/lib/
    #COPY  mysql-connector-java-5.1.30-bin.jar /opt/atlassian/confluence/confluence/WEB-INF/lib
    COPY "atlassian-agent.jar" /opt/atlassian/confluence/
    RUN apt-get update && apt-get install tzdata \
      && ln -snf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo Asia/Shanghai >/etc/timezone \
      && dpkg-reconfigure --frontend noninteractive tzdata \
      && echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh

构建镜像：

    docker build -t javachen/confluence-server:7.3.1 .
    docker push javachen/confluence-server:7.3.1

获取license：

    # Confluence
    java -jar atlassian-agent.jar -p conf -m junecloud@163.com -n junecloud \
    	-o http://javachen.com -s BLYI-OHFJ-KTJN-M6C9
    # Confluence Questions
    java -jar atlassian-agent.jar -p questions -m junecloud@163.com -n junecloud \
    	-o http://javachen.com -s BOYX-UEMX-WRLS-MKBW
    # 团队日程表
    java -jar atlassian-agent.jar -p tc -m junecloud@163.com -n junecloud \
    	-o http://javachen.com -s BOYX-UEMX-WRLS-MKBW



## 安装破解版镜像

    docker run -d \
      --name confluence \
      -p 8090:8090 -p 8091:8091 \
      -v /data/docker/confluence:/var/atlassian/application-data/confluence \
      --restart=always \
      javachen/confluence-server:7.3.1

检查时区是否设置正确：

    $ docker exec -it confluence date
    Wed Apr 29 00:43:11 CST 2020



# K8s安装

创建命名空间和证书：

    kubectl create namespace confluence
    cat << EOF | kubectl create -f -   
    apiVersion: cert-manager.io/v1alpha2
    kind: Certificate
    metadata:
      name: confluence-test-wesine-com-cn-cert
      namespace: confluence
    spec:
      secretName: confluence-test-wesine-com-cn-cert
      renewBefore: 720h
      dnsNames:
      - "*.javachen.xyz"
      - "*.test.javachen.xyz"
      issuerRef:
        name: cert-manager-webhook-dnspod-cluster-issuer
        kind: ClusterIssuer
    EOF

cert-manager-webhook-dnspod-cluster-issuer是提前创建好的cluster-issuer。


下载chart文件：

    git clone https://github.com/javachen/charts/charts/
    cd charts

创建 confluence-values.yaml文件：

    cat <<EOF > confluence-values.yaml
    image:
      repository: javachen/confluence-server:7.3.1
    ingress:
      enabled: true
      annotations:
        kubernetes.io/ingress.class: nginx
        nginx.ingress.kubernetes.io/ssl-redirect: "true"
        nginx.ingress.kubernetes.io/proxy-body-size: 100m
      hosts:
        - confluence.javachen.xyz
        - confluence.test.javachen.xyz
      tls:
        - secretName: confluence-test-wesine-com-cn-cert
          hosts:
            - confluence.javachen.xyz
            - confluence.test.javachen.xyz
    env:
      - name: TZ
        value: Asia/Shanghai
      - name: JVM_MINIMUM_MEMORY
        value: "3072m"
      - name: JVM_MAXIMUM_MEMORY
        value: "4096m"
      - name: ATL_PROXY_NAME
        value: confluence.test.javachen.xyz
      - name: ATL_PROXY_PORT
        value: "443"
      - name: ATL_TOMCAT_SCHEME
        value: "https"
        
    persistence:
      enabled: true
      storageClass: "ceph-rbd"
      accessMode: ReadWriteOnce
      size: 20Gi
    EOF

使用helm3安装：

```bash
helm install confluence -n confluence -f confluence-values.yaml ./confluence
helm install confluence -f confluence-values.yaml ./confluence
```

卸载：

```bash
helm del confluence -n confluence
```

安装个人偏好的插件：RefinedToolki 

# 备份

    current_date=`date "+%Y_%m_%d" -d today`
    echo $current_date
    mkdir -p /data/backup/confluence
    #备份数据库
    docker exec -it postgres pg_dump -U postgres -d confluence -f confluence-${current_date}.sql
    docker cp postgres:/confluence-${current_date}.sql /data/backup/confluence/
    #备份confluence
    pod=$(kubectl get pod --output=name -n confluence |awk -F '/' '{print $2}')
    echo $pod
    #kubectl exec -it $pod -n confluence -- bash
    kubectl cp confluence/$pod:/var/atlassian/application-data/confluence/backups/backup-${current_date}.zip /data/backup/confluence/backup-${current_date}.zip



# 其他问题



## 异常

1、confluence ip变更之后，登陆失败，日志提示异常

    ApplicationPermissionException: Forbidden (403) Encountered a "403 - Forbidden" error while loading

参考：<https://community.atlassian.com/t5/Confluence-questions/ApplicationPermissionException-Forbidden-403-Encountered-a-quot/qaq-p/667228>


解决办法：

- K8s中，通过env环境变量设置 JVM\_MAXIMUM\_MEMORY 值为 `4096m -Datlassian.recovery.password=admin`
- 改用ldap目录 

## Confluenc宏乱码解决

1、方法1：下载中文编码：

    wget https://mirror.shileizcc.com/wiki_Resources/Confluence/chinses.tar.gz
    tar zxf chinses.tar.gz

dockerfile

    FROM  atlassian/confluence-server:7.5.0-m62-ubuntu
    # 将代理破解包加入容器
    COPY "atlassian-agent.jar" /opt/atlassian/confluence/
    COPY ./chinses/ /usr/share/fonts/truetype/chinses
    RUN chmod 644 /usr/share/fonts/truetype/chinese/* \
    		&& fc-cache -fv \
    		&& echo 'CATALINA_OPTS="-Dconfluence.document.conversion.fontpath=/usr/share/fonts/truetype ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh \
    	 && echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh

2、方法2：

    FROM  atlassian/confluence-server:7.5.0-m62-ubuntu
    # 安装微软字体并设置启动加载代理包
    RUN apt-get update \
    	&& apt-get install ttf-mscorefonts-installer \
    	&& echo 'export CATALINA_OPTS="-Dconfluence.document.conversion.fontpath=/usr/share/fonts/truetype/msttcorefonts ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh \
    	&& echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/confluence/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/confluence/bin/setenv.sh



## 修改忘记密码链接

参考 https://confluence.atlassian.com/doc/confluence-installation-and-upgrade-guide-214864161.html 

# 参考文章

- [使用Docker部署Confluence](https://www.jianshu.com/p/1813bf05f71e)
- \[confluence 安装及破解]\(<https://www.qinjj.tech/2019/01/04/confluence> install/)
- <https://gitee.com/pengzhile/atlassian-agent>
