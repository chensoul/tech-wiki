首先安装Gitea，再安装Drone 

# 安装Gitea



## 创建证书

参考 [使用Cert Manager配置Let’s Encrypt证书](/2019/11/04/using-cert-manager-with-nginx-ingress/) ，先要创建一个ClusterIssuer：cert-manager-webhook-dnspod-cluster-issuer


因为证书是有命名空间的，所以需要在gitea命名空间创建证书：

```bash
kubectl create namespace gitea
cat << EOF | kubectl create -f -   
apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: gitea-test-wesine-com-cn-cert
  namespace: gitea
spec:
  secretName: gitea-test-wesine-com-cn-cert
  renewBefore: 240h
  dnsNames:
  - "gitea.javachen.com"
  issuerRef:
    name: cert-manager-webhook-dnspod-cluster-issuer
    kind: ClusterIssuer
EOF
```



## 安装

下载chart代码：

    git clone https://github.com/javachen/charts
    cd charts

创建gitea-values.yaml

```bash
cat <<EOF > gitea-values.yaml
image:
  repository: gitea/gitea:1.11.4
ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: 100m
  hosts:
    - gitea.javachen.com
  tls:
    - secretName: gitea-test-wesine-com-cn-cert
      hosts:
        - gitea.javachen.com
persistence:
  enabled: true
  storageClass: "ceph-rbd"
  accessMode: ReadWriteOnce
  size: 10Gi
EOF
```

使用helm3安装：

    helm install gitea -n gitea -f gitea-values.yaml ./gitea

