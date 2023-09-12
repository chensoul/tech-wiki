本文主要记录在虚拟机中安装Ceph单节点集群的过程，其实多配置几个节点，也就是安装集群的过程。


Ceph中午文档：<http://docs.ceph.org.cn/> 

# 集群规划

集群环境：

| 主机 | IP | 操作系统 | 软件 | 组件 |
| --- | --- | --- | --- | --- |
| k8s-rke-node001 | 192.168.56.111 | Centos7.7.1908 | Ceph 14.2.4 | ceph\_deploy、OSD、MON |
| k8s-rke-node002 | 192.168.56.112 | Centos7.7.1908 | Ceph 14.2.4 | OSD、MON |
| k8s-rke-node003 | 192.168.56.113 | Centos7.7.1908 | Ceph 14.2.4 | OSD、MON |

安装用户：chenzj 

# 准备环境

你的管理节点必须能够通过 SSH 无密码地访问各 Ceph 节点。如果 `ceph-deploy` 以某个普通用户登录，那么这个用户必须有无密码使用 `sudo` 的权限。 

## 设置hosts

```bash
echo << EOF > /etc/hosts
192.168.56.111 k8s-rke-node001
192.168.56.112 k8s-rke-node002
192.168.56.113 k8s-rke-node003
EOF
```



## 创建部署 CEPH 的用户

`ceph-deploy` 工具必须以普通用户登录 Ceph 节点，且此用户拥有无密码使用 `sudo` 的权限，因为它需要在安装软件及配置文件的过程中，不必输入密码。


在各 Ceph 节点创建新用户并设置密码。

```bash
USER=chenzj
useradd -G docker -d /home/$USER -m $USER 
echo $USER|passwd $USER --stdin >/dev/null 2>&1
```

确保各 Ceph 节点上新创建的用户都有 `sudo` 权限。

```bash
sudo echo "$USER ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$USER
sudo chmod 0440 /etc/sudoers.d/$USER
```



## 允许无密码 SSH 登录

正因为 `ceph-deploy` 不支持输入密码，你必须在管理节点上生成 SSH 密钥并把其公钥分发到各 Ceph 节点。 `ceph-deploy` 会尝试给初始 monitors 生成 SSH 密钥对。


生成 SSH 密钥对，但不要用 `sudo` 或 `root` 用户。提示 “Enter passphrase” 时，直接回车，口令即为空：

```bash
su - $USER
yes|ssh-keygen -f ~/.ssh/id_rsa -t rsa -N ""
```

把公钥拷贝到各 Ceph 节点

```bash
ssh-copy-id $USER@k8s-rke-node001
ssh-copy-id $USER@k8s-rke-node002
ssh-copy-id $USER@k8s-rke-node003
```

修改 `ceph-deploy` 管理节点上的 `~/.ssh/config` 文件，这样 `ceph-deploy` 就能用你所建的用户名登录 Ceph 节点了，而无需每次执行 `ceph-deploy` 都要指定 `--username`

```bash
cat << EOF >> ~/.ssh/config
Host      	k8s-rke-node001
	Hostname  k8s-rke-node001
	User      $USER
Host      	k8s-rke-node002
	Hostname  k8s-rke-node002
	User      $USER
Host      	k8s-rke-node003
	Hostname  k8s-rke-node003
	User      $USER
EOF
chmod 644 ~/.ssh/config
```



## 配置时钟同步



## 设置SELINUX

在 CentOS 和 RHEL 上， SELinux 默认为 `Enforcing` 开启状态。为简化安装，我们建议把 SELinux 设置为 `Permissive` 或者完全禁用，也就是在加固系统配置前先确保集群的安装、配置没问题。用下列命令把 SELinux 设置为 `Permissive` ：

```bash
sudo setenforce 0
```



## 设置TTY

