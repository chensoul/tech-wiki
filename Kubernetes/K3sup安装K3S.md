k3sup是一个支持在PC、虚拟机、ARM设备上安装k3s的工具，官方网站：<https://k3sup.dev/>


![](../assets/e1ed7cbe845094deeae9fb45ca435fbc/00831rSTgy1gcz8rqm769j33ie0u0dol.jpg) 

# 配置SSH

k3sup是基于ssh，所以必须先生成ssh私钥并且配置无密码登陆。


修改sshd配置：

```bash
sed -i '/PasswordAuthentication/s/^/#/'  /etc/ssh/sshd_config
sed -i 's/^[ ]*StrictHostKeyChecking.*/StrictHostKeyChecking no/g' /etc/ssh/ssh_config
#禁用sshd服务的UseDNS、GSSAPIAuthentication两项特性
sed -i -e 's/^#UseDNS.*$/UseDNS no/' /etc/ssh/sshd_config
sed -i -e 's/^GSSAPIAuthentication.*$/GSSAPIAuthentication no/' /etc/ssh/sshd_config
systemctl restart sshd
```

生成ssh私钥：

```bash
[ ! -d ~/.ssh ] && ( mkdir ~/.ssh )
[ ! -f ~/.ssh/id_rsa.pub ] && (yes|ssh-keygen -f ~/.ssh/id_rsa -t rsa -N "")
( chmod 600 ~/.ssh/id_rsa.pub ) && cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

配置无密码登陆：

```bash
ssh-copy-id 192.168.56.141 #192.168.56.141是我虚拟机的ip
```

如果ssh-agent没启动，则启动：

```bash
eval `ssh-agent`
ssh-add ~/.ssh/id_rsa
```



# 安装k3sup

在线安装：

```bash
curl -sLS https://get.k3sup.dev | sh
```

离线安装，下载地址：<https://github.com/alexellis/k3sup/releases>

```bash
wget https://github.com/alexellis/k3sup/releases/download/0.9.2/k3sup
mv k3sup /usr/local/bin/
chmod +x /usr/local/bin/k3sup
```



# 创建k3s集群

启动一个k3s：

```bash
export SERVER_IP=144.34.194.100
export USER=chenzj
k3sup install --ip $SERVER_IP --user $USER --ssh-port 29219 --k3s-version v1.17.3+k3s1 --k3s-extra-args '--docker --no-deploy=traefik'
```

安装日志：

```bash
Running: k3sup install
Public IP: 144.34.194.100
ssh -i /home/chenzj/.ssh/id_rsa -p 29219 chenzj@144.34.194.100
ssh: curl -sLS https://get.k3s.io | INSTALL_K3S_EXEC='server  --tls-san 144.34.194.100 --docker' INSTALL_K3S_VERSION='v1.17.3+k3s1' sh -
[INFO]  Using v1.17.3+k3s1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.17.3+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.17.3+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
chcon: can't apply partial context to unlabeled file ‘/usr/local/bin/k3s’
[INFO]  Skipping /usr/local/bin/kubectl symlink to k3s, already exists
[INFO]  Skipping /usr/local/bin/crictl symlink to k3s, already exists
[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, command exists in PATH at /usr/bin/ctr
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
[INFO]  systemd: Starting k3s
```

运行成功之后，会将kubeconfig保存到 ~/kubeconfig，可以将该文件拷贝到kubectl的配置文件目录下：

    mkdir ~/.kube
    cp kubeconfig ~/.kube/config

或者是设置KUBECONFIG变量：

    export KUBECONFIG=/home/chenzj/kubeconfig

参考更多参数说明：

```bash
$ k3sup install --help
Install k3s on a server via SSH.
Usage:
  k3sup install [flags]
Examples:
  k3sup install --ip 192.168.0.100 --user root
Flags:
      --cluster                 Form a dqlite cluster
      --context string          Set the name of the kubeconfig context. (default "default")
  -h, --help                    help for install
      --ip ip                   Public IP of node (default 127.0.0.1)
      --ipsec                   Enforces and/or activates optional extra argument for k3s: flannel-backend option: ipsec
      --k3s-extra-args string   Optional extra arguments to pass to k3s installer, wrapped in quotes (e.g. --k3s-extra-args '--no-deploy servicelb')
      --k3s-version string      Optional version to install, pinned at a default (default "v1.17.2+k3s1")
      --local                   Perform a local install without using ssh
      --local-path string       Local path to save the kubeconfig file (default "kubeconfig")
      --merge                   Merge the config with existing kubeconfig if it already exists.
                                Provide the --local-path flag with --merge if a kubeconfig already exists in some other directory
      --no-extras               Disable "servicelb" and "traefik"
      --skip-install            Skip the k3s installer
      --ssh-key string          The ssh key to use for remote login (default "~/.ssh/id_rsa")
      --ssh-port int            The port on which to connect for ssh (default 22)
      --sudo                    Use sudo for installation. e.g. set to false when using the root user and no sudo is available. (default true)
      --user string             Username for SSH login (default "root")
```

安装日志：

```bash
Running: k3sup install
Public IP: 192.168.56.141
ssh -i /root/.ssh/id_rsa -p 22 root@192.168.56.141
ssh: curl -sLS https://get.k3s.io | INSTALL_K3S_EXEC='server --tls-san 192.168.56.141 ' INSTALL_K3S_VERSION='v1.17.2+k3s1' sh -
[INFO]  Using v1.17.2+k3s1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.17.2+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.17.2+k3s1/k3s
```

从安装日志，可以看出来k3sup其实就是对k3s进行了一层封装。启动K3s的命令如下：

```bash
curl -sLS https://get.k3s.io | INSTALL_K3S_EXEC='server --tls-san 192.168.56.141 ' sh -
```

可以看到:

- 使用的是container，没有使用docker



- 这里指定了--tls-san参数，给TLS SAN添加了一个IP。查看节点信息进行验证：




```bash
NAME          STATUS   ROLES    AGE     VERSION        INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIMEk3s-node001   Ready    master   2m38s   v1.17.3+k3s1   10.0.2.15     <none>        CentOS Linux 7 (Core)   3.10.0-1062.4.3.el7.x86_64   containerd://1.3.3-k3s1
```

查看集群状态：

```bash
$ kubectl get svc -n kube-systemNAME                 TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGEkube-dns             ClusterIP      10.43.0.10      <none>           53/UDP,53/TCP,9153/TCP       14hmetrics-server       ClusterIP      10.43.194.172   <none>           443/TCP                      14htraefik-prometheus   ClusterIP      10.43.31.237    <none>           9100/TCP                     14htraefik              LoadBalancer   10.43.53.194    144.34.194.100   80:32572/TCP,443:31258/TCP   14h
```

- 可以看到traefik的EXTERNAL-IP是外网IP地址。

添加一个work节点到集群：

```bash
export SERVER_IP=192.168.56.141export USER=rootexport AGENT_IP=192.168.56.142k3sup join --ip $AGENT_IP --server-ip $SERVER_IP --user $USER
```



# 创建一个HA集群

创建一个集群：

```bash
export SERVER_IP=192.168.56.141export USER=rootk3sup install \  --ip $SERVER_IP \  --user $USER \  --cluster
```

再创建一个master节点：

```bash
export USER=rootexport SERVER_IP=192.168.56.141export NEXT_SERVER_IP=192.168.56.143k3sup join \  --ip $NEXT_SERVER_IP \  --user $USER \  --server-user $USER \  --server-ip $SERVER_IP \  --server
```

查看集群节点：

```bash
kubectl get node
```



# arkade安装应用



## 下载

```bash
k3sup app install nginx-ingress
```

提示下载arkade：

```bash
curl -sSL https://dl.get-arkade.dev/ | sudo sh
#离线安装
wget https://github.com/alexellis/arkade/releases/download/0.2.0/arkade
cp arkade /usr/local/bin/
chmod +x /usr/local/bin/arkade
```



## 使用说明

查看帮助：

```postgresql
arkade install --help
```

安装应用：

```bash
arkade install cert-manager
# 制定命名空间
arkade install postgresql --helm3 -n postgresql
arkade install nginx-ingress --help
```

更新应用：

    arkade update

查看应用：

    arkade info postgresql

查看能够安装的应用有哪些：

```bash
$ arkade install
You can install:
 - openfaas
 - nginx-ingress
 - cert-manager
 - openfaas-ingress
 - inlets-operator
 - metrics-server
 - chart
 - linkerd
 - cron-connector
 - kafka-connector
 - minio
 - postgresql
 - kubernetes-dashboard
 - istio
 - crossplane
 - mongodb
 - docker-registry
 - docker-registry-ingress
 - traefik2
 - grafana
Run arkade install NAME --help to see configuration options.
```



## 安装应用



### 安装nginx-ingress

    arkade install nginx-ingress

查看安装日志：

```bash
Using kubeconfig: /home/chenzj/kubeconfig
Using helm3
Client: x86_64, Linux
2020/03/20 09:20:41 User dir established as: /home/chenzj/.arkade/
https://get.helm.sh/helm-v3.1.1-linux-amd64.tar.gz
/home/chenzj/.arkade/bin/helm3/linux-amd64 linux-amd64/
/home/chenzj/.arkade/bin/helm3/README.md linux-amd64/README.md
/home/chenzj/.arkade/bin/helm3/LICENSE linux-amd64/LICENSE
/home/chenzj/.arkade/bin/helm3/helm linux-amd64/helm
2020/03/20 09:20:43 extracted tarball into /home/chenzj/.arkade/bin/helm3: 3 files, 0 dirs (1.059805161s)
"stable" has been added to your repositories
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈
Node architecture: "amd64"
Chart path:  /tmp/charts
VALUES values.yaml
Command: /root/.arkade/bin/helm3/helm [upgrade --install nginx-ingress stable/nginx-ingress --namespace default --values /tmp/charts/nginx-ingress/values.yaml --set defaultBackend.enabled=false]
Release "nginx-ingress" has been upgraded. Happy Helming!
```

可以看到arkade自动做了一下几件事：

- 下载helm3



- 查找合适的chart仓库，并添加



- 使用helm3安装应用，安装脚本：

```bash
Command: /root/.arkade/bin/helm3/helm [upgrade --install nginx-ingress stable/nginx-ingress --namespace default --values /tmp/charts/nginx-ingress/values.yaml --set defaultBackend.enabled=false]
```

稍等几分钟，查看服务：

```bash
$ kubectl get svc nginx-ingress-controllerNAME                       TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGEnginx-ingress-controller   LoadBalancer   10.43.155.95   <pending>     80:31992/TCP,443:31153/TCP   48m
```

如果使用的是共有云，则EXTERNAL-IP会是一个公网IP，如果一直显示pending状态，可以pod信息：

```bash
$ kubectl describe pod/svclb-nginx-ingress-controller-h96vg  Warning  FailedScheduling  <unknown>  default-scheduler  0/1 nodes are available: 1 node(s) didn't have free ports for the requested pod ports.
```

提示没有找到可用的端口，这是因为安装k3s时候，默认安装了traefix，而traefik已经占用了80和443端口：

```bash
get svc -n kube-system
NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
kube-dns             ClusterIP      10.43.0.10      <none>           53/UDP,53/TCP,9153/TCP       15h
metrics-server       ClusterIP      10.43.194.172   <none>           443/TCP                      15h
traefik-prometheus   ClusterIP      10.43.31.237    <none>           9100/TCP                     14h
traefik              LoadBalancer   10.43.53.194    144.34.194.100   80:32572/TCP,443:31258/TCP   14h
```

所以，需要禁用traefik，修改k3s启动脚本 /etc/systemd/system/k3s.service：添加 --no-deploy=traefik

```bash
ExecStart=/usr/local/bin/k3s \
    server \
        '--tls-san' \
        '144.34.194.100' \
        '--docker' \
        '--no-deploy=traefik' \
```

然后，重启k3s：

    systemctl daemon-reload
    systemctl restart k3s

如果上面没有生效，则卸载treafik，或者如果可能的话，重装k3s。


最好，再次查看svc：

```bash
$ kubectl get svc nginx-ingress-controller
NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
nginx-ingress-controller   LoadBalancer   10.43.188.86   144.34.194.100   80:30079/TCP,443:30806/TCP   7s
```



### 安装TLS的docker-registry

详细步骤可以参考：<https://blog.alexellis.io/get-a-tls-enabled-docker-registry-in-5-minutes/>


![](../assets/e1ed7cbe845094deeae9fb45ca435fbc/00831rSTgy1gd0jnwxu6uj31ss0u0kfj.jpg)


先设置kubeconfig：

    cat /etc/rancher/k3s/k3s.yaml > ~/.kube/config

安装：

```bash
arkade install nginx-ingress
arkade install cert-manager
arkade install docker-registry
```

查看cert-manager是否安装成功：

```bash
kubectl get all -n cert-manager
```

下面是将svc/docker-registry端口转发进行访问：

```bash
kubectl port-forward svc/docker-registry --address 0.0.0.0 5000 & 
export PASSWORD=h1p580SuX14352N9ZLje
export IP="144.34.194.100"
docker login $IP:5000 --username admin --password $PASSWORD
docker tag alpine:3.11 $IP:5000/alpine:3.11
docker push $IP:5000/alpine:3.11
```

安装docker-registry-ingress配置TLS：

    arkade install docker-registry-ingress \
      --email junecloud@163.com \
      --domain vps.javachen.space

- 注意：vps.javachen.space是做了DNS解析到了144.34.194.100

查看证书是否生成：

```bash
$ kubectl get cert
NAME              READY   SECRET            AGE
docker-registry   True    docker-registry   12m
```

这时候可以通过域名访问：

```bash
docker login vps.javachen.space
docker pull alpine:3.11
docker tag alpine:3.11 vps.javachen.space/alpine:3.11
docker push vps.javachen.space/alpine:3.11
```

浏览器访问：<https://vps.javachen.space/v2/>，输入用户名和秘密，可以查看生成的证书


![](../assets/e1ed7cbe845094deeae9fb45ca435fbc/00831rSTgy1gd07jd3prkj31lc0rk46k.jpg)
