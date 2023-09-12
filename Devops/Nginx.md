

## 编译安装



### 安装目录及参数规划

- nginx安装目录：`/usr/local/nginx`
- nginx配置文件目录：`/usr/local/nginx/conf/nginx.conf`
- nginx虚拟服务器配置目录：`/usr/local/nginx/conf/vhost/`
- log日志目录：`/var/log/nginx/`
- pid文件目录：`/var/run/nginx.pid`
- lock锁目录：`/var/run/nginx.lock`
- 临时缓存目录：`/var/cache/nginx`
- 站点目录：`/www/wwwroot/`
- nginx运行用户名：`nginx`
- nginx运行用户组：`nginx` 

### 安装依赖

Linux环境下，安装 GCC编译器、正则表达式PCRE库、zlib压缩库、OpenSSL开发库

```bash
yum install gcc gcc-c++
```

Debian环境下安装：

```bash
apt-get -y update
apt-get -y install curl wget perl unzip build-essential libmaxminddb-dev libgd-dev
```



### configure的命令参数

- 列出configure包含的参数:`./configure --help` 

#### 通用配置选项解释

| 选项 | 解释 |
| :--- | :--- |
| --prefix=PATH | Ngi口x 安装的根路径，所有其他的安装路径都要依赖于该选项 |
| --sbin-path=PATH | 指定口ginx 二进制文件的路径。如果没有指定，那么这个路径会 依赖于 prefix 选项 |
| --conf-path=PATH | 如果在命令行没有指定配置文件，那么将会通过这里指定的路径，nginx 将会去那里查找它的配置文件 |
| --error-log-path=PATH | 指定错误文件的路径，nginx 将会往其中写入错误日志文件，除非有其他的配置 |
| --pid-path=PATH | 指定的文件将会写入nginx master进程的pid通常卸载/var/run/目录下 |
| --lock-path=PATH | 共享储存器互斥锁文件的路径 |
| --user=USER | worker进程运行的用户 |
| --group=GROUP | worker进程运行的用户组 |
| --with-file-aio | 为FreeBSD 4.3+和linux 2.6.22+系统启用异步I/O |
| --with-debug | 这个选项用于调试日志,在生产环境的系统中不推荐使用该选项 |



#### 临时路径配置选项

| 选项 | 解释 |
| :--- | :--- |
| --error-log-path=PATH | 错误日志的默认路径 |
| --http-log-path=PATH | http 访问日志的默认路径 |
| --http-client-body-temp-path=PATH | 从客户端收到请求后，该选项设置的目录用于作为请求体 临时存放的目录。如果 WebDAV 模块启用，那么推荐设置 该路径为同 一文件系统上的目录作为最终的目的地 |
| --http-proxy-temp-path=PATH | 在使用代理后，通过该选项设置存放临时文件路径 |
| --http-fastcgi-temp-path=PATH | 设置 FastCGI 临时文件的目录 |
| --http-uwsgi-temp-path=PATH | 设置 uWSG工临时文件的目录 |
| --http-scgi-temp-path=PATH | 设置 SCGII临时文件的目录 |



#### PCRE的配置参数

| 选项 | 解释 |
| :--- | :--- |
| --without-pcre | 如果确定Nginx不用解析正则表达式，那么可以使用这个参数 |
| --with-pcre | 强制使用PCRE库 |
| --with-pcre=DIR | 指定PCRE库的源码位置，在编译nginx时会进入该目录编译PCRE源码 |
| --with-pcre-opt=OPTIONS | 编译PCRE源码是希望加入的编译选项 |



#### OpenSSL的配置参数

| 选项 | 解释 |
| :--- | :--- |
| --with-openssl=DIR | 指定OpenSSL库的源码位置，在编译nginx时会进入该目录编译OpenSSL。如果web服务器需要使用HTTPS，那么Nginx要求必须使用OpenSSL |
| --with-openssl-opt=OPTIONS | 编译OpenSSL源码时希望加入的编译选项 |



