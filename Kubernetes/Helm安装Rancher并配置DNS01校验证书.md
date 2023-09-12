

# 版本说明

版本说明：

- Rancher：v2.3.5
- Cert-Manager：v0.12.0 

# Rancher Chart源码

Rancher Chart源码在 <https://github.com/rancher/rancher/tree/master/chart> ，当前rancher版本为2.3.5，可以看下[创建Issuer的源码](https://github.com/rancher/rancher/blob/master/chart/templates/issuer-letsEncrypt.yaml) issuer-letsEncrypt.yaml：

```yaml
{{- if eq .Values.tls "ingress" -}}
  {{- if eq .Values.ingress.tls.source "letsEncrypt" -}}
    {{- $certmanagerVer :=  split "." .Values.certmanager.version -}}
    {{- if or (.Capabilities.APIVersions.Has "cert-manager.io/v1alpha2") (and (eq (int $certmanagerVer._0) 0) (ge (int $certmanagerVer._1) 11)) }}
apiVersion: cert-manager.io/v1alpha2
    {{- else if or (.Capabilities.APIVersions.Has "certmanager.k8s.io/v1alpha1") (and (eq (int $certmanagerVer._0) 0) (lt (int $certmanagerVer._1) 11)) }}
apiVersion: certmanager.k8s.io/v1alpha1
    {{- end }}
kind: Issuer
metadata:
  name: {{ template "rancher.fullname" . }}
  labels:
    app: {{ template "rancher.fullname" . }}
    chart: {{ .Chart.Name }}-{{ .Chart.Version }}
    heritage: {{ .Release.Service }}
    release: {{ .Release.Name }}
spec:
  acme:
    {{- if eq .Values.letsEncrypt.environment "production" }}
    server: https://acme-v02.api.letsencrypt.org/directory
    {{- else }}
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    {{- end }}
    email: {{ .Values.letsEncrypt.email }}
    privateKeySecretRef:
      name: letsencrypt-{{ .Values.letsEncrypt.environment }}
    {{- if or (.Capabilities.APIVersions.Has "cert-manager.io/v1alpha2") (and (eq (int $certmanagerVer._0) 0) (ge (int $certmanagerVer._1) 11)) }}
    solvers:
    - http01:
        ingress:
          class: nginx
    {{- else if or (.Capabilities.APIVersions.Has "certmanager.k8s.io/v1alpha1") (and (eq (int $certmanagerVer._0) 0) (lt (int $certmanagerVer._1) 11)) }}
    http01: {}
    {{- end }}
  {{- end -}}
{{- end -}}
```

可以看到

- letsEncrypt使用的是http01校验，因为服务器上没有配置web服务器，所以用该方式校验比较麻烦，故改为dns01方式校验。
- 正对certmanager的api版本做了兼容 

# 添加DNS01校验

