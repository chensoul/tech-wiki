

## docker安装

获取rabbit镜像：

    docker pull rabbitmq:management
    firewall-cmd --zone=public --add-port=5672/tcp --permanent
    firewall-cmd --zone=public --add-port=15672/tcp --permanent

创建并运行容器：

    docker run -d --name rabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=123456 -p 15672:15672 -p 5672:5672 --restart=always rabbitmq:management

容器运行正常，使用<http://192.168.56.100:15672>访问rabbit控制台，用户名和密码admin和123456。


cshop-sms使用：

- 在管理界面添加一个虚拟主机/chsop，添加一个交换机cshop.sms.exchange，添加一个队里cshop.sms.queue，然后交换机绑定虚拟主机和队列。

cshop-email使用： 

## docker-compose安装

```yml
version: '3.1'
services:
  nexus:
    image: rabbitmq:management
    container_name: rabbitmq
    restart: always
    environment:
      RABBITMQ_DEFAULT_USER: cshop
      RABBITMQ_DEFAULT_PASS: cshop
    ports:
      - 15672:15672
      - 5672:5672
```