#### zlib的配置参数

| 选项 | 解释 |
| :--- | :--- |
| --with-zlib=DIR | 指定zlib库的源码位置，在编译nginx时会进入该目录编译zlib。如果需要使用gzip压缩就必须要zlib库的支持 |
| --with-zlib-opt=OPTIONS | 编译zlib源码时希望加入的编译选项 |
| --with-zlib-asm=CPU | 指定对特定的CPU使用zlib库的汇编优化功能,目前支持两种架构：pentium和pentiumpro |



### Nginx编译步骤

1、创建用户与用户组

```bash
groupadd nginx
useradd -M -g nginx nginx -s /sbin/nologin
```

2、编译 openssl


安装 openssl

```bash
openssl_version='1.1.1g'
cd /usr/local
wget --no-check-certificate -O openssl.tar.gz https://www.openssl.org/source/openssl-1.1.1g.tar.gz
tar -zxvf openssl.tar.gz
cd openssl-1.1.1g
./config shared --openssldir=/usr/local/openssl --prefix=/usr/local/openssl
make && make install
echo "/usr/local/lib64/" >> /etc/ld.so.conf
ldconfig
openssl version
```

3、下载nginx源码包，解压并进入nginx源码根目录

```bash
nginx_version='1.19.9'
cd /usr/local
wget http://nginx.org/download/nginx-${nginx_version}.tar.gz
tar -zxvf nginx-${nginx_version}.tar.gz
cd nginx-${nginx_version}
```

4、生成makefile文件

```bash
mkdir /var/cache/nginx
./configure \
--prefix=/usr/local/nginx \
--sbin-path=/usr/local/nginx/sbin/nginx \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx.pid \
--lock-path=/var/run/nginx.lock \
--http-client-body-temp-path=/var/cache/nginx/client_temp \
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
--user=nginx \
--group=nginx \
--with-file-aio \
--with-threads \
--with-http_addition_module \
--with-http_auth_request_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_mp4_module \
--with-http_random_index_module \
--with-http_realip_module \
--with-http_secure_link_module \
--with-http_slice_module \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_sub_module \
--with-http_v2_module \
--with-mail \
--with-mail_ssl_module \
--with-stream \
--with-stream_realip_module \
--with-stream_ssl_module \
--with-stream_ssl_preread_module \
--with-openssl=../openssl
```

5、编译与安装

```bash
make -j4 && make -j4 install
```

6、安装成功执行以下命令查看nginx版本号

```bash
nginx -v
```

查看编译参数和模块：

```bash
nginx -V
```



### 编译其他可选模块

1、编译Lua

- 下载安装luaji：<http://luajit.org/download/LuaJIT-2.0.2.tar.gz>
- ngx\_devel\_kit下载：<https://github.com/simplresty/ngx_devel_kit/archive/v0.3.1rc1.tar.gz>
- lua-nginx-module下载：<https://github.com/openresty/lua-nginx-module/archive/v0.10.9rc7.tar.gz>

安装：

```bash
cd /usr/local
wget http://luajit.org/download/LuaJIT-2.0.2.tar.gz
tar -axv -f LuaJIT-2.0.2.tar.gz 
cd LuaJIT-2.0.2
make install PREFIX=/usr/local/LuaJIT
export LUAJIT_LIB=/usr/local/LuaJIT/lib
export LUAJIT_INC=/usr/local/LuaJIT/include/luajit-2.0
wget https://github.com/simplresty/ngx_devel_kit/archive/v0.3.1rc1.zip
wget https://github.com/openresty/lua-nginx-module/archive/v0.10.9rc7.tar.gz
tar -axf v0.3.1rc1.tar.gz 
tar -zxf v0.10.9rc7.tar.gz
mv lua-nginx-module-0.10.9rc7/ lua-nginx-module 
mv ngx_devel_kit-0.3.1rc1/ ngx_devel_kit
#重新生成makefile加入lua-module和lua-devel
cd /usr/local/nginx-${nginx_version}
./configure \
--prefix=/usr/local/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx.pid \
--lock-path=/var/run/nginx.lock \
--http-client-body-temp-path=/var/cache/nginx/client_temp \
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
--user=nginx \
--group=nginx \
--with-file-aio \
--with-threads \
--with-http_addition_module \
--with-http_auth_request_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_mp4_module \
--with-http_random_index_module \
--with-http_realip_module \
--with-http_secure_link_module \
--with-http_slice_module \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_sub_module \
--with-http_v2_module \
--with-mail \
--with-mail_ssl_module \
--with-stream \
--with-stream_realip_module \
--with-stream_ssl_module \
--with-stream_ssl_preread_module \
--add-module=../ngx_devel_kit \
--add-module=../lua-nginx-module \
--with-ld-opt="-Wl,-rpath,$LUAJIT_LIB"
#编译安装
make -j2 && make install
nginx -V
```