参考 [使用Cert Manager配置Let’s Encrypt证书](/2019/11/04/using-cert-manager-with-nginx-ingress/) 这篇完整，我们可以利用 [cert-manager-webhook-godaddy](https://github.com/inspectorioinc/cert-manager-webhook-godaddy) 通过webhook的方式进行dns01校验。

> 注意：请先安装 cert-manager-webhook-godaddy ，再往下操作

其创建issuer代码如下：

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: junecloud@163.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - selector:
        dnsNames:
        - '*.javachen.space'
      dns01:
        webhook:
          groupName: acme.javachen.space 
          solverName: godaddy
          config:
            authApiKey: e4hN4QrFgzdo_RHXe1ef2qpBPmiJPD2ZUcW
            authApiSecret: XXXXXXXX
            production: true
            ttl: 600
```

对比上面两个yaml文件，我们修改Rancher Chart源码，将http01改为solvers：

```yaml
solvers:
    - selector:
        dnsNames:
        - '*.javachen.space'
      dns01:
        webhook:
          groupName: acme.javachen.space 
          solverName: godaddy
          config:
            authApiKey: e4hN4QrFgzdo_RHXe1ef2qpBPmiJPD2ZUcW
            authApiSecret: XXXXXXXX
            production: true
            ttl: 600
```



# 修改Rancher Chart源码

查看 Rancher Chart：

```bash
$ helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
$ helm search repo rancher
NAME                  	CHART VERSION	APP VERSION	DESCRIPTION
rancher-stable/rancher	2.3.5        	v2.3.5     	Install Rancher Server to manage Kubernetes clusters ...
```

下载chart：

```bash
helm fetch rancher-stable/rancher
tar zxvf rancher-2.3.5.tgz
```

1、修改chart/templates/issuer-letsEncrypt.yaml：

```bash
{{- if eq .Values.tls "ingress" -}}
  {{- if eq .Values.ingress.tls.source "letsEncrypt" -}}
    {{- $certmanagerVer :=  split "." .Values.certmanager.version -}}
    {{- if or (.Capabilities.APIVersions.Has "cert-manager.io/v1alpha2") (and (eq (int $certmanagerVer._0) 0) (ge (int $certmanagerVer._1) 11)) }}
apiVersion: cert-manager.io/v1alpha2
    {{- else if or (.Capabilities.APIVersions.Has "certmanager.k8s.io/v1alpha1") (and (eq (int $certmanagerVer._0) 0) (lt (int $certmanagerVer._1) 11)) }}
apiVersion: certmanager.k8s.io/v1alpha1
    {{- end }}
kind: ClusterIssuer
metadata:
  name: {{ template "rancher.fullname" . }}
  labels:
    app: {{ template "rancher.fullname" . }}
    chart: {{ .Chart.Name }}-{{ .Chart.Version }}
    heritage: {{ .Release.Service }}
    release: {{ .Release.Name }}
spec:
  acme:
    {{- if eq .Values.letsEncrypt.environment "production" }}
    server: https://acme-v02.api.letsencrypt.org/directory
    {{- else }}
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    {{- end }}
    email: {{ .Values.letsEncrypt.email }}
    privateKeySecretRef:
      name: letsencrypt-{{ .Values.letsEncrypt.environment }}
    {{- if or (.Capabilities.APIVersions.Has "cert-manager.io/v1alpha2") (and (eq (int $certmanagerVer._0) 0) (ge (int $certmanagerVer._1) 11)) }}
{{- if .Values.letsEncrypt.solvers  }}   
    solvers:  
{{ toYaml .Values.letsEncrypt.solvers | indent 4 }}
{{- else }}
    solvers:
    - http01:
        ingress:
          class: nginx
{{- end }} 
    {{- else if or (.Capabilities.APIVersions.Has "certmanager.k8s.io/v1alpha1") (and (eq (int $certmanagerVer._0) 0) (lt (int $certmanagerVer._1) 11)) }}
    http01: {}
    {{- end }}
  {{- end -}}
{{- end -}}
```

说明：改动的地方有：

- Issuer改为ClusterIssuer，避免出现没有权限访问secret的问题
- 添加自定义的 .Values.letsEncrypt.solvers

2、修改rancher/ingress.yaml，将cert-manager.io/issuer改为cert-manager.io/cluster-issuer，certmanager.k8s.io/issuer改为certmanager.k8s.io/cluster-issuer。


3、修改rancher/values.yaml，在environment下面添加`solvers: []`

    letsEncrypt:
      # email: none@example.com
      environment: production
      solvers: []



# 安装Rancher

创建rancher-values.yaml：

```bash
cat <<EOF > rancher-values.yaml
hostname: rancher.javachen.space
ingress:
  tls:
    source: letsEncrypt
letsEncrypt:
  email: junecloud@163.com
  environment: production
  solvers: 
  - selector:
      dnsNames:
      - 'rancher.javachen.space'
    dns01:
      webhook:
        groupName: acme.javachen.space 
        solverName: godaddy
        config:
          authApiKey: e4hN4QrFgzdo_RHXe1ef2qpBPmiJPD2ZUcW
          authApiSecret: QsHuDdnnCbzp5DmEQzq4ts
          production: true
          ttl: 600
certmanager:
  version: "0.12.0"
EOF
```

从本地chart文件安装：

```bash
helm install rancher -n cattle-system \
	./rancher -f rancher-values.yaml
```

查看证书：

```bash
$ kubectl get secret,certificate,ClusterIssuer -n cattle-system
NAME                                   TYPE                                  DATA   AGE
secret/cattle-credentials-5582582      Opaque                                2      98d
secret/cattle-token-nxsvr              kubernetes.io/service-account-token   3      98d
secret/default-token-sx45c             kubernetes.io/service-account-token   3      98d
secret/letsencrypt-production          Opaque                                1      96m
secret/rancher-token-l2s27             kubernetes.io/service-account-token   3      10m
secret/sh.helm.release.v1.rancher.v1   helm.sh/release.v1                    1      10m
secret/tls-rancher                     kubernetes.io/tls                     2      98d
secret/tls-rancher-ingress             kubernetes.io/tls                     3      98d
NAME                                              READY   SECRET                AGE
certificate.cert-manager.io/tls-rancher-ingress   True    tls-rancher-ingress   10m
NAME                                                                       READY   AGE
clusterissuer.cert-manager.io/cert-manager-webhook-dnspod-cluster-issuer   True    58m
clusterissuer.cert-manager.io/rancher
```

可以看到letsEncrypt自动创建了证书 tls-rancher-ingress ，查看证书状态：

    kubectl get secret,certificate,ClusterIssuer -n cattle-system
    kubectl describe certificate tls-rancher-ingress -n cattle-system
    kubectl describe CertificateRequest tls-rancher-ingress-485758058 -n cattle-system
    kubectl describe order tls-rancher-ingress-2260491668 -n cattle-system
    kubectl describe Challenge tls-rancher-ingress-2260491668-881515835-3335958646 -n cattle-system

当tls-rancher-ingress状态为true时候，证书就创建成功了。

```bash
$ kubectl get certificate tls-rancher-ingress -n cattle-system
NAME                  READY   SECRET                AGE
tls-rancher-ingress   True    tls-rancher-ingress   35m
```

如果出错，根据日志提示进行排查原因。


查看rancher状态：

    kubectl -n cattle-system rollout status deploy/rancher
    kubectl -n cattle-system get deploy rancher



# 浏览器访问

先在本地配置hosts：

```bash
192.168.56.111 rancher.javachen.space
```

浏览器访问 <https://rancher.javachen.space/>


![](../assets/db9b9b6a83e1c1920147af2776836c7b/006y8mN6gy1g987c68rlwj30z20kojt0.jpg) 

# 卸载重装

```bash
helm del rancher -n cattle-system
kubectl delete pod,service,deploy,ingress,secret,pvc,replicaset,daemonset --all -n cattle-system
kubectl delete ServiceAccount,ClusterRoleBinding,Role,clusterissuer rancher -n cattle-system
```



# 升级版本

在rke安装目录下备份etcd：

```bash
rke etcd snapshot-save --name etcd-20200407.db --config cluster.yml
```

**结果:** RKE会获取每个`etcd`节点的快照，并保存在每个etcd节点的`/opt/rke/etcd-snapshots`目录下


查看本地repo：

    helm repo list

从源代码chart升级，先修改 ramcher/Chart.yml中的版本号

```yaml
apiVersion: v1
appVersion: v2.4.3
description: Install Rancher Server to manage Kubernetes clusters across providers.
home: https://rancher.com
icon: https://github.com/rancher/ui/blob/master/public/assets/images/logos/welcome-cow.svg
keywords:
- rancher
maintainers:
- email: charts@rancher.com
  name: Rancher Labs
name: rancher
sources:
- https://github.com/rancher/rancher
- https://github.com/rancher/server-chart
version: 2.4.3
```

执行升级命令：

```bash
helm upgrade rancher ./rancher -n cattle-system --version v2.4.3 -f rancher-values.yaml
```

> 通过`--version`指定升级版本，`镜像tag`不需要指定，会自动根据chart版本获取。

查看状态：

```bash
kubectl -n cattle-system rollout status deploy/rancher
kubectl -n cattle-system get deploy rancher
```



# 总结

我将修改后的rancher chart源代码提交到了 <https://github.com/javachen/charts> ，欢迎提意见。
