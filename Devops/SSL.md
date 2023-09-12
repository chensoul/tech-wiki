

## 配置SSL证书

参考[免费申请HTTPS证书六大方法](https://zhuanlan.zhihu.com/p/138792764)：

- 阿里云
- 腾讯云
- certbot
- [acme.sh](https://github.com/acmesh-official/acme.sh/)
- [freessl.cn](https://freessl.cn/)
- [ohttps.com](http://ohttps.com)
- [lego](https://go-acme.github.io/) 

### Acme.sh

1、安装 **acme.sh**

```bash
curl  https://get.acme.sh | sh -s email=my@example.com
source ~/.bashrc
```

2、生成证书


**acme.sh** 实现了 **acme** 协议支持的所有验证协议。 一般有两种方式验证：http 和 dns 验证。**acme.sh** 目前支持 cloudflare、dnspod、cloudxns、godaddy 以及 ovh 等数十种解析商的自动集成。


以 AWS 为例

```bash
export  AWS_ACCESS_KEY_ID=AKIAXFCWT6IQHWGTX2PV
export  AWS_SECRET_ACCESS_KEY=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
.acme.sh/acme.sh --issue --server letsencrypt --dns dns_aws -d chensoul.com -d '*.chensoul.com'
```

3、安装证书


以 nginx 为例，将证书拷贝到 nginx 的对应目录下面

```bash
mkdir /usr/local/nginx/ssl/
.acme.sh/acme.sh --installcert -d chensoul.com -d *.chensoul.com\
  --cert-file /usr/local/nginx/ssl/chensoul.com.cer \
	--key-file /usr/local/nginx/ssl/chensoul.com.key \
	--fullchain-file /usr/local/nginx/ssl/fullchain.cer \
	--ca-file /usr/local/nginx/ssl/ca.cer \
  --reloadcmd "sudo nginx -s reload"
```

在 nginx 中配置ssl证书，以 gitea 为例：

```bash
server {
    listen 80;
    listen [::]:80;
    server_name gitea.chensoul.com;
    return 301 https://$host$request_uri;
}
server {
    listen          443 ssl;
    server_name     gitea.chensoul.com;
    ssl_certificate      /usr/local/nginx/ssl/fullchain.cer;
    ssl_certificate_key  /usr/local/nginx/ssl/chensoul.com.key;
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
    ssl_ciphers  HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers  on;
    location / {
        proxy_pass http://127.0.0.1:3000;
    }
}
```

4、更新证书


目前证书在 60 天以后会自动更新，你无需任何操作。今后有可能会缩短这个时间，不过都是自动的，你不用关心。