`nginx -V`出来能看到 `--add-module=./ngx_devel_kit --add-module=./lua-nginx-module`说明lua模块编译安装成功了。


2、编译 purecache

```bash
cd /usr/local && wget http://soft.xiaoz.org/nginx/ngx_cache_purge-2.3.tar.gz
tar -zxvf ngx_cache_purge-2.3.tar.gz
mv ngx_cache_purge-2.3 ngx_cache_purge
```

在nginx目录，重新生成makefile加入ngx\_cache\_purge模块：

```bash
--add-module=../ngx_cache_purge
```

3、编译安装 brotli

```bash
cd /usr/local
wget http://soft.xiaoz.org/nginx/ngx_brotli.tar.gz
tar -zxvf ngx_brotli.tar.gz
```

在nginx目录，重新生成makefile加入ngx\_cache\_purge模块：

```bash
--add-module=../ngx_brotli
```

4、编译pcre


安装 pcre

```bash
cd /usr/local
wget --no-check-certificate https://ftp.pcre.org/pub/pcre/pcre-${pcre_version}.tar.gz
tar -zxvf pcre-8.43.tar.gz
mv pcre-8.43 pcre
cd pcre
./configure
make -j4 && make -j4 install
```

在nginx目录，重新生成makefile加入pcre模块：

```bash
--with-pcre=../pcre \
--with-pcre-jit \
```

5、编译 zlib


安装zlib

```bash
cd /usr/local
wget http://soft.xiaoz.org/linux/zlib-1.2.11.tar.gz
tar -zxvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure
make -j4 && make -j4 install
```

在nginx目录，重新生成makefile加入zlib模块：

```bash
--with-zlib=../zlib-1.2.11 \
```



### 编译之后的配置

清理安装文件：

```bash
cd /usr/local
rm -rf zlib-1.*
rm -rf pcre-8.*
rm -rf ngx_cache_purge*
rm -rf ngx_brotli*
```

备份 nginx.conf：

```bash
mv /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx.conf.bak
mkdir -p /usr/local/nginx/conf/vhost
```

