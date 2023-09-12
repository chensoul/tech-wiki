脚本安装：

```bash
wget -q -O - https://raw.githubusercontent.com/rancher/k3d/master/install.sh | bash
curl -s https://raw.githubusercontent.com/rancher/k3d/master/install.sh | bash
```

homebrew安装：

    brew install k3d

设置环境变量：

```bash
export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"
```

本地安装kubectl:

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg 
  http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
yum install kubectl -y
ln /usr/bin/kubectl /usr/local/bin/kubectl
```

查看docker容器内集群信息：

```bash
kubectl cluster-info
```

查看帮助文档：

```bash
$ k3d -h
NAME:
   k3d - Run k3s in Docker!
USAGE:
   k3d [global options] command [command options] [arguments...]
VERSION:
   v1.7.0
COMMANDS:
     check-tools, ct   Check if docker is running
     shell             Start a subshell for a cluster
     create, c         Create a single- or multi-node k3s cluster in docker containers
     add-node          [EXPERIMENTAL] Add nodes to an existing k3d/k3s cluster (k3d by default)
     delete, d, del    Delete cluster
     stop              Stop cluster
     start             Start a stopped cluster
     list, ls, l       List all clusters
     get-kubeconfig    Get kubeconfig location for cluster
     import-images, i  Import a comma- or space-separated list of container images from your local docker daemon into the cluster
     version           print k3d and k3s version
     help, h           Shows a list of commands or help for one command
GLOBAL OPTIONS:
   --verbose      Enable verbose output
   --timestamp    Enable timestamps in logs messages
   --help, -h     show help
   --version, -v  print the version
```



# 创建k3s集群

创建一个集群，设置参数不使用traefik：

```bash
k3d create --server-arg "--no-deploy=traefik"
```

下载arkade

```bash
curl -sSL https://dl.get-arkade.dev | sudo sh
```

接下来可以通过arkade安装应用：

```bash
arkade install --help
```

删除默认集群：

```bash
k3d delete
```



# 暴露服务

参考 [https://github.com/rancher/k3d/blob/master/docs/examples.md]() 

## Ingress

1、创建集群：

    k3d create --publish 8081:80 --workers 2

2、设置环境变量：

```bash
export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"
```

安装kubectl，参考上文。查看集群信息：

```bash
$ kubectl cluster-info
Kubernetes master is running at https://localhost:6443
CoreDNS is running at https://localhost:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Met
```

查看节点信息：

```bash
$ kubectl get node -o wide
NAME                       STATUS   ROLES    AGE   VERSION        INTERNAL-IP   EXTERNAL-IP   OS-IMAGE   KERNEL-VERSION               CONTAINER-RUNTIME
k3d-k3s-default-worker-1   Ready    <none>   10m   v1.17.3+k3s1   172.19.0.4    <none>        Unknown    4.10.4-1.el7.elrepo.x86_64   containerd://1.3.3-k3s1
k3d-k3s-default-worker-0   Ready    <none>   10m   v1.17.3+k3s1   172.19.0.3    <none>        Unknown    4.10.4-1.el7.elrepo.x86_64   containerd://1.3.3-k3s1
k3d-k3s-default-server     Ready    master   10m   v1.17.3+k3s1   172.19.0.2    <none>        Unknown    4.10.4-1.el7.elrepo.x86_64   containerd://1.3.3-k3s1
```

3、创建nginx deployment：

```bash
kubectl create deployment nginx --image=nginx
```

4、创建一个clueterip服务：

```bash
kubectl create service clusterip nginx --tcp=80:80
```

5、创建一个Ingress，注意：k3s默认安装了treafik ingress

```bash
cat << EOF | kubectl apply -f -
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx
  annotations:
    ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /
        backend:
          serviceName: nginx
          servicePort: 80
EOF
```

查看状态：

```bash
kubectl get pod,svc,ing
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-86c57db685-vsmlq   1/1     Running   0          3m44s
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP   8m45s
service/nginx        ClusterIP   10.43.71.250   <none>        80/TCP    3m38s
NAME                       HOSTS   ADDRESS      PORTS   AGE
ingress.extensions/nginx   *       172.19.0.3   80      119s
```

可以看到ingress的地址为 172.19.0.3，是docker容器的IP，k3d创建k3s集群时，通过 `--publish 8081:80` 将容器的80端口暴露到宿主机的8081了，所以可以本地访问8081


6、本地访问：

    curl localhost:8081/

通过浏览器访问：<http://144.34.194.100:8081/> 

## NodePort

1、创建集群：


将k3d-k3s-default-worker-0节点上的30080端口暴露到宿主机的8082端口

```bash
k3d create --publish 8082:30080@k3d-k3s-default-worker-0 --workers 2
```

2、3步骤和上面一样


4、创建Service

```bash
cat << EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - name: 80-80
    nodePort: 30080
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
EOF
```

5、本地访问：

```bash
curl localhost:8082/
```
