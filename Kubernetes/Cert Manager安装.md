参考官方文档： [https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html]() 

# 创建命名空间

    kubectl create namespace cert-manager

cert-manager 部署时会生成 [ValidatingWebhookConfiguration](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/) 注册ValidatingAdmissionWebhook 来实现 CRD 校验，而`ValidatingWebhookConfiguration` 里需要写入 `cert-manager` 自身校验服务端的证书信息，就需要在自己命名空间创建 `ClusterIssuer` 和 `Certificate` 来自动创建证书，创建这些 CRD 资源又会被校验服务端校验，但校验服务端证书还没有创建所以校验请求无法发送到校验服务端，这就是一个鸡生蛋还是蛋生鸡的问题了，所以我们需要关闭 `cert-manager` 所在命名空间的 CRD 校验，通过打 label 来实现:

```bash
kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true
kubectl label namespace cert-manager cert-manager.io/disable-validation=true
```



# 使用kubectl安装

```bash
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.13.0/cert-manager.yaml
```

> 使用 `kubectl v1.15` 及其以下的版本需要加上 `--validate=false`



# 使用helm安装

安装 CRD

```bash
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.13/deploy/manifests/00-crds.yaml
```

添加 repo：

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

执行安装

```bash
helm install cert-manager \
  -n cert-manager \
  --version v0.13.0 \
  jetstack/cert-manager
```

更多详细的配置参数可以参考：<https://github.com/jetstack/cert-manager/blob/master/deploy/charts/cert-manager/values.yaml>


用于设置certificate删除时自动删除secret：

```bash
extraArgs:
  - --enable-certificate-owner-ref=true
```

查看状态

```bash
$ kubectl -n cert-manager rollout status deploy/cert-manager
deployment "cert-manager" successfully rolled out
$ kubectl get pods -n cert-manager
NAME                                           READY   STATUS              RESTARTS   AGE
pod/cert-manager-5c47f46f57-k78l6              1/1     Running             0          91s
pod/cert-manager-cainjector-6659d6844d-tr8rf   1/1     Running             0          91s
pod/cert-manager-webhook-547567b88f-8lthd      1/1     Running   					 0          91s
```

使用helm3查看：

```bash
$ helm list -n cert-managerNAME        	NAMESPACE   	REVISION	UPDATED                                	STATUS  	CHART               	APP VERSIONcert-manager	cert-manager	1       	2019-12-09 15:55:03.245917473 +0800 CST	deployed	cert-manager-v0.13.0	v0.13.0
```



# 卸载

```bash
#通过kubectl删除kubectl delete -f https://github.com/jetstack/cert-manager/releases/download/v0.13.0/cert-manager.yaml#通过helm3删除helm del cert-manager -n cert-manager
```
