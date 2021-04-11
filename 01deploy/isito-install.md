# isito 安装

> istio 同样 推荐通过 operator 形式来安装

## 1. 安装 istio 的operator 和crd

安装 operator deployment 和 operator crd，此处安装的是 istio 1.8.0版本，[官网安装链接](https://istio.io/latest/docs/setup/install/operator/)

首先下载 `istioctl` [https://github.com/istio/istio/releases/](https://github.com/istio/istio/releases/)

此处下载的是 `istioctl` `istio 1.8.0` 版本

```yaml
istioctl operator  dump --hub docker.io/istio > istio_operator.yaml
```

```yaml
kubectl apply -f istio_operator.yaml # 安装 operator 和 CR 资源
```

通过 istioctl 生成对应的yaml ,其中 `--hub docker.io/istio`可以指定 operator deployment 镜像的 hub地址，此处换成自己的私有仓库地址，如图 `--hub docker.io/istio` 生成的镜像地址是 `image: docker.io/istio/operator:1.8.0` ,版本和 istioctl 的版本相同, 如果想自定义还可以通过 `--tag` 来指定。

## 2. 安装对应的 CR 资源

{% tabs %}
{% tab title="简单版本" %}
```yaml
$ kubectl create ns istio-system
$ kubectl apply -f - <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istiocontrolplane
spec:
  profile: demo
EOF
```
{% endtab %}

{% tab title="自定义参数版本" %}
```yaml
$ kubectl create ns istio-system
$ kubectl apply -f - <<EOF
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istiocontrolplane
spec:
  components:
    ingressGateways:
      - enabled: true
        name: istio-ingressgateway #这个是默认启用的
      - enabled: true
        label:
          app: cluster-local-gateway
          istio: cluster-local-gateway
        name: cluster-local-gateway  # 这个默认是不启用的，如果 knative 启用集群内访问，这里要打开
  meshConfig:
    rootNamespace: istio-system #istio的 namespace
  values:
    global:
      istioNamespace: istio-system #istio的 namespace
      hub: docker.io/istio  # 此处可以改为自己的私有仓库
      tag: 1.8.0  # 指定 istio 组件的版本
      imagePullPolicy: IfNotPresent # 指定镜像拉取策略
      proxy:
        clusterDomain: cluster.local # 如果 k8s 的域名 后缀不是 cluster.local,要改这里
      trustDomain: cluster.local  # 同上
EOF
```
{% endtab %}
{% endtabs %}