在 CentOS 和 RHEL 上执行 `ceph-deploy` 命令时可能会报错。如果你的 Ceph 节点默认设置了 `requiretty` ，执行 `sudo visudo` 禁用它，并找到 `Defaults requiretty` 选项，把它改为 `Defaults:ceph !requiretty` 或者直接注释掉，这样 `ceph-deploy` 就可以用之前创建的用户（[创建部署 Ceph 的用户](http://docs.ceph.org.cn/start/quick-start-preflight/#id3) ）连接了。

```bash
sed -i 's/Defaults *requiretty/#Defaults requiretty/g' /etc/sudoers
sed -i 's/Defaults *!visiblepw/Defaults   visiblepw/g' /etc/sudoers
```



## 安装yum-plugin-priorities

确保你的包管理器安装了优先级/首选项包且已启用。

```bash
sudo yum install yum-plugin-priorities -y
```



## 加载rbd模块

    sudo modprobe rbd



# 创建集群



## 安装 Ceph-Deploy

把 Ceph 仓库添加到 `ceph-deploy` 管理节点，然后安装 `ceph-deploy` 。


安装包管理工具：

```bash
sudo yum install -y yum-utils curl -k -s -o /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo
```

配置yum源，注意：这里ceph使用的是nautilus版本。

```bash
cat << EOF > ceph.repo[ceph-base]name=Ceph Base packagesbaseurl=https://mirrors.cloud.tencent.com/ceph/rpm-nautilus/el7/x86_64enabled=1gpgcheck=1type=rpm-mdgpgkey=https://download.ceph.com/keys/release.asc[ceph-noarch]name=Ceph noarch packagesbaseurl=https://mirrors.cloud.tencent.com/ceph/rpm-nautilus/el7/noarchenabled=1gpgcheck=1priority=1type=rpm-mdgpgkey=http://mirrors.cloud.tencent.com/ceph/keys/release.ascEOFsudo mv ceph.repo /etc/yum.repos.d/
```

设置环境变量，使ceph-deploy使用腾讯云源。

```bash
export CEPH_DEPLOY_REPO_URL=https://mirrors.cloud.tencent.com/ceph/rpm-nautilus/el7export CEPH_DEPLOY_GPG_URL=https://mirrors.cloud.tencent.com/ceph/keys/release.asc
```

更新软件库并安装 `ceph-deploy`

```bash
sudo yum update -y && sudo yum install ceph-deploy python-pip -y
```



## 创建配置目录

先在ceph-deploy管理节点上创建一个目录，用于保存 `ceph-deploy` 生成的配置文件和密钥对。

```bash
mkdir ceph-cluster && cd ceph-cluster
```



## 创建集群

在管理节点上，进入刚创建的放置配置文件的目录，用 `ceph-deploy` 执行如下步骤，安装规划在k8s-rke-node001节点初始化Monitor。


1、创建集群，部署新的 monitor 节点

```bash
ceph-deploy new k8s-rke-node001 \  --public-network 192.168.56.0/24 \  --cluster-network 192.168.56.0/24
```

> 注意：
>
> - 如有多网卡环境，应该把集群网络和数据网络区分开：（官方文档：<http://docs.ceph.org.cn/rados/configuration/network-config-ref/>）
>
>
>
> - public-network要和当前ip的网段保持对应。
>
>
>

2、在当前目录下用 `ls` 和 `cat` 检查 `ceph-deploy` 的输出，应该有一个 Ceph 配置文件、一个 monitor 密钥环和一个日志文件。

```bash
-rw-r--r-- 1 root root    291 9月  17 22:15 ceph.conf-rw-r--r-- 1 root root 212100 9月  17 22:16 ceph-deploy-ceph.log-rw------- 1 root root     73 9月  16 09:40 ceph.mon.keyring
```

3、如果出现异常：

```bash
Traceback (most recent call last):  File "/usr/bin/ceph-deploy", line 18, in <module>    from ceph_deploy.cli import main  File "/usr/lib/python2.7/site-packages/ceph_deploy/cli.py", line 1, in <module>    import pkg_resourcesImportError: No module named pkg_resources
```

安装相关依赖即可：

```bash
sudo yum install python-pkg-resources python-setuptools -y
```

4、把 Ceph 配置文件里的默认副本数从 `3` 改成 `2` ，这样只有两个 OSD 也可以达到 `active + clean` 状态。把下面这行加入 `[global]` 段：

    echo "osd_pool_default_size =2[mon]mon_allow_pool_delete = truemon_max_pg_per_osd = 1024#时钟同步mon_clock_drift_allowed = 2mon_clock_drift_warn_backoff = 30" >> ceph.conf



## 安装 Ceph 到各节点

```bash
ceph-deploy install k8s-rke-node001 k8s-rke-node002 k8s-rke-node003
```

此过程需要等待一段时间，因为 ceph-deploy 会 SSH 登录到各 node 上去，依次执行安装 ceph 依赖的组件包。


如果下载太慢，可以设置镜像：

```bash
sudo rpm --import https://mirrors.cloud.tencent.com/ceph/keys/release.ascceph-deploy install --repo-url https://mirrors.cloud.tencent.com/ceph/rpm-nautilus k8s-rke-node001
```

如果还是太慢，则在**每个节点配置ceph的yum源**，然后通过yum安装：

```bash
sudo yum -y install ceph ceph-radosgw
```

安装成功之后，检查每个节点的ceph版本是否一致：

    sudo ceph --versionceph version 14.2.4 (75f4de193b3ea58512f204623e6c5a16e6c1e1ba) nautilus (stable)



## 初始化 Monitor

初始化 Monitor , 获取秘钥，生产对应的 key：（服务默认端口 6789）

    ceph-deploy mon create-initial

完成上述操作后，当前目录里应该会出现这些密钥：

    ceph.client.admin.keyringceph.bootstrap-mgr.keyringceph.bootstrap-osd.keyringceph.bootstrap-mds.keyringceph.bootstrap-rgw.keyringceph.bootstrap-rbd.keyringceph.bootstrap-rbd-mirror.keyring



## 初始化磁盘

    fdisk -lsudo mkfs /dev/sdbparted /dev/sdb



## 创建 OSD 存储节点

列出节点所有磁盘信息：

```bash
ceph-deploy disk list k8s-rke-node001ceph-deploy disk list k8s-rke-node002ceph-deploy disk list k8s-rke-node003
```

创建 OSD 存储节点:（服务默认端口 74053）

    ceph-deploy osd create k8s-rke-node001 --data /dev/sdb1ceph-deploy osd create k8s-rke-node002 --data /dev/sdb1ceph-deploy osd create k8s-rke-node003 --data /dev/sdb1

查看创建的osd：

    ceph-deploy osd list k8s-rke-node001ceph-deploy osd list k8s-rke-node002ceph-deploy osd list k8s-rke-node003

清除磁盘分区和内容：

    ceph-deploy disk zap k8s-rke-node001 /dev/sdbceph-deploy disk zap k8s-rke-node002 /dev/sdbceph-deploy disk zap k8s-rke-node003 /dev/sdb



## 分配 admin 配置文件

用 `ceph-deploy` 把配置文件和 admin 密钥拷贝到管理节点和 Ceph 节点，这样你每次执行 Ceph 命令行时就无需指定 monitor 地址和 `ceph.client.admin.keyring` 了。

```bash
ceph-deploy --overwrite-conf admin k8s-rke-node001 k8s-rke-node002 k8s-rke-node003
```

确保你对 `ceph.client.admin.keyring` 有正确的操作权限。

```bash
sudo chmod +r /etc/ceph/ceph.client.admin.keyring
```



## 查看集群状态

    ceph -s

此时出现错误，没有活跃的 mgr 服务。 

## 部署 MGR

```bash
ceph-deploy mgr create k8s-rke-node001 k8s-rke-node002 k8s-rke-node003
```

继续查看状态：

    ceph -s

此时状态 HEALTH\_OK 则集群正常。 

## 安装 Dashboard

安装 Dashboard，首先必须有 mgr


开启模块：

    ceph mgr module enable dashboard

创建证书：

    ceph dashboard create-self-signed-cert

创建密码：

    ceph dashboard set-login-credentials admin admin

查看服务访问：

    ceph mgr services

修改**ceph.conf**配置：

    [mgr]mgr modules = dashboard

推送配置文件：

    ceph-deploy --overwrite-conf admin k8s-rke-node001 k8s-rke-node002 k8s-rke-node003

重启服务：

    systemctl restart ceph*.service ceph*.target

可以修改访问地址：

    $ ceph config-key put mgr/dashboard/server_addr 34.97.202.172$ ceph config-key put mgr/dashboard/server_port 8000



## 清除集群

```bash
$ ceph-deploy purge k8s-rke-node001 k8s-rke-node002 k8s-rke-node003$ ceph-deploy purgedata k8s-rke-node001 k8s-rke-node002 k8s-rke-node003$ ceph-deploy forgetkeys
```



# 配置调优

配置文件优化如下：

    [global]fsid = bf24f836-509b-4dbb-9f73-2518818a5cd2mon_initial_members = instance-template-3, instance-template-4, instance-template-5mon_host = 10.174.0.16,10.174.0.17,10.174.0.18 # 集群认证auth_cluster_required = cephx # 服务认证auth_service_required = cephx # 客户端认证auth_client_required = cephx # 如果设置了该选项，Ceph 会设置系统的 max open fds，默认值为 0max open files = 0 # 最小副本数 默认是3osd pool default size = 3  #PG 处于 degraded 状态不影响其 IO 能力,min_size 是一个PG 能接受 IO 的最小副本数osd pool default min size = 1  [mon.a]host = instance-template-3mon addr = 10.174.0.16:6789 [mon.b]host = instance-template-4mon addr = 10.174.0.17:6789 [mon.c]host = instance-template-5mon addr = 10.174.0.18:6789 [mon]mon data = /var/lib/ceph/mon/ceph-$id # monitor 间的 clock drift，默认值 0.05mon clock drift allowed = 1 # 向 monitor 报告 down 的最小 OSD 数，默认值 1mon osd min down reporters = 1 # 标记一个OSD状态为down和out之前ceph等待的秒数，默认值300mon osd down out interval = 600 [osd]# osd 数据路径osd data = /var/lib/ceph/osd/ceph-$id # 默认 pool pg,pgp 数量osd pool default pg num =  1024osd pool default pgp num = 1024 # osd 的 journal 写日志时的大小默认 5120osd journal size = 20000 # 格式化文件系统类型osd mkfs type = xfs # 格式化文件系统时附加参数osd mkfs options xfs = -f # 为 XATTRS 使用 object map，EXT4 文件系统时使用，XFS 或者 btrf 也可以使用，默认 falsefilestore xattr use omap = true # 从日志到数据盘最小同步间隔(seconds)，默认值 0.1filestore min sync interval = 10 # 从日志到数据盘最大同步间隔(seconds)，默认值 5filestore max sync interval = 15 # 数据盘最大接受的操作数，默认值 500filestore queue max ops = 25000 # 数据盘能够 commit 的最大字节数(bytes)，默认值 100filestore queue max bytes = 10485760 # 数据盘能够 commit 的操作数，500filestore queue committing max ops = 5000 # 数据盘能够 commit 的最大字节数(bytes)，默认值 100filestore queue committing max bytes = 10485760000 # 前一个子目录分裂成子目录中的文件的最大数量，默认值 2filestore split multiple = 8 # 前一个子类目录中的文件合并到父类的最小数量，默认值10filestore merge threshold = 40 # 对象文件句柄缓存大小，默认值 128filestore fd cache size = 1024 # 并发文件系统操作数，默认值 2filestore op threads = 32 # journal 一次性写入的最大字节数(bytes)，默认值 1048560journal max write bytes = 1073714824 # journal一次性写入的最大记录数，默认值 100journal max write entries = 10000 # journal一次性最大在队列中的操作数，默认值 50journal queue max ops = 50000 # journal一次性最大在队列中的字节数(bytes)，默认值 33554432journal queue max bytes = 10485760000 # # OSD一次可写入的最大值(MB), 默认 90osd max write size = 512 # 客户端允许在内存中的最大数据(bytes), 默认值100osd client message size cap = 2147483648 # 在 Deep Scrub 时候允许读取的字节数(bytes), 默认值524288osd deep scrub stride = 131072 # 并发文件系统操作数, 默认值 2osd op threads = 8 # OSD 密集型操作例如恢复和 Scrubbing 时的线程, 默认值1osd disk threads = 4 # 保留 OSD Map 的缓存(MB), 默认 500osd map cache size = 1024 # OSD 进程在内存中的 OSD Map 缓存(MB), 默认 50osd map cache bl size = 128 # 默认值rw,noatime,inode64, Ceph OSD xfs Mount选项osd mount options xfs = "rw,noexec,nodev,noatime,nodiratime,nobarrier" # 恢复操作优先级，取值 1-63，值越高占用资源越高, 默认值 10osd recovery op priority = 4 # 同一时间内活跃的恢复请求数, 默认值 15osd recovery max active = 10 # 一个 OSD 允许的最大 backfills 数, 默认值 10osd max backfills = 4 [client]# RBD缓存, 默认 truerbd cache = true # RBD缓存大小(bytes), 默认 335544320（320M）rbd cache size = 268435456 # 缓存为 write-back 时允许的最大 dirty 字节数(bytes)，如果为0，使用 write-through，默认值为 25165824rbd cache max dirty = 134217728 # 在被刷新到存储盘前 dirty 数据存在缓存的时间(seconds), 默认值为 1rbd cache max dirty age = 5 # 该选项是为了兼容 linux-2.6.32 之前的 virtio 驱动，避免因为不发送 flush 请求，数据不回写 默认值 true # 设置该参数后，librbd 会以 writethrough 的方式执行 io，直到收到第一个 flush 请求，才切换为 writeback 方式。# rbd cache writethrough until flush = false # 最大的 Object 对象数，默认为 0，表示通过 rbd cache size 计算得到，librbd 默认以 4MB 为单位对磁盘 Image 进行逻辑切分，默认值 0# 每个 chunk 对象抽象为一个 Object；librbd 中以 Object 为单位来管理缓存，增大该值可以提升性能# rbd cache max dirty object = 2 # 开始执行回写过程的脏数据大小，不能超过 rbd_cache_max_dirty，默认值 16777216# rbd cache target dirty = 235544320  # GW 端口，默认 7480# rgw frontends = civetweb port=7480

配置完成后，需要重启加载配置文件到所有节点：

    ceph-deploy --overwrite-conf admin k8s-rke-node001 k8s-rke-node002 k8s-rke-node003

重启服务：

```bash
systemctl restart ceph*.service ceph*.target
```



# 集群扩容

一个基本的集群启动并开始运行后，下一步就是扩展集群。在 k8s-rke-node004 上添加一个 OSD 守护进程和一个元数据服务器。然后分别在 k8s-rke-node002 和 k8s-rke-node003 上添加 Ceph Monitor ，以形成 Monitors 的法定人数。 

## 添加 OSD

在 k8s-rke-node004 上添加一个 OSD：

```bash
ssh k8s-rke-node004 "sudo mkdir -p /data/ceph/osd"ceph-deploy osd create --data /data/ceph/osd k8s-rke-node004
```

一旦你新加了 OSD ， Ceph 集群就开始重均衡，把归置组迁移到新 OSD 。可以用下面的 `ceph` 命令观察此过程：

    ceph -w

你应该能看到归置组状态从 `active + clean` 变为 `active` ，还有一些降级的对象；迁移完成后又会回到 `active + clean` 状态（ Control-C 退出）。


查看 osd 状态：

    ceph osd tree



## 添加 Monitor

Ceph 存储集群需要至少一个 Monitor 才能运行。为达到高可用，典型的 Ceph 存储集群会运行多个 Monitors，这样在单个 Monitor 失败时不会影响 Ceph 存储集群的可用性。Ceph 使用 PASOX 算法，此算法要求有多半 monitors（即 1 、 2:3 、 3:4 、 3:5 、 4:6 等 ）形成法定人数。


k8s-rke-node002 和 k8s-rke-node003 上添加 Ceph Monitor

    ceph-deploy mon add k8s-rke-node002ceph-deploy mon add k8s-rke-node003

新增 Monitor 后，Ceph 会自动开始同步并形成法定人数。你可以用下面的命令检查法定人数状态：

    ceph quorum_status --format json-pretty

- 请确保ntp时钟同步 

## 添加 MDS

如果需要使用 Cephfs，则需要创建 MDS:

    ceph-deploy mds create k8s-rke-node001

> 注意：
> 当前生产环境下的 Ceph 只能运行一个元数据服务器。

查看服务状态：

```bash
$ netstat -lnputActive Internet connections (only servers)Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name...tcp        0      0 0.0.0.0:6805            0.0.0.0:*               LISTEN      71047/ceph-mds
```



## 添加 RGW

添加 RGW 支持使用 Object Storage 存储：(服务端 7480)

    ceph-deploy rgw create k8s-rke-node001

可以更改该节点 ceph.conf 内服务端口：

```bash
[client]rgw frontends = civetweb port=80
```

浏览器访问：<http://192.168.56.111:7480> 

# 使用



## CephFS

查看 pool 存储池：

```bash
ceph osd pool ls
```

创建 pool 名为 test\_data 的 pg 32：

```bash
ceph osd pool create test_data 32
```

> pg 数量：
> 创建pool 通常在创建pool之前，需要覆盖默认的 pg\_num，官方推荐： 若少于 5 个OSD， 设置 pg\_num 为 128。 5~10 个 OSD，设置 pg\_num 为 512。 10~50 个 OSD，设置 pg\_num 为 4096。 超过 50 个 OSD，可以参考 pgcalc 计算。

```bash
Total PGs = (Total_number_of_OSD * 100) / max_replication_count
```

创建 pool 名为 test\_metadata 的 pg 32：

```bash
ceph osd pool create test_metadata 32
```

创建名为 data 的 fs：

```bash
$ ceph fs new data test_metadata test_datanew fs with metadata pool 10 and data pool 9
```

查看 cephfs 列表：

```bash
$ ceph fs lsname: data, metadata pool: test_metadata, data pools: [test_data ]
```

安装 ceph-fuse 进行挂载文件系统的支持：

```bash
sudo yum install -y ceph-fuse
```

挂载：

```bash
$ mkdir /data$ ceph-fuse -m 192.168.1.121:6789,192.168.1.122:6789,192.168.1.123:6789 /data$ df -hFilesystem      Size  Used Avail Use% Mounted on...ceph-fuse        47G     0   47G   0% /data
```

可以使用 mount 进行挂载：

```bash
$ mount -t ceph -o name=admin,secret=`ceph auth get-key client.admin`,rw 192.168.1.121:6789,192.168.1.122:6789,192.168.1.123:6789:/ /data
```

可能会超时：

```bash
$ mount -vvvv -t ceph -o name=admin,secret=`ceph auth get-key client.admin`,rw 192.168.1.121:6789,192.168.1.122:6789,192.168.1.123:6789:/ /data         parsing options: rw,name=admin,secret=xxxmount error 110 = Connection timed outmount error 110 = Connection timed out
```

通过如下方式解决：

```bash
$ ceph osd crush tunables hammer$ ceph osd crush reweight-all
```

删除 pool：

```bash
ceph osd pool rm test_data test_data --yes-i-really-really-mean-it
```

如出现如下错误：

```bash
$ ceph osd pool rm test_data test_data --yes-i-really-really-mean-itError EPERM: WARNING: this will *PERMANENTLY DESTROY* all data stored in pool data.  If you are *ABSOLUTELY CERTAIN* that is what you want, pass the pool name *twice*, followed by --yes-i-really-really-mean-it.
```

请添加配置：

    [mon]...mon_allow_pool_delete = true

重启服务:

```bash
systemctl restart ceph*.service ceph*.target
```

然后再进行删除。 

## RBD

使用 RBD 存储，加载模块：

```bash
$ modprobe rbd$ modinfo rbdfilename:       /lib/modules/4.4.197-1.el7.elrepo.x86_64/kernel/drivers/block/rbd.kolicense:        GPLdescription:    RADOS Block Device (RBD) driverauthor:         Jeff Garzik <jeff@garzik.org>author:         Yehuda Sadeh <yehuda@hq.newdream.net>author:         Sage Weil <sage@newdream.net>author:         Alex Elder <elder@inktank.com>srcversion:     6B2F83CD15A20F9CDD5DDF0depends:        libcephretpoline:      Yintree:         Yvermagic:       4.4.197-1.el7.elrepo.x86_64 SMP mod_unload modversionsparm:           single_major:Use a single major number for all rbd devices (default: false) (bool)
```

查看存储池：

```bash
$ ceph osd lspools7 rbd8 k8s9 test_data10 test_metadata
```

查看某个存储池中的镜像：

    rbd list k8s

查看各个存储池使用情况：

```bash
$ rados dfPOOL_NAME        USED OBJECTS CLONES COPIES MISSING_ON_PRIMARY UNFOUND DEGRADED  RD_OPS      RD   WR_OPS      WR USED COMPR UNDER COMPRk8s            48 GiB    5235      0  15705                  0       0        0 3331022 180 GiB 52592993 533 GiB        0 B         0 Brbd           192 KiB       3      0      9                  0       0        0    2100  16 MiB      522 360 MiB        0 B         0 Btest_data         0 B       0      0      0                  0       0        0       0     0 B        0     0 B        0 B         0 Btest_metadata 1.5 MiB      22      0     66                  0       0        0       0     0 B       45  13 KiB        0 B         0 Btest_rbd      192 KiB       3      0      9                  0       0        0     608 1.8 MiB      235  23 MiB        0 B         0 Btotal_objects    5263total_used       51 GiBtotal_avail      1.4 TiBtotal_space      1.5 TiB
```

创建名为 test\_rbd 的 Pool：

```bash
ceph osd pool create test_rbd 70
```

初始化 rbd pool 块：

```bash
rbd pool init test_rbd
```

创建 rbd，名为 rbd1 大小为 10GB：

```bash
rbd create test_rbd/rbd1 --size 10240# rbd create test_rbd/rbd1 --size 10240 --image-format 2 --object-size 22# rbd create test_rbd/rbd1  --size 1024 --image-feature layering
```

说明：

- 默认创建 image 时，没有任何模式也就是 --image-format 为 1。
- 如果 --image-format 为 2，则支持 RBD分层（layering），是实现 COW （Copy-On-Write）的前提。
- 如果 --object-size 24（16MB） 则设置 Object 的大小，默认值 22（4MB）

查看池中的 image：

```bash
$ rbd ls test_rbdrbd1  $ rbd info test_rbd/rbd1rbd image 'rbd1':	size 10 GiB in 2560 objects	order 22 (4 MiB objects)	snapshot_count: 0	id: 51e6e345fdee6	block_name_prefix: rbd_data.51e6e345fdee6	format: 2	features: layering, exclusive-lock, object-map, fast-diff, deep-flatten	op_features:	flags:	create_timestamp: Mon Feb  3 20:48:14 2020	access_timestamp: Mon Feb  3 20:48:14 2020	modify_timestamp: Mon Feb  3 20:48:14 2020
```

改写镜像大小：

```bash
$ rbd resize test_rbd/rbd1 --size 20480Resizing image: 100% complete...done.  $ rbd info test_rbd/rbd1rbd image 'rbd1':	size 20 GiB in 5120 objects	order 22 (4 MiB objects)	snapshot_count: 0	id: 51e6e345fdee6	block_name_prefix: rbd_data.51e6e345fdee6	format: 2	features: layering, exclusive-lock, object-map, fast-diff, deep-flatten	op_features:	flags:	create_timestamp: Mon Feb  3 20:48:14 2020	access_timestamp: Mon Feb  3 20:48:14 2020	modify_timestamp: Mon Feb  3 20:48:14 2020
```

映射到物理机：

```bash
$ rbd map test_rbd/rbd1/dev/rbd0  # 卸载# rbd unmap test_rbd/rbd1
```

如出现如下提示：

```bash
$ rbd map test_rbd/rbd1rbd: sysfs write failedRBD image feature set mismatch. You can disable features unsupported by the kernel with "rbd feature disable test_rbd/rbd1 object-map fast-diff deep-flatten".In some cases useful info is found in syslog - try "dmesg | tail".rbd: map failed: (6) No such device or address
```

表示当前系统不支持 feature。


RBD支持的特性，及具体BIT值的计算如下：

| 属性 | 功能 | BIT码 |
| --- | --- | --- |
| layering | 支持分层 | 1 |
| striping | 支持条带化v2 | 2 |
| exclusive-lock | 支持独占锁 | 4 |
| object-map | 支持对象映射（依赖 exclusive-lock ） | 8 |
| fast-diff | 快速计算差异（依赖 object-map ） | 16 |
| deep-flatten | 支持快照扁平化操作 | 32 |
| journaling | 支持记录 IO 操作（依赖独占锁） | 64 |

而实际上 Centos 7 的 3.10 内核只支持 layering… 所以我们要手动关闭一些 features，然后重新 map；如果想要一劳永逸，可以在 ceph.conf 中加入 rbd\_default\_features = 1 来设置默认 features(数值仅是 layering 对应的 bit 码所对应的整数值)。


禁用内核不支持的 feature：

```bash
$ rbd feature disable test_rbd/rbd1 object-map fast-diff deep-flatten exclusive-lock
```

如果还想一劳永逸，那么就在执行创建rbd镜像命令的服务器中，修改Ceph配置文件/etc/ceph/ceph.conf，在global section下，增加

    rbd_default_features = 1

如果还无法进行映射通过 info 指令查看是否还有其他的 featires，然后 disable 后重试即可。

```bash
$ rbd info test_rbd/rbd1rbd image 'rbd1':	size 20 GiB in 5120 objects	order 22 (4 MiB objects)	snapshot_count: 0	id: 51e6e345fdee6	block_name_prefix: rbd_data.51e6e345fdee6	format: 2	features: layering, exclusive-lock	op_features:	flags:	create_timestamp: Mon Feb  3 20:48:14 2020	access_timestamp: Mon Feb  3 20:48:14 2020	modify_timestamp: Mon Feb  3 20:48:14 2020
```

查看映射列表：

```bash
$ rbd showmappedid pool     namespace image snap device0  test_rbd           rbd1  -    /dev/rbd0
```

格式化 xfs 格式：

```bash
sudo mkfs.xfs /dev/rbd0
```

挂载磁盘：

```bash
sudo mount /dev/rbd0 /mnt  $ df -hTFilesystem     1K-blocks     Used Available Use% Mounted on.../dev/rbd0      xfs              20G   55M   20G   1% /mnt
```

当 RBD 扩容时，则需要重新执行格式化并进行挂载：(此时数据会丢失)

```bash
$ sudo umount /mnt$ sudo mkfs.xfs /dev/rbd0 -f$ rbd resize test_rbd/rbd1 --size 20480$ sudo mount /dev/rbd0 /mnt
```

进入磁盘，创建测试文件：

```bash
$ cd /mnt$ sudo dd if=/dev/zero of=/mnt/file bs=10M count=1 oflag=direct记录了1+0 的读入记录了1+0 的写出10485760字节(10 MB)已复制，0.0783636 秒，134 MB/秒$ lsfile$ df -h /mnt文件系统        容量  已用  可用 已用% 挂载点/dev/rbd0        20G   43M   20G    1% /mnt
```

删除 RBD:

```bash
sudo umount /mntsudo rbd unmap test_rbd/rbd1sudo rbd rm test_rbd/rbd1
```



## k8s

创建一个rbd镜像：

```bash
rbd create foobar -s 1024 -p k8s #在k8s pool中创建名为foobar的image，大小为1024MB
```

这时候，不要手动不要挂载！ 

### RBD用作volume

```yaml
apiVersion: v1kind: Podmetadata:  name: rbdspec:  containers:    - image: gcr.io/nginx      name: rbd-rw      volumeMounts:      - name: rbdpd        mountPath: /mnt/rbd  volumes:    - name: rbdpd      rbd:        monitors:        - '1.2.3.4:6789'        pool: k8s        image: foobar        fsType: ext4        readOnly: false        user: admin        keyring: /etc/ceph/ceph.client.admin.keyring
```

Pod启动后，可以看到文件系统由k8s做好并挂载到了容器里。我们将/etc/hosts文件拷贝到/mnt/rbd/目录去。

    kubectl exec rbd -- df -h|grep rbdkubectl exec rbd -- cp /etc/hosts /mnt/rbd/kubectl exec rbd -- cat /mnt/rbd/hosts

然后将Pod删除、重新挂载foobar image。


前面Pod要求各node上都要有keyring文件，很不方便也不安全。新的Pod我使用推荐的做法：secret（虽然也安全不到哪里）


先创建一个secret。

```yaml
apiVersion: v1kind: Secretmetadata:  name: ceph-secrettype: "kubernetes.io/rbd"  data:  key: QVFCXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX9PQ==
```

在新的Pod里Ref这个secret。

```yaml
apiVersion: v1kind: Podmetadata:  name: rbd3spec:  containers:    - image: gcr.io/nginx      name: rbd-rw      volumeMounts:      - name: rbdpd        mountPath: /mnt/rbd  volumes:    - name: rbdpd      rbd:        monitors:         - '1.2.3.4:6789'        pool: k8s        image: foobar        fsType: ext4        readOnly: false        user: admin        secretRef:          name: ceph-secret
```

再来看看前面写到image上的文件还在不在。

```bash
kubectl exec rbd3 -- cat /mnt/rbd/hosts
```



### RBD用作PV/PVC

先在k8s pool里创建一个名为pv的 image。

```bash
rbd image create pv -s 1024 -p k8s
```

再创建一个PV，使用上面创建的image pv。

```yaml
apiVersion: v1kind: PersistentVolumemetadata:  name: ceph-rbd-pvspec:  capacity:    storage: 1Gi  accessModes:    - ReadWriteOnce  rbd:    monitors:      - '1.2.3.4:6789'    pool: k8s    image: pv    user: admin    secretRef:      name: ceph-secret    fsType: ext4    readOnly: false  persistentVolumeReclaimPolicy: Recycle
```

看下现在pv的状态，还是 Available。

```bash
kubectl get pv|grep rbdceph-rbd-pv                                1Gi        RWO            Recycle          Available
```

创建一个PVC，要求一块1G的存储。

```yaml
apiVersion: v1kind: PersistentVolumeClaimmetadata:  name: ceph-rbd-pv-claimspec:  accessModes:    - ReadWriteOnce  resources:    requests:      storage: 1Gi
```

因为上面已经创建满足要求的PV了，可以看到pvc和pv的状态都已经是Bound了。

```bash
# kubectl get pvc|grep rbdceph-rbd-pv-claim   Bound     pvc-bdaf8358-5dc6-11e8-a37d-ecf4bbdeea94   1Gi        RWO            fast           10s# kubectl get pv|grep ceph-rbd-pvceph-rbd-pv     1Gi    RWO    Recycle    Bound     12m
```



### RBD用作storage class

先创建一个SC。

```yaml
kind: StorageClassapiVersion: storage.k8s.io/v1metadata:  name: fastprovisioner: kubernetes.io/rbdparameters:  monitors: 172.25.60.3:6789  adminId: admin  adminSecretName: ceph-secret  adminSecretNamespace: resource-quota  pool: k8s  userId: admin  userSecretName: ceph-secret  fsType: ext4  imageFormat: "2"  imageFeatures: "layering"
```

pvc中需要指定其storageClassName为上面创建的sc的name

```yaml
kind: PersistentVolumeClaimapiVersion: v1metadata:  name: rbd-pvc-pod-pvcspec:  accessModes:    - ReadWriteOnce  volumeMode: Filesystem  resources:    requests:      storage: 8Gi  storageClassName: fast
```

RBD只支持 ReadWriteOnce 和 ReadOnlyAll，不支持ReadWriteAll。注意这两者的区别点是，不同nodes之间是否可以同时挂载。同一个node上，即使是ReadWriteOnce，也可以同时挂载到2个容器上的。

```yaml
apiVersion: v1kind: Podmetadata:  labels:    test: rbd-pvc-pod  name: ceph-rbd-sc-pod2spec:  containers:  - name: ceph-rbd-sc-nginx    image: gcr.io/nginx    volumeMounts:    - name: ceph-rbd-vol1      mountPath: /mnt/ceph-rbd-pvc/nginx      readOnly: false  volumes:  - name: ceph-rbd-vol1    persistentVolumeClaim:      claimName: rbd-pvc-pod-pvc
```

如果是多副本的应用怎么办呢？可以用StatefulSet。

```yaml
apiVersion: apps/v1kind: StatefulSetmetadata:  name: nginxspec:  selector:    matchLabels:      app: nginx  serviceName: "nginx"  replicas: 3  template:    metadata:      labels:        app: nginx    spec:      terminationGracePeriodSeconds: 10      containers:      - name: nginx        image: gcr.io/nginx        volumeMounts:        - name: www          mountPath: /usr/share/nginx/html  volumeClaimTemplates:  - metadata:      name: www    spec:      accessModes: [ "ReadWriteOnce" ]      storageClassName: "fast"      resources:        requests:          storage: 8Gi
```

此时去看PVC，可以看到创建了3个PVC。

```bash
# kubectl get pvc|grep wwwwww-nginx-0         Bound     pvc-05f10f64-58df-11e8-8bd4-ecf4bbdeea94   8Gi        RWO            fast           6dwww-nginx-1         Bound     pvc-0e1768f3-58df-11e8-8bd4-ecf4bbdeea94   8Gi        RWO            fast           6dwww-nginx-2         Bound     pvc-17347e8e-58df-11e8-8bd4-ecf4bbdeea94   8Gi        RWO            fast           6d# kubectl get pv|grep wwwpvc-05f10f64-58df-11e8-8bd4-ecf4bbdeea94   8Gi        RWO            Delete           Bound       dex/www-nginx-0                     fast                     6dpvc-0e1768f3-58df-11e8-8bd4-ecf4bbdeea94   8Gi        RWO            Delete           Bound       dex/www-nginx-1                     fast                     6dpvc-17347e8e-58df-11e8-8bd4-ecf4bbdeea94   8Gi        RWO            Delete           Bound       dex/www-nginx-2                     fast                     6d
```

但注意不要用Deployment。因为，如果Deployment的副本数是1，那么还是可以用的，跟Pod一致；但如果副本数 >1 ，此时创建deployment后会发现，只启动了1个Pod，其他Pod都在ContainerCreating状态。过一段时间describe pod可以看到，等volume等很久都没等到。 

# 参考文章

- [kubernetes笔记: Ceph RBD]()
- \[K8S and Eph RBD Integration - Examples of Multi-master and Master-slave Databases]\(
