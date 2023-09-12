

## 使用 Docker 安装

1、先安装 docker 和 docker-compose


2、安装数据库，这里使用 postgresql


创建数据库和用户：

```bash
CREATE USER gitea WITH PASSWORD 'gitea'; 
CREATE DATABASE gitea owner=gitea; 
GRANT ALL privileges ON DATABASE gitea TO gitea;
```

3、安装 nginx，建议通过编译源码进行安装。


安装之后，配置nginx，可以通过 gitea.chensoul.com 访问 gitea：

```bash
server {
    listen 80;
    listen [::]:80;
    server_name gitea.chensoul.com;
    location / {
        proxy_pass http://127.0.0.1:3000;
    }
}
```

另外，需要添加dns解析：

    gitea.chensoul.com
    postgres.chensoul.com

如果，需要开启SSL，则可以通过 acme.sh 申请证书：

- 我这里使用 Amazon Route53 的域名 <https://github.com/Neilpang/acme.sh/wiki/How-to-use-Amazon-Route53-API>

4、通过 docker-compose 安装 gitea


gitea.yaml

```yaml
version: "3"
services:
  server:
    image: gitea/gitea:1
    container_name: gitea
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - DB_TYPE=postgres
      - DB_HOST=postgres.chensoul.com:5432
      - DB_NAME=gitea
      - DB_USER=gitea
      - DB_PASSWD=gitea
    restart: always
    volumes:
      - /data/gitea:/data/gitea
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    ports:
      - "3000:3000"
      - "2222:22"
```

启动容器：

```bash
docker-compose -f gitea.yaml up -d
```

4、初始化 gitea，通过 <https://gitea.chensoul.com> 访问 gitea，并做相应设置


![](/Users/chensoul/IdeaProjects/chensoul.github.io/static/img/gitea-setup-1.png#)


![](/Users/chensoul/IdeaProjects/chensoul.github.io/static/img/gitea-setup-2.png#)


5、配置域名

    server {
        listen 80;
        listen [::]:80;
        server_name gitea.chensoul.com;
        return 301 https://$host$request_uri;
    }
    server {
        listen          443 ssl http2;
        listen          [::]:443 ssl http2;
        server_name     gitea.chensoul.com;
        
        # SSL
        ssl_certificate      /usr/local/nginx/ssl/fullchain.cer;
        ssl_certificate_key  /usr/local/nginx/ssl/chensoul.com.key;
        ssl_session_cache    shared:SSL:1m;
        ssl_session_timeout  5m;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers  on;
        
        # logging
        access_log          /var/log/nginx/gitea.chensoul.com.access.log;
        error_log           /var/log/nginx/gitea.chensoul.com.error.log warn;
        
        # reverse proxy
        location / {
            proxy_pass http://127.0.0.1:3000;
            
            # Proxy Config
            proxy_http_version                 1.1;
            proxy_cache_bypass                 $http_upgrade;
            # Proxy headers
            proxy_set_header Upgrade           $http_upgrade;
            proxy_set_header Connection        $connection_upgrade;
            proxy_set_header Host              $host;
            proxy_set_header X-Real-IP         $remote_addr;
            proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-Forwarded-Host  $host;
            proxy_set_header X-Forwarded-Port  $server_port;
            # Proxy timeouts
            proxy_connect_timeout              60s;
            proxy_send_timeout                 60s;
            proxy_read_timeout                 60s;
        }
        
        # additional config
        # favicon.ico
        location = /favicon.ico {
            log_not_found off;
            access_log    off;
        }
        
        # gzip
        gzip            on;
        gzip_vary       on;
        gzip_proxied    any;
        gzip_comp_level 6;
        gzip_types      text/plain text/css text/xml application/json application/javascript application/rss+xml application/atom+xml image/svg+xml;
    }

映射SSH端口：


<https://ssr.carrotwu.com/post/42>


[https://www.whao.fun/posts/2021/03/02/gitea-install.html]()


[https://www.escapelife.site/posts/af961a30.html]()


5、卸载

```bash
docker-compose -f git.yaml down
rm -rf /data/gitea/*
```