参考 [Exposing TCP and UDP services](https://kubernetes.github.io/ingress-nginx/user-guide/exposing-tcp-udp-services/?spm=a2c4e.10696291.0.0.a27619a4RLzFYg#exposing-tcp-and-udp-services) ，让Ingress暴露一个ssh端口。


创建配置文件更新tcp-services：

```yaml
cat << EOF | kubectl apply -f -   
apiVersion: v1
kind: ConfigMap
metadata:
  name: tcp-services
  namespace: ingress-nginx
data:
  # namespace/service:protocol
  3022: "gitea/gitea:ssh"
EOF
```



## 查看状态

```bash
kubectl get all -n gitea
```



## 测试

访问 <https://gitea.javachen.com> ，填入数据库：


![](../assets/fd7a3cec1593ce540f8d0d03ce68afa7/006y8mN6gy1g8iiesm31hj31da0s6gpd.jpg)


**填入域名，SSH端口号填上面配置文件中的3022**


![](../assets/fd7a3cec1593ce540f8d0d03ce68afa7/006y8mN6ly1g8pouibtasj311u0psq5z.jpg)


修改服务器和第三方设置：


![](../assets/fd7a3cec1593ce540f8d0d03ce68afa7/006y8mN6ly1g8iih58nsgj31d00nqad3.jpg)


再次登录，注册账号 chenzj，登陆后在gitea中配置本地SSH密钥，然后创建一个test项目，本地通过ssh下载：

```bash
git clone ssh://git@gitea.javachen.space:3022/chenzj/test.git
```

可以看到不需要使用密码。 

## 卸载

```bash
helm del gitea  -n gitea
kubectl delete certificates,secret,pvc --all -n gitea
```



# 安装Drone



## 创建证书

参考 [使用Cert Manager配置Let’s Encrypt证书](/2019/11/04/using-cert-manager-with-nginx-ingress/)

```bash
kubectl create namespace dronecat << EOF | kubectl create -f -   apiVersion: cert-manager.io/v1alpha2kind: Certificatemetadata:  name: drone-test-wesine-com-cn-cert  namespace: dronespec:  secretName: drone-test-wesine-com-cn-cert  renewBefore: 240h  dnsNames:  - "drone.javachen.com"  issuerRef:    name: cert-manager-webhook-dnspod-cluster-issuer    kind: ClusterIssuerEOF
```



## 集成Gitea安装Drone

在gitea上创建应用，设置重定向地址：[https://drone.javachen.com/ ](https://drone.wesine.com.cn/login)hook，得到 clientID 和 clientSecretValue


![](../assets/fd7a3cec1593ce540f8d0d03ce68afa7/006y8mN6ly1g8if11xslij312y0sumzm.jpg)


用ingress的方式，不能走ssh协议，所以必须用http的方式，但是该方式是需要用户名和密码的。创建gitea用户drone，并将其加入到gitea项目中。


创建secret：

```bash
cat << EOF | kubectl create -f -  apiVersion: v1kind: Secretmetadata:  name: drone-gitea-login-secrets  namespace: dronetype: Opaquedata:  DRONE_GIT_USERNAME: `echo -n drone | base64`  DRONE_GIT_PASSWORD: `echo -n 123456 | base64`EOF
```

创建drone-gitea-values.yaml

```yaml
cat <<EOF > drone-gitea-values.yamlimages:	server:		tag: 1.7.0	agent:		tag: 1.7.0ingress:  enabled: true  annotations:    kubernetes.io/ingress.class: nginx    nginx.ingress.kubernetes.io/ssl-redirect: "true"    nginx.ingress.kubernetes.io/proxy-body-size: 10m  hosts:    - drone.javachen.com  tls:    - secretName: drone-test-wesine-com-cn-cert      hosts:        - drone.javachen.comsourceControl:  provider: gitea  gitea:    clientID: 7b0531cf-d894-4d00-b73a-f5de1002e051    clientSecretKey: clientSecret    clientSecretValue: Uivl6aItgxxCNCbN0yzkE0gnqpRlDc3pp_F9dUuUBhU=    server: https://gitea.javachen.comserver:  host: drone.javachen.com  adminUser: admin  protocol: https  alwaysAuth: true  envSecrets:    drone-gitea-login-secrets:    	- DRONE_GIT_USERNAME    	- DRONE_GIT_PASSWORD        kubernetes:    enabled: true    persistence:  enabled: true  storageClass: ceph-rbd  size: 5GiEOF
```

> 注意：上面配置了ceph的存储类ceph-rbd，需要提前创建。

安装

```bash
helm install --name gitea --namespace gitea -f drone-gitea-values.yaml alibaba/drone
```



## 测试

浏览器访问 <https://drone.javachen.space> ，跳到授权页面。


在Gitea中创建项目，设置.drone.yaml，然后修改代码，提交触发Drone构建。查看gitea仓库中webhook：


![](../assets/fd7a3cec1593ce540f8d0d03ce68afa7/006y8mN6ly1g8pjfq187jj31qy0hwmzl.jpg)


参考：[CentOS7 Docker x509: certificate signed by unknown authority 解决方案]()

```bash
sudo mkdir -p /etc/docker/certs.d/harbor.javachen.space/kubectl get secret -n harbor harbor-javachen-space-cert \    -o jsonpath="{.data.tls\.crt}" | base64 --decode | \    sudo tee /etc/docker/certs.d/harbor.javachen.space/ca.crt      sudo update-ca-trust
```



## 自动移除Jobs

默认情况下，执行完成后的Kubernetes Jobs不会自动从系统中清除，主要是为了便于对Pipeline执行过程中报错的故障排查。


如果需要实现自动清理，这里我找到了两种方式，可以深入的去了解一下：

1. 设置Kubernetes的 `TTLAfterFinished` 特性，这也是Drone官方给出的解决方法
2. 通过第三方的工具来实现：[lwolf/kube-cleanup-operator: Kubernetes Operator to automatically delete completed Jobs and their Pods](https://github.com/lwolf/kube-cleanup-operator) 

## 卸载

```bash
helm del drone -n dronekubectl delete certificates,secret,pvc --all -n drone
```



# 参考文章

- \[在Kubernetes上执行Drone CI/CD]\(
