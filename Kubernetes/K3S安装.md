本文在我购买的虚拟机主机上安装k3s，虚拟主机有公网IP并且做了DNS域名解析。 

# 安装Docker

```bash
curl -fsSL get.docker.com | sh
usermod -aG docker `whoami`
systemctl enable docker && systemctl start docker
```



# 离线安装

如果服务器在国内，需要事先准备下载好 k3s 的可执行文件和 airgap 的镜像包。 从 [k3s Release](https://github.com/rancher/k3s/releases) 下载对应文件，本例为 `k3s` 和 `k3s-airgap-images-amd64.tar`

```bash
mkdir -p /var/lib/rancher/k3s/agent/images/
wget https://github.com/rancher/k3s/releases/download/v1.17.3%2Bk3s1/k3s-airgap-images-amd64.tar -O /var/lib/rancher/k3s/agent/images/k3s-airgap-images-amd64.tar
wget https://github.com/rancher/k3s/releases/download/v1.17.3%2Bk3s1/k3s -O /usr/local/bin/k3s
chmod +x /usr/local/bin/k3s
```

然后，在master节点：

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--docker" sh -
```

该脚本会帮你自动安装k3s、kubectl等等并且启动k3s服务，如果检测到已经安装，则会跳过。 

# 在线安装

```bash
#默认安装
curl -sfL https://get.k3s.io | sh -
#使用docker
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--docker" sh -
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--docker --no-deploy=traefik" sh -
```

- k3s默认使用contrainer，可以设置使用docker
- k3s默认安装了traefik、flannel

安装日志：

```bash
[INFO]  Finding latest release
[INFO]  Using v1.17.3+k3s1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.17.3+k3s1/sha256sum-amd64.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.17.3+k3s1/k3s
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
chcon: can't apply partial context to unlabeled file ‘/usr/local/bin/k3s’
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Creating /usr/local/bin/crictl symlink to k3s
[INFO]  Skipping /usr/local/bin/ctr symlink to k3s, command exists in PATH at /usr/bin/ctr
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink from /etc/systemd/system/multi-user.target.wants/k3s.service to /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
```

安装成功之后，一个kubeconfig文件保存在/etc/rancher/k3s/k3s.yaml。我们可以将其保存到kubectl的目录下面。

```bash
cat /etc/rancher/k3s/k3s.yaml > ~/.kube/config
```



# 检查状态

查看kubectl是否安装：

```bash
$ which kubectl
$ kubectl version
$ ll /usr/local/bin/kubectl
lrwxrwxrwx. 1 root root 3 Mar 18 04:00 /usr/local/bin/kubectl -> k3s
```

查看k3s版本和服务运行状态：

```bash
k3s --version
systemctl status k3s
```

查看节点状态：

```bash
$ kubectl get nodes
NAME          STATUS   ROLES    AGE   VERSION
vps.javachen.space   Ready    master   31m   v1.17.3+k3s1
$ kubectl get node -o wide
AME                 STATUS   ROLES    AGE     VERSION        INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
vps.javachen.space   Ready    master   4m35s   v1.17.3+k3s1   144.34.194.100   <none>        CentOS Linux 7 (Core)   4.10.4-1.el7.elrepo.x86_64   docker://19.3.8
```

- 可以看到 INTERNAL-IP 是我VPS的公网IP

查看命名空间：

```bash
$ kubectl get ns
NAME              STATUS   AGE
default           Active   31m
kube-system       Active   31m
kube-public       Active   31m
kube-node-lease   Active   31m
```

查看运行状态：

```bash
$ kubectl get all -n kube-system
NAME                                          READY   STATUS      RESTARTS   AGE
pod/local-path-provisioner-58fb86bdfd-lmprq   1/1     Running     0          30s
pod/metrics-server-6d684c7b5-lrvwr            1/1     Running     0          30s
pod/helm-install-traefik-dvrn8                0/1     Completed   1          31s
pod/svclb-traefik-lh8g7                       2/2     Running     0          24s
pod/coredns-d798c9dd-2qlrs                    1/1     Running     0          30s
pod/traefik-6787cddb4b-64v7b                  1/1     Running     0          24s
NAME                         TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
service/kube-dns             ClusterIP      10.43.0.10      <none>           53/UDP,53/TCP,9153/TCP       44s
service/metrics-server       ClusterIP      10.43.156.171   <none>           443/TCP                      44s
service/traefik-prometheus   ClusterIP      10.43.60.39     <none>           9100/TCP                     24s
service/traefik              LoadBalancer   10.43.138.254   144.34.194.100   80:30280/TCP,443:30058/TCP   24s
NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/svclb-traefik   1         1         1       1            1           <none>          24s
NAME                                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/local-path-provisioner   1/1     1            1           44s
deployment.apps/metrics-server           1/1     1            1           44s
deployment.apps/coredns                  1/1     1            1           44s
deployment.apps/traefik                  1/1     1            1           24s
NAME                                                DESIRED   CURRENT   READY   AGE
replicaset.apps/local-path-provisioner-58fb86bdfd   1         1         1       30s
replicaset.apps/metrics-server-6d684c7b5            1         1         1       30s
replicaset.apps/coredns-d798c9dd                    1         1         1       30s
replicaset.apps/traefik-6787cddb4b                  1         1         1       24s
NAME                             COMPLETIONS   DURATION   AGE
job.batch/helm-install-traefik   1/1           7s         43s
```

- 可以看到 service/traefik 类型为 LoadBalancer，EXTERNAL-IP 为公网IP 144.34.194.100 。 

# 配置参数

查看帮助文档：

```bash
$ k3s -h
NAME:
   k3s - Kubernetes, but small and simple
USAGE:
   k3s [global options] command [command options] [arguments...]
VERSION:
   v1.17.3+k3s1 (5b17a175)
COMMANDS:
   server        Run management server
   agent         Run node agent
   kubectl       Run kubectl
   crictl        Run crictl
   ctr           Run ctr
   check-config  Run config check
   help, h       Shows a list of commands or help for one command
GLOBAL OPTIONS:
   --debug        Turn on debug logs [$K3S_DEBUG]
   --help, -h     show help
   --version, -v  print the version
```

查看安装k3s可以配置的参数：

    k3s server -h

当k3s安装成功之后，我们也可以修改/etc/systemd/system/k3s.service中的启动参数

```bash
[Unit]
Description=Lightweight Kubernetes
Documentation=https://k3s.io
Wants=network-online.target
[Install]
WantedBy=multi-user.target
[Service]
Type=notify
EnvironmentFile=/etc/systemd/system/k3s.service.env
KillMode=process
Delegate=yes
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
TimeoutStartSec=0
Restart=always
RestartSec=5s
ExecStartPre=-/sbin/modprobe br_netfilter
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/k3s \
    server \
	'--docker' \
```



# 集成calico

在k3s启动参数中添加 "--docker --flannel-backend=none" 进行安装。

```bash
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--docker --flannel-backend=none" sh -
```

安装calico：

```bash
curl https://docs.projectcalico.org/v3.13/manifests/calico.yaml -O
POD_CIDR=$(kubectl get nodes -o jsonpath='{.items[*].spec.podCIDR}')
sed -i -e "s?192.168.0.0/16?$POD_CIDR?g" calico.yaml
kubectl apply -f calico.yaml
```

查看calico是否安装成功：

    kubectl get all -n kube-system|grep calico



# 添加Worker节点

在master节点查看\_K3S\_TOKEN\_：

```bash
$ cat /var/lib/rancher/k3s/server/node-token
K106a6165c91f88a8a167f87453a144d7f2d70c4adfe22cef8505e4f24bcf98e998::server:a0dc42b328b48b9853985babe00a317c
```

在worker节点安装：

```bash
k3s_url="http://144.34.194.100:6443"
k3s_token="K106a6165c91f88a8a167f87453a144d7f2d70c4adfe22cef8505e4f24bcf98e998::server:a0dc42b328b48b9853985babe00a317c"
curl -sfL https://get.k3s.io | K3S_URL=${k3s_url} K3S_TOKEN=${k3s_token} sh -
```

查看集群配置：

```bash
kubectl config get-clusters 
kubectl cluster-info
```

查看节点：

```bash
kubectl get nodes
kubectl get endpoints -n kube-system
```



# 安装 Nginx-Ingress

```yaml
cat << EOF | kubectl apply -f -
apiVersion: helm.cattle.io/v1
kind: HelmChart
metadata:
  name: nginx-ingress
  namespace: kube-system
spec:
  chart: stable/nginx-ingress
  valuesContent: |-
    controller:
      kind: DaemonSet
      daemonset:
        useHostPort: true
      service:
        type: ClusterIP
EOF
```



# 安装 kubernetes dashboard

> 注意：以下安装的是 kubernetes dashboard 2.0 版本

```bash
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/recommended.yaml
```

建立一个能够 Cluster Role view 的账户：

```bash
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

获取到账户的 Token：

```bash
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

为了能进行外网访问， 有三种方式：


1、我们可以使代理：

```bash
#配置端口转发
ssh -L8001:localhost:8001 root@<ip-adress of the master>
kubectl proxy --address='0.0.0.0' --port=8001 --accept-hosts='.*'
```

- 需要指定 `--accept-hosts` 选项，否则浏览器访问 dashboard 页面时提示 “Unauthorized”；

这样就可以用本地地址访问：<http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/>


2、使用端口转发

```bash
kubectl port-forward service/kubernetes-dashboard -n kubernetes-dashboard 8443:8443
```

> 注意：暂时没有测试成功！！

3、创建 Ingress


使用ingress配置TLS，域名为 dashboard.javachen.space ，并且DNS解析到了 144.34.194.100

```bash
cat << EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubernetes-dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: dashboard.javachen.space
    http:
      paths:
        - backend:
            serviceName: kubernetes-dashboard
            servicePort: 443
  tls:
  - hosts:
    - dashboard.javachen.space
    secretName: kubernetes-dashboard-tls
EOF
```

查看状态： 

# 卸载

如果使用 *<https://get.k3s.io>* 脚本的安装k3s，则会在安装过程中生成一个卸载脚本，该脚本将在\_`/usr/local/bin/k3s-uninstall.sh`*（或*`k3s-agent-uninstall.sh`\_）处的服务器节点上创建。

```bash
/usr/local/bin/k3s-uninstall.sh
```