修改 nginx.conf：

    user nginx nginx;
    worker_processes  auto;
    worker_rlimit_nofile 50000;
    error_log  logs/error.log;
    #error_log  logs/error.log  notice;
    #error_log  logs/error.log  info;
    pid        /var/run/nginx.pid;
    events {
        use epoll;
        worker_connections 51200;
        #worker_connections  1024;
        multi_accept on;
    }
    http {
        include       mime.types;
        default_type  application/octet-stream;
        server_names_hash_bucket_size 128;
        client_header_buffer_size 32k;
        large_client_header_buffers 4 32k;
        client_max_body_size 1024m;
        client_body_buffer_size 10m;
        sendfile on;
        tcp_nopush on;
        keepalive_timeout 120;
        server_tokens off;
        tcp_nodelay on;
    		proxy_headers_hash_max_size 51200;
    		proxy_headers_hash_bucket_size 6400;
        
        #开启Brotli压缩
        #brotli on;
        #brotli_comp_level 6;
        #最小长度
        #brotli_min_length   512;
        #brotli_types text/plain text/javascript text/css text/xml text/x-component application/javascript application/x-javascript application/xml application/json application/xhtml+xml application/rss+xml application/atom+xml application/x-font-ttf application/vnd.ms-fontobject image/svg+xml image/x-icon font/opentype;
        #brotli_static       always;
        gzip on;
        gzip_buffers 16 8k;
        gzip_comp_level 6;
        gzip_http_version 1.1;
        gzip_min_length 256;
        gzip_proxied any;
        gzip_vary on;
        gzip_types
        text/xml application/xml application/atom+xml application/rss+xml application/xhtml+xml image/svg+xml
        text/javascript application/javascript application/x-javascript
        text/x-json application/json application/x-web-app-manifest+json
        text/css text/plain text/x-component
        font/opentype application/x-font-ttf application/vnd.ms-fontobject
        image/x-icon;
      	gzip_disable "MSIE [1-6]\.(?!.*SV1)";
        #If you have a lot of static files to serve through Nginx then caching of the files' metadata (not the actual files' contents) can save some latency.
        open_file_cache max=1000 inactive=20s;
        open_file_cache_valid 30s;
        open_file_cache_min_uses 2;
        open_file_cache_errors on;
        #limit connection
        limit_conn_zone $binary_remote_addr zone=addr:10m;
    	
        server {
            listen       80;
            server_name  localhost;
            #charset koi8-r;
            #access_log  logs/host.access.log  main;
            location / {
                root   html;
                index  index.html index.htm;
            }
            #error_page  404              /404.html;
            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
            # proxy the PHP scripts to Apache listening on 127.0.0.1:80
            #
            #location ~ \.php$ {
            #    proxy_pass   http://127.0.0.1;
            #}
            # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
            #
            #location ~ \.php$ {
            #    root           html;
            #    fastcgi_pass   127.0.0.1:9000;
            #    fastcgi_index  index.php;
            #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            #    include        fastcgi_params;
            #}
            # deny access to .htaccess files, if Apache's document root
            # concurs with nginx's one
            #
            #location ~ /\.ht {
            #    deny  all;
            #}
        }
        # another virtual host using mix of IP-, name-, and port-based configuration
        #
        #server {
        #    listen       8000;
        #    listen       somename:8080;
        #    server_name  somename  alias  another.alias;
        #    location / {
        #        root   html;
        #        index  index.html index.htm;
        #    }
        #}
        # HTTPS server
        #
        #server {
        #    listen       443 ssl;
        #    server_name  localhost;
        #    ssl_certificate      cert.pem;
        #    ssl_certificate_key  cert.key;
        #    ssl_session_cache    shared:SSL:1m;
        #    ssl_session_timeout  5m;
        #    ssl_ciphers  HIGH:!aNULL:!MD5;
        #    ssl_prefer_server_ciphers  on;
        #    location / {
        #        root   html;
        #        index  index.html index.htm;
        #    }
        #}
    	include vhost/*.conf;
    }

设置环境变量：

```bash
echo "export PATH=$PATH:/usr/local/nginx/sbin" >> /etc/profile
export PATH=$PATH:'/usr/local/nginx/sbin'
```

安装服务：

```bash
cat >> /etc/systemd/system/nginx.service <<EOF
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network.target
[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStartPost=/bin/sleep 0.1
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s QUIT $MAINPID
LimitNOFILE=1000000
LimitNPROC=1000000
LimitCORE=1000000
[Install]
WantedBy=multi-user.targe
EOF
```

设置开机启动：

```bash
systemctl daemon-reload
systemctl enable nginx
```

启动：

```bash
/usr/local/nginx/sbin/nginx
```

设置日志文件切割：

```bash
cat >> /etc/logrotate.d/nginx <<EOF
/data/wwwlogs/*nginx.log {
  daily
  rotate 5
  missingok
  dateext
  compress
  notifempty
  sharedscripts
  postrotate
    [ -e /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid`
  endscript
}
EOF
```

卸载

```bash
# 杀掉nginx进程
pkill nginx
#删除nginx用户
userdel nginx && groupdel nginx 
#备份一下配置
cp -a /usr/local/nginx/conf/vhost /home/vhost_bak
#删除目录
rm -rf /usr/local/nginx
sed -i "s%:/usr/local/nginx/sbin%%g" /etc/profile
#删除自启
sed -i '/^.*nginx/d' /etc/rc.d/rc.local
rm -rf /etc/systemd/system/nginx.service
#删除日志分割
rm -rf /etc/logrotate.d/nginx
```



## docker compose安装

创建目录：

    mkdir -p /data/docker/nginx/wwwroot/html80
    mkdir -p /data/docker/nginx/wwwroot/html8080
    mkdir -p /data/docker/nginx/conf

修改 `/data/docker/nginx/conf/nginx.conf` 目录下的 nginx.conf

```yml
version: '3.1'
services:
  nginx:
    restart: always
    image: nginx
    container_name: nginx
    ports:
      - 4180:80
      - 4181:8080
    volumes:
      - /data/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf
      - /data/docker/nginx/wwwroot:/usr/share/nginx/wwwroot
```

nginx.conf

    worker_processes  1;
    events {
        worker_connections  1024;
    }
    http {
        include       mime.types;
        default_type  application/octet-stream;
        sendfile        on;
        
        keepalive_timeout  65;
        server {
            listen       80;
            server_name  static.cshop.com;
    				# 所有的请求都以 / 开始，所有的请求都可以匹配此 location
            location / {
                root   /usr/share/nginx/wwwroot/html80;
                index  index.html index.htm;
            }
        }
        server {
            listen       8080;
            server_name  admin.cshop.com;
            location / {
                root   /usr/share/nginx/wwwroot/html8080;
                index  index.html index.htm;
            }
        }
    }

配置hosts：

    192.168.56.100 admin.cshop.com
    192.168.56.100 static.cshop.com



## 语法

Location 语法：

```bash
location [=||*|^~] /uri/ { … }
```

说明：

| 规则 | 说明 | 例子 |
| --- | --- | --- |
| = | 精准匹配 | location = /api/list |
| ~ | 正则匹配（区分大小写），支持正则 | location ~ /api/ |
| ~* | 正则匹配（**不** 区分大小写） | location ~* /api/ |
| !~ | 正则不匹配（区分大小写） | location !~ /api/ |
| !~* | 正则不匹配（**不** 区分大小写） | location !~* /api/ |
| ^~ | 字符串匹配（区分大小写），优先级高于正则 | location ^~ /api/ |
| / | 通用匹配 | location / |

查找顺序和优先级

- 带有“=“的精确匹配优先



- 没有修饰符的精确匹配



- 正则表达式按照他们在配置文件中定义的顺序



- 带有“^~”修饰符的，开头匹配



- 带有“” 或“*” 修饰符的，如果正则表达式与URI匹配



- 没有修饰符的，如果指定字符串与URI开头匹配




例子：

    server {
        listen              80;
        server_name         abc.com;
        access_log  "pipe:rollback /data/log/nginx/access.log interval=1d baknum=7 maxsize=1G"  main;
        location ^~/user/ {
            proxy_set_header Host $host;
            proxy_set_header  X-Real-IP        $remote_addr;
            proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
            proxy_set_header X-NginX-Proxy true;
            proxy_pass http://user/;
        }
        location ^~/order/ {
            proxy_set_header Host $host;
            proxy_set_header  X-Real-IP        $remote_addr;
            proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
            proxy_set_header X-NginX-Proxy true;
            proxy_pass http://order/;
        }
    }

`^~/user/`表示匹配前缀是user的请求，`proxy_pass`的结尾有 `/`， 则会把`/user/*`后面的路径直接拼接到后面，即移除`user`.
