GitLab 是利用 Ruby on Rails 一个开源的版本管理系统，实现一个自托管的 Git 项目仓库，可通过 Web 界面进行访问公开的或者私人项目。它拥有与 Github 类似的功能，能够浏览源代码，管理缺陷和注释。可以管理团队对仓库的访问，它非常易于浏览提交过的版本并提供一个文件历史库。团队成员可以利用内置的简单聊天程序 (Wall) 进行交流。它还提供一个代码片段收集功能可以轻松实现代码复用，便于日后有需要的时候进行查找。 

## yum安装



### 配置yum源

/etc/yum.repos.d/gitlab-ce.repo

```latex
[gitlab-ce]
name=Gitlab CE Repository
baseurl=https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el$releasever/
gpgcheck=0
enabled=1
```



### 安装GitLab社区版

```latex
yum install -y gitlab-ce
```



### GitLab常用命令

```bash
sudo gitlab-ctl start    # 启动所有 gitlab 组件；
sudo gitlab-ctl stop        # 停止所有 gitlab 组件；
sudo gitlab-ctl restart        # 重启所有 gitlab 组件；
sudo gitlab-ctl status        # 查看服务状态；
sudo gitlab-ctl reconfigure        # 启动服务；
sudo vim /etc/gitlab/gitlab.rb        # 修改默认的配置文件；
gitlab-rake gitlab:check SANITIZE=true --trace    # 检查gitlab；
sudo gitlab-ctl tail        # 查看日志；
```



### 修改配置文件

/etc/gitlab/gitlab.rb

```latex
external_url 'http://192.168.56.100:4080'
gitlab_rails['time_zone'] = 'Asia/Shanghai'
gitlab_rails['gitlab_shell_ssh_port'] = 4022
unicorn['listen'] = '192.168.56.100'
```



### 汉化（可选）

下载最新的汉化包

```latex
git clone https://gitlab.com/xhang/gitlab.git
```

查看该汉化补丁的版本

```latex
cd gitlab
cat VERSION
```

查看gitlab版本

```latex
head -1 /opt/gitlab/version-manifest.txt
```

比较汉化标签和原标签，导出 patch 用的 diff 文件到/root下

```latex
git diff v12.0.3 v12.0.3-zh > ../12.0.3-zh.diff
```

打补丁：

```bash
yum install patch -y 
patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < 12.0.3-zh.diff
```



### 配置gitlab

内存不够，我先加 swap空间

```bash
cd /mnt/swap/
dd if=/dev/zero of=swapfile bs=1M count=9999 
mkswap swapfile
swapon swapfile
```

添加开机自动挂

```latex
vim /etc/fstab 
/mnt/swap/swapfile swap swap defaults 0 0
```

查看：

```latex
top -c
```

设置让系统积极使用swap空间：

```latex
echo 100 > /proc/sys/vm/swappiness
```

sysctl -p 或者重启生效

```latex
sysctl -p
```

配置gitlab：

```latex
gitlab-ctl reconfigure
```



### 启动gitlab

```latex
gitlab-ctl start
```



### 测试

访问<http://192.168.56.100:4080/>，设置登陆密码为admin123，登陆用户为root。 

### 设置

第一次使用时需要做一些初始化设置，点击“管理区域”-->“设置”，账户与限制设置：关闭头像功能，由于 Gravatar 头像为网络头像，在网络情况不理想时可能导致访问时卡顿


由于是内部代码托管服务器，可以直接关闭注册功能，由管理员统一创建用户即可 

## docker安装

> 说明：用root安装

1、下载镜像文件

```bash
docker pull docker.io/gitlab/gitlab-ce-zh
```

2、 创建目录


将GitLab 的配置 (etc) 、 日志 (log) 、数据 (data) 放到容器之外， 便于日后升级， 因此请先准备这三个目录。

```latex
mkdir -p /data/docker/gitlab/conf
mkdir -p /data/docker/gitlab/logs
mkdir -p /data/docker/gitlab/data
```

3、运行容器

```bash
docker run -d --network "host" -p 4022:22 -p 4443:443 -p 4080:80  \
	-v /data/docker/gitlab/etc:/etc/gitlab \
	-v /data/docker/gitlab/logs:/var/log/gitlab \
	-v /data/docker/gitlab/data:/var/opt/gitlab \
	-v /etc/localtime:/etc/localtime:ro \
	--name gitlab docker.io/twang2218/gitlab-ce-zh
```

4、修改配置文件


进入容器修改配置文件：

```latex
docker exec -it gitlab vim /etc/gitlab/gitlab.rb
```

修改以下配置：

```latex
external_url 'http://192.168.56.100:4080'
gitlab_rails['gitlab_ssh_host'] = '192.168.56.100'
gitlab_rails['gitlab_shell_ssh_port'] = 4022
gitlab_rails['time_zone'] = 'Asia/Shanghai'
```

