

## 1、安装 Docker

通过脚本安装：

```bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```



## 2、安装 Compose

linux上安装：

```bash
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

设置执行权限：

```bash
$ sudo chmod +x /usr/local/bin/docker-compose
```

查看版本：

```bash
docker-compose --version
```
