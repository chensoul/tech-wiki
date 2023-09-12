

# 安装 Cert Manager

参考 [安装Cert Manager](/2019/11/02/install-cert-manager/) ，进行安装。 

# Let’s Encrypt 证书颁发原理

Let’s Encrypt 利用 [ACME](https://tools.ietf.org/html/rfc8555) 协议来校验域名是否真的属于你，校验成功后就可以自动颁发免费证书，证书有效期只有 90 天，在到期前需要再校验一次来实现续期，幸运的是 cert-manager 可以自动续期，这样就可以使用永久免费的证书了。如何校验你对这个域名属于你呢？主流的两种校验方式是 HTTP-01 和 DNS-01，下面简单介绍下校验原理: 

## HTTP-01 校验原理

HTTP-01 的校验原理是给你域名指向的 HTTP 服务增加一个临时 location ，`Let’s Encrypt` 会发送 http 请求到 `http:///.well-known/acme-challenge/`，`YOUR_DOMAIN` 就是被校验的域名，`TOKEN` 是 ACME 协议的客户端负责放置的文件，在这里 ACME 客户端就是 cert-manager，它通过修改 Ingress 规则来增加这个临时校验路径并指向提供 `TOKEN` 的服务。`Let’s Encrypt` 会对比 `TOKEN` 是否符合预期，校验成功后就会颁发证书。此方法仅适用于给使用 Ingress 暴露流量的服务颁发证书，并且不支持泛域名证书。 

## DNS-01 校验原理

DNS-01 的校验原理是利用 DNS 提供商的 API Key 拿到你的 DNS 控制权限， 在 Let’s Encrypt 为 ACME 客户端提供令牌后，ACME 客户端 (cert-manager) 将创建从该令牌和您的帐户密钥派生的 TXT 记录，并将该记录放在 `_acme-challenge.`。 然后 Let’s Encrypt 将向 DNS 系统查询该记录，如果找到匹配项，就可以颁发证书。此方法不需要你的服务使用 Ingress，并且支持泛域名证书。 

# 创建签发机构

我们需要先创建一个用于签发证书的 Issuer 或 ClusterIssuer，它们唯一区别就是 Issuer 只能用来签发自己所在 namespace 下的证书，ClusterIssuer 可以签发任意 namespace 下的证书，除了名称不同之外，两者所有字段完全一致，下面给出一些示例，简单起见，我们仅以 ClusterIssuer 为例。


Let’s Encrypt目前支持ACME和CA方式的Issuer，而ACME又分DNS/HTTP两种DNS校验方式。


Let’s Encrypt的API分为staging和prod两种，下面的例子使用staging的API做测试。

- staging：<https://acme-staging-v02.api.letsencrypt.org/directory>
- prod： <https://acme-v02.api.letsencrypt.org/directory> 

## 创建使用 HTTP-01 校验的 ClusterIssuer

使用 HTTP-01 方式校验，ACME 服务端 (Let’s Encrypt) 会向客户端 (cert-manager) 提供令牌，客户端会在 web server 上特定路径上放置一个文件，该文件包含令牌以及帐户密钥的指纹。ACME 服务端会请求该路径并校验文件内容，校验成功后就会签发免费证书，更多细节参考: <https://letsencrypt.org/zh-cn/docs/challenge-types/>


有个问题，ACME 服务端通过什么地址去访问 ACME 客户端的 web server 校验域名？答案是通过将被签发的证书中的域名来访问。这个机制带来的问题是:

1. 不能签发泛域名证书，因为如果是泛域名，没有具体域名，ACME 服务端就不能知道该用什么地址访问 web server 去校验文件。
2. 域名需要提前在 DNS 提供商配置好，这样 ACME 服务端通过域名请求时解析到正确 IP 才能访问成功，也就是需要提前知道你的 web server 的 IP 是什么。

cert-manager 作为 ACME 客户端，它将这个要被 ACME 服务端校验的文件通过 Ingress 来暴露，我们需要提前知道 Ingress 对外的 IP 地址是多少，这样才好配置域名。


一些云厂商自带的 ingress controller 会给每个 Ingress 都创建一个外部地址 (通常对应一个负载均衡器)，这个时候我们需要提前创建好一个 Ingress，拿到外部 IP 并配置域名到此 IP，ACME 客户端 (cert-manager) 修改此 Ingress 的 rules，临时增加一个路径指向 cert-manager 提供的文件，ACME 服务端请求这个域名+指定路径，根据 Ingress 规则转发会返回 cert-manger 提供的这个文件，最终 ACME 服务端 (Let’s Encrypt) 校验该文件，通过后签发免费证书。


指定 Ingress 的创建 `ClusterIssuer` 的示例：

```bash
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-stage-http01-cluster-issuer
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: junecloud@163.com
    privateKeySecretRef:
      name: letsencrypt-stage-http01-secret
    solvers:
    - selector: {} # An empty 'selector' means that this solver matches all domains
      http01: # ACME HTTP-01 solver configurations
        ingress:
          class: nginx
EOF
```

- `solvers.http01`: 配置 HTTP-01 校验方式所需的参数，`ingress.name` 指定提前创建好的 ingress 名称

有些自己安装的 ingress controller，所有具有相同 ingress class 的 ingress 都共用一个流量入口，通常是用 LoadBalancer 类型的 Service 暴露 ingress controller，这些具有相同 ingress class 的 ingress 的外部 IP 都是这个 Service 的外部 IP。这种情况我们创建 `ClusterIssuer` 时可以指定 ingress class，校验证书时，cert-manager 会直接创建新的 Ingress 资源并指定 `kubernetes.io/ingress.class` 这个 annotation。 

## 创建使用 DNS-01 校验的 ClusterIssuer

Cert Manager目前支持的DNS01服务商如下：

- [ACME-DNS]()
- [Akamai FastDNS]()
- [AzureDNS]()
- [Cloudflare]()
- [Google CloudDNS]()
- [Amazon Route53]()
- [DigitalOcean]()
- [RFC-2136]()

可以看到，上面不包括阿里云、DNSPOD等等，所以只能用webhook的方式，可以查看目前支持的webhookl的提供商：[Webhook](https://github.com/jetstack/cert-manager/blob/37e201244651311f9ec87616a780f76796ac636f/docs/tasks/issuers/setup-acme/dns01/webhook.rst)

- [alidns-webhook](https://github.com/pragkent/alidns-webhook)
- [cert-manager-webhook-dnspod](https://github.com/qqshfox/cert-manager-webhook-dnspod)
- <https://github.com/kaelzhang/cert-manager-webhook-dnspod>
- [cert-manager-webhook-selectel](https://github.com/selectel/cert-manager-webhook-selectel)
- [cert-manager-webhook-softlayer](https://github.com/cgroschupp/cert-manager-webhook-softlayer)

当然，也可以查看 [cert-manager-webhook-example](https://github.com/jetstack/cert-manager-webhook-example/network/members) 的folk情况，看是否有人已经实现了其他的webhook：


![](../assets/1489abb0b5616fef858ab3ae08dedae9/006y8mN6gy1g983dtlq2rj310o0u0n9q.jpg) 

### dnspod

1、安装cert-manager-webhook-dnspod


下载源代码

```bash
git clone https://github.com/qqshfox/cert-manager-webhook-dnspod.git
```

2、获取DNSPOD的id、token


参考 <https://support.dnspod.cn/Kb/showarticle/tsid/227/> ，新建密钥页面：<https://console.dnspod.cn/account/token#>


3、安装cert-manager-webhook-dnspod

```bash
cd cert-manager-webhook-dnspod
helm install cert-manager-webhook-dnspod -n cert-manager  \
    --set groupName=javachen.xyz \
    --set secrets.apiID=123438,secrets.apiToken=a028d574e4e390f259b97e4c3f4cb861 \
    --set clusterIssuer.enabled=true,clusterIssuer.email=admin@javachen.com \
    ./deploy/cert-manager-webhook-dnspod
```

6、查看状态：

```bash
kubectl get all,Issuer,clusterIssuer -n cert-manager
helm list -n cert-manager
```

查看签发机构：

```bash
$ kubectl get Issuer,clusterIssuer -n cert-manager |grep dnspod
issuer.cert-manager.io/cert-manager-webhook-dnspod-ca         True    23m
issuer.cert-manager.io/cert-manager-webhook-dnspod-selfsign   True    23m
clusterissuer.cert-manager.io/cert-manager-webhook-dnspod-cluster-issuer   True    23m
```



### godaddy

1、安装cert-manager-webhook-godaddy


下载源代码

```bash
git clone https://github.com/inspectorioinc/cert-manager-webhook-godaddy.git
```

进入目录，使用helm3安装：

```bash
cd cert-manager-webhook-godaddy
helm install cert-manager-webhook-godaddy --namespace cert-manager \
	--set groupName=acme.javachen.space \
	./deploy/example-webhook
```

> 注意：必须设置 groupName 参数，其对应一个域名，必须是唯一的

2、获取godaddy的API key和secret


在 <https://developer.godaddy.com/keys/> 页面创建一个API Key


3、查看状态：

    kubectl get all,secret,issuer,clusterIssuer -n cert-manager

可以看到cert-manager-webhook-godaddy只创建了issuer类型的签发机构，没有创建clusterIssuer类型的，所以需要手动创建。


创建 production issuer: [production-issuer.yaml](https://raw.githubusercontent.com/jetstack/cert-manager/release-0.11/docs/tutorials/acme/quick-start/example/production-issuer.yaml)

```yaml
cat << EOF | kubectl create -f -apiVersion: cert-manager.io/v1alpha2kind: ClusterIssuermetadata:  name: cert-manager-webhook-godaddy-cluster-issuerspec:  acme:    server: https://acme-v02.api.letsencrypt.org/directory    email: junecloud@163.com    privateKeySecretRef:      name: cert-manager-webhook-godaddy-letsencrypt    solvers:    - selector:        dnsNames:        - '*.javachen.space'      dns01:        webhook:          groupName: acme.javachen.space           solverName: godaddy          config:            authApiKey: e4hN4QrFgzdo_RHXe1ef2qpBPmiJPD2ZUcW            authApiSecret: QsHuDdnnCbzp5DmEQzq4ts            production: true            ttl: 600EOF
```

- `metadata.name`: 是我们创建的签发机构的名称，后面我们创建证书的时候会引用它
- `acme.email`: 是你自己的邮箱，证书快过期的时候会有邮件提醒，不过 cert-manager 会利用 acme 协议自动给我们重新颁发证书来续期
- `acme.server`: 是 acme 协议的服务端，我们这里用 Let’s Encrypt，这个地址就写死成这样就行
- `acme.privateKeySecretRef` 指示此签发机构的私钥将要存储到哪个 Secret 中，在 cert-manager 所在命名空间
- `solvers.dns01`: 配置 DNS-01 校验方式所需的参数，最重要的是 API Key (引用提前创建好的 Secret)，不同 DNS 提供商配置不一样，具体参考官方API文档
- 更多字段参考 API 文档: [https://docs.cert-manager.io/en/latest/reference/api-docs/index.html#clusterissuer-v1alpha2](https://docs.cert-manager.io/en/latest/reference/api-docs/index.html/#clusterissuer-v1alpha2)

查看状态

```bash
kubectl get ClusterIssuer,secret
```



# 手动创建证书

有了 Issuer/ClusterIssuer，接下来我们就可以生成免费证书了，cert-manager 给我们提供了 Certificate 这个用于生成证书的自定义资源对象，它必须局限在某一个 namespace 下，证书最终会在这个 namespace 下以 Secret 的资源对象存储。


使用letsencrypt-staging-http01创建证书：

```yaml
cat <<EOF | kubectl apply -f -apiVersion: cert-manager.io/v1alpha2kind: Certificatemetadata:  name: kubernetes-dashboard-cert  namespace: kubernetes-dashboardspec:  secretName: kubernetes-dashboard-tls  renewBefore: 240h  issuerRef:    name: letsencrypt-stage-http01-cluster-issuer    kind: ClusterIssuer  dnsNames:  - dashboard.javachen.spaceEOF
```

配置说明：

- `secretName`: 指示证书最终存到哪个 Secret 中
- `issuerRef.kind`: ClusterIssuer 或 Issuer，ClusterIssuer 可以被任意 namespace 的 Certificate 引用，Issuer 只能被当前 namespace 的 Certificate 引用。
- `issuerRef.name`: 引用我们创建的 Issuer/ClusterIssuer 的名称
- `commonName`: 对应证书的 common name 字段
- `dnsNames`: 对应证书的 Subject Alternative Names (SANs) 字段

使用说明：

- 使用该方式，需要在域名提供商那里添加一个DNS的A记录 dashboard.javachen.space，绑定到公网地址


  ![](../assets/1489abb0b5616fef858ab3ae08dedae9/00831rSTgy1gcydx2iu0bj31iq03qt8y.jpg)



- Ingress 服务的 EXTERNAL-IP 有一个外网可以访问的IP地址。ps：在k3s中traefik服务的 EXTERNAL-IP 就是当前机器的外网的地址，这是k3s特性？详细说明，可以参考 <https://opensource.com/article/20/3/ssl-letsencrypt-k3s>

```bash
$ kubectl get svc -n kube-system|grep traefikNAME                 TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGEtraefik-prometheus   ClusterIP      10.43.41.166    <none>           9100/TCP                     21mtraefik              LoadBalancer   10.43.95.236    144.34.194.100  80:31064/TCP,443:31072/TCP   21m
```

或者使用cert-manager-webhook-dnspod-cluster-issuer创建证书：

```yaml
cat <<EOF | kubectl apply -f -apiVersion: cert-manager.io/v1alpha2kind: Certificatemetadata:  name: kubernetes-dashboard-cert  namespace: kubernetes-dashboardspec:  secretName: kubernetes-dashboard-tls  renewBefore: 240h  issuerRef:    name: cert-manager-webhook-dnspod-cluster-issuer    kind: ClusterIssuer  dnsNames:  - dashboard.javachen.spaceEOF
```

查看secret中tls.crt、tls.key是否有值

```bash
kubectl get secret,certificatekubectl describe secret kubernetes-dashboard-tls
```

- `tls.crt` 就是颁发的证书
- `tls.key` 是证书密钥

查看证书是否生成：

```bash
kubectl describe certificate -n kubernetes-dashboard
```



# 自动创建证书

Ingress里面可以配置自动创建证书：

```yaml
cat << EOF | kubectl apply -f -apiVersion: extensions/v1beta1kind: Ingressmetadata:  name: flasktest-ingress  annotations:    kubernetes.io/ingress.class: nginx    cert-manager.io/cluster-issuer: cert-manager-webhook-dnspod-cluster-issuerspec:  tls:    - secretName: flask-javachen-space-tls      hosts:        - flask-javachen-space  rules:  - host: flask-javachen-space    http:      paths:        - path: /          backend:            serviceName: flasktest            servicePort: 80EOF
```

关键配置：

- cert-manager.io/cluster-issuer z指定签发机构
- tls 启用TLS 

# 参考文章

- [Make SSL certs easy with k3s](https://opensource.com/article/20/3/ssl-letsencrypt-k3s)
- <https://www.thebookofjoel.com/k3s-cert-manager-letsencrypt>
- [使用 LetsEncrypt配置kubernetes ingress-nginx免费HTTPS证书]()
- [cert-manager管理k8s集群证书]()
- [ACME-DNS]()
- <https://blog.csdn.net/xichenguan/article/details/100709830>
- [阿里云DNS配置Cert Manager Webhook]()
- [使用 cert-manager 自动生成证书]()