修改`/data/docker/gitlab/data/gitlab-rails/etc/gitlab.yml`，找到关键字`* ## Web server settings *` ，将host的值改成映射的外部主机IP地址 `192.168.56.100`，这里会显示在gitlab克隆地址。

```yaml
gitlab:
    ## Web server settings (note: host is the FQDN, do not include http://)
    host: 192.168.56.100
    port: 4080
    https: false
```

重启容器：

```latex
docker restart gitlab
```

查看日志：

```latex
docker logs -f gitlab
firewall-cmd --zone=public --add-port=8380/tcp --permanent
```

5、测试


访问 <http://192.168.56.100:4080>，修改默认密码为`admin123`，然后再以root用户登陆。


创建项目，然后再克隆代码进行测试。

> 问题：ssh端口映射失败，ssh协议暂时用不了。



## docker-compose安装

我们使用 Docker 来安装和运行 GitLab 中文版，由于新版本问题较多，这里我们使用目前相对稳定的 10.5 版本，`docker-compose.yml` 配置如下：

```yaml
version: '3'
services:
    gitlab:
      image: 'gitlab/gitlab-ce'
      container_name: "gitlab"
      restart: unless-stopped
      privileged: true
      hostname: 'gitlab.javachen.com'
      environment:
        TZ: 'Asia/Shanghai'
        GITLAB_OMNIBUS_CONFIG: |
          external_url 'http://gitlab.javachen.com:4080'
          gitlab_rails['time_zone'] = 'Asia/Shanghai'
          gitlab_rails['gitlab_shell_ssh_port'] = 4022
      ports:
        - '4080:8080'
      volumes:
        - /data/docker/gitlab/conf:/etc/gitlab
        - /data/docker/gitlab/data:/var/opt/gitlab
        - /data/docker/gitlab/logs:/var/log/gitlab
      extra_hosts:
        - "gitlab.javachen.com:192.168.56.100"
```

后台运行：

```latex
docker-compose up -d
```



## 安装GitLab Runner



### yum安装

```latex
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-ci-multi-runner/script.rpm.sh | sudo bash
yum install gitlab-ci-multi-runner -y
```

注册：

```latex
gitlab-ci-multi-runner register
```

说明：

- gitlab-ci-multi-runner register：执行注册命令
- Please enter the gitlab-ci coordinator URL：输入 ci 地址
- Please enter the gitlab-ci token for this runner：输入 ci token
- Please enter the gitlab-ci description for this runner：输入 runner 名称
- Please enter the gitlab-ci tags for this runner：设置 tag
- Whether to run untagged builds：这里选择 true ，代码上传后会能够直接执行
- Whether to lock Runner to current project：直接回车，不用输入任何口令
- Please enter the executor：选择 runner 类型，这里我们选择的是 shell

CI 的地址和令牌，在**创建的项目**的项目 --> 设置 --> CI/CD --> Runner 设置：


为保证能够正常集成，我们还需要一些其它配置：

- 安装完 GitLab Runner 后系统会增加一个 gitlab-runner 账户，我们将它加进 root 组：

```latex
gpasswd -a gitlab-runner root
```

- 配置需要操作目录的权限，比如你的 runner 要在 gaming 目录下操作：

```latex
chmod 775 gaming
```

- 由于我们的 shell 脚本中有执行 git pull 的命令，我们直接设置以 ssh 方式拉取代码：

```bash
su gitlab-runner
ssh-keygen -t rsa -C "你在 GitLab 上的邮箱地址"
cd 
cd .ssh
cat id_rsa.pub
```

- 复制 id\_rsa.pub 中的秘钥到 GitLab：

删除注册信息：

```latex
gitlab-ci-multi-runner unregister --name "名称"
```

查看注册列表：

```latex
gitlab-ci-multi-runner list
```



### docker安装

```bash
docker run -d --name gitlab-runner --restart always \
   -v /var/run/docker.sock:/var/run/docker.sock \
   --network=gitlab_net \
   --add-host=gitlab.javachen.com:192.168.56.100 \
   gitlab/gitlab-runner
```



### docker-compose安装

- 创建工作目录 `/data/docker/runner`
- 创建构建目录 `/data/docker/runner/environment`

在 `/usr/local/docker/runner` 目录下创建 `docker-compose.yml`

```yaml
version: "3.1"
services:
  runner:
    image: 'gitlab/gitlab-runner:latest'
    container_name: 'gitlab-runner'
    restart: always
    volumes:
      - '/data/gitlab-runner/conf:/etc/gitlab-runner'
      - '/var/run/docker.sock:/var/run/docker.sock'
    networks:
      - gitlab_net
    extra_hosts:
      - "gitlab.javachen.com:192.168.56.100"
networks:
  gitlab_net:
    external: true
```

注册：

```bash
docker exec -it gitlab-runner gitlab-runner register
```
