

# 安装Postgres

    docker run -d --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=123456 postgres

创建数据库：

    docker exec -it postgres psql -Upostgres -w -c "CREATE DATABASE jira WITH OWNER postgres;"

如果是重装，则需要重建数据库：

    REVOKE CONNECT ON DATABASE jira FROM public;
    SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname='jira' AND pid<>pg_backend_pid(); 
    drop database jira;



# Docker安装

直接使用官方镜像安装：

    docker run -d --name jira \
     -p 8080:8080 \
     -v /data/docker/jira:/var/atlassian/application-data/jira \
     -e TZ=Asia/Shanghai \
     atlassian/jira-software:8.7.0

从 <https://gitee.com/pengzhile/atlassian-agent> 下载破解文件，然后将破解文件拷贝到容器：

    docker cp atlassian-agent.jar jira:/opt/atlassian/jira/
    docker exec -it jira bash
    echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/jira/atlassian-agent.jar ${CATALINA_OPTS}"' \
    	>> /opt/atlassian/jira/bin/setenv.sh
    docker restart jira

或者直接编译到镜像：

    # https://gitee.com/pengzhile/atlassian-agent
    FROM  atlassian/jira-software:8.7.0
    #ADD  http://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.30.tar.gz /opt/atlassian/jira/lib/
    #COPY  mysql-connector-java-5.1.30-bin.jar /opt/atlassian/jira/lib/
    # 将代理破解包加入容器
    COPY atlassian-agent.jar /opt/atlassian/jira/
    # 设置启动加载代理包
    RUN echo 'export CATALINA_OPTS="-javaagent:/opt/atlassian/jira/atlassian-agent.jar ${CATALINA_OPTS}"' >> /opt/atlassian/jira/bin/setenv.sh

构建镜像：

    docker build -t javachen/jira-software:8.7.0 .
    docker push javachen/jira-software:8.7.0

获取license：

    java -jar atlassian-agent.jar -p jira -m chenzj@javachen.com -n wesine \
    	-o http://www.javachen.com -s BABY-MGVE-GCQ1-OAZD



# K8s安装

创建命名空间和证书：

    kubectl create namespace jira
    cat << EOF | kubectl create -f -   
    apiVersion: cert-manager.io/v1alpha2
    kind: Certificate
    metadata:
      name: jira-test-wesine-com-cn-cert
      namespace: jira
    spec:
      secretName: jira-test-wesine-com-cn-cert
      renewBefore: 720h
      dnsNames:
      - "*.javachen.xyz"
      issuerRef:
        name: cert-manager-webhook-dnspod-cluster-issuer
        kind: ClusterIssuer
    EOF

cert-manager-webhook-dnspod-cluster-issuer是提前创建好的cluster-issuer。


下载chart文件：

    git clone https://github.com/junetalk/charts
    cd charts

创建 jira-values.yaml 文件：

    cat <<EOF > jira-values.yaml
    image:
      repository: javachen/jira-software:8.7.0
    ingress:
      enabled: true
      annotations:
        kubernetes.io/ingress.class: nginx
        nginx.ingress.kubernetes.io/ssl-redirect: "true"
        nginx.ingress.kubernetes.io/proxy-body-size: 100m
      hosts:
        - jira.javachen.xyz
        - jira.test.javachen.xyz
      tls:
        - secretName: jira-test-wesine-com-cn-cert
          hosts:
            - jira.javachen.xyz
    env:
      - name: TZ
        value: Asia/Shanghai
      - name: JVM_MINIMUM_MEMORY
        value: "3072m"
      - name: JVM_MAXIMUM_MEMORY
        value: "4096m"
      - name: ATL_PROXY_NAME
        value: jira.javachen.com
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

    helm install jira -n jira -f jira-values.yaml ./jira

卸载：

    helm del jira -n jira



# 集成OpenLDAP

参考：

- <https://www.jianshu.com/p/a33d5305ba5b>



- [https://www.cnblogs.com/Bourbon-tian/p/8675784.html]()



- <https://www.jianshu.com/p/c877b317f294>



- <https://yq.aliyun.com/articles/560943>




首先，设置memberof：

    dn: cn=module,cn=config
    cn: module
    objectClass: olcModuleList
    olcModuleLoad: memberof.la
    olcModulePath: /usr/lib64/openldap
    dn: olcOverlay=memberof,olcDatabase={1}hdb,cn=config
    objectClass: olcConfig
    objectClass: olcMemberOf
    objectClass: olcOverlayConfig
    objectClass: top
    olcOverlay: memberof
    olcMemberOfDangling: ignore
    olcMemberOfRefInt: TRUE
    olcMemberOfGroupOC: groupOfUniqueNames                      
    olcMemberOfMemberAD: uniqueMember                           
    olcMemberOfMemberOfAD: memberOf

接着使用 `ldapadd` 命令将其导入：

    ldapadd -Y EXTERNAL -H ldapi:/// -f memberof.ldif

创建组：

    cat > basedomain.ldif <<EOF
    dn: dc=javachen,dc=com
    objectClass: top
    objectClass: dcObject
    objectclass: organization
    o: Javachen
    dc: javachen
    dn: cn=admin,dc=javachen,dc=com
    objectClass: organizationalRole
    cn: admin
    description: Directory Manager
    dn: ou=People,dc=javachen,dc=com
    objectClass: organizationalUnit
    ou: People
    dn: ou=Group,dc=javachen,dc=com
    objectClass: organizationalUnit
    ou: Group
    EOF
    ldapadd -x -D cn=admin,dc=javachen,dc=com -W -f basedomain.ldif

创建添加用户脚本add\_user.sh：

    #!/bin/bash
    # const
    LDAP_SERVER_IP="localhost"
    LDAP_SERVER_PORT="389"
    LDAP_ADMIN_USER="cn=admin,dc=javachen,dc=com"
    LDAP_ADMIN_PASS="admin"
    if [ x"$#" != x"3" ];then
        echo "Usage: $0 <username> <realname>"
        exit -1
    fi
    # param
    USER_ID="$1"
    SN="$2"
    NAME="$3"
    PASSWORD="123456"
    ENCRYPT_PASSWORD=$(slappasswd -h {ssha} -s "$PASSWORD")
    # add count & group 
    cat <<EOF | ldapmodify -c -h $LDAP_SERVER_IP -p $LDAP_SERVER_PORT \
        -w $LDAP_ADMIN_PASS -D $LDAP_ADMIN_USER 
    dn: cn=$USER_ID,ou=People,javachen,dc=com
    changetype: add
    objectClass: top
    objectClass: person
    objectClass: organizationalPerson
    objectClass: inetOrgPerson
    cn: $USER_ID
    sn: $SN
    givenName: $NAME
    displayName: $SN$NAME
    mail: $USER_ID@javachen.com
    userPassword: $ENCRYPT_PASSWORD
    dn: cn=jira,ou=Group,dc=javachen,dc=com
    changetype: modify
    add: uniqueMember
    uniqueMember: cn=$USER_ID,ou=People,javachen,dc=com
    EOF

添加用户：

    add_user.sh zhangsan 张 三

在jiar中添加openldap目录：


![](../assets/devops_jira/008i3skNgy1gtm2pln41sj61010u077802.jpg)


设置用户模式：


![](../assets/devops_jira/008i3skNgy1gtm2pmg91mj61290u0ju502.jpg)


设置组成员模式：


![](../assets/devops_jira/008i3skNgy1gtm2plwpqhj61960owju002.jpg)


输入用户和秘密进行测试：


![](../assets/devops_jira/008i3skNgy1gtm2pmw3yxj611e0u0goe02.jpg)
