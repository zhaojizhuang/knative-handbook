# Knative 安装

主要介绍通过 `Operator` 来安装 **`Knative Serving`** 和 **`Eventing`** 系统组件。

## 1. Knative Operator 安装 

通过下面的命令安装  v0.23.0 版本的 **Knative  operator**

```text
kubectl apply -f https://github.com/knative/operator/releases/download/v0.23.0/operator.yaml 
```

## 2. 安装 Serving 

### 1. 通过 CR 安装 Serving 组件

```yaml
cat <<-EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
 name: knative-serving
---
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
spec:
  version: 0.23.0
  # config
  config:
    defaults:
      container-concurrency: "50"
    autoscaler:
      enable-scale-to-zero: "true"
    deployment:
      # replace queue
      queueSidecarImage: gcr.io/knative-releases/knative.dev/serving/cmd/queue:v0.23.0
    domain:
      mydomain.com: |
      
    network: 
      domainTemplate: |-
        {{if index .Annotations "my.func/subdomain" -}} 
          {{- index .Annotations "my.func/subdomain" -}}
        {{else -}}
          {{- .Name}}.{{.Namespace -}}
        {{end -}}
          .{{.Domain}}
    istio:
      local-gateway.knative-serving.cluster-local-gateway: "cluster-local-gateway.istio-system.svc.cluster.local"
    # image
  registry:
    override:
      # replace images 此处可换为自己的私有镜像,这里用的原生默认的镜像，如果要自己替换，按照下面的选项一次替换即可
      activator: gcr.io/knative-releases/knative.dev/serving/cmd/activator:v0.23.0
      webhook: gcr.io/knative-releases/knative.dev/serving/cmd/webhook:v0.23.0
      controller: gcr.io/knative-releases/knative.dev/serving/cmd/controller:v0.23.0
      autoscaler: gcr.io/knative-releases/knative.dev/serving/cmd/autoscaler:v0.23.0
      autoscaler-hpa: gcr.io/knative-releases/knative.dev/serving/cmd/autoscaler-hpa:v0.23.0
      istio-webhook/webhook: gcr.io/knative-releases/knative.dev/net-istio/cmd/webhook:v0.23.0
      networking-istio: gcr.io/knative-releases/knative.dev/net-istio/cmd/controller:v0.23.0
     # migrate: gcr.io/knative-releases/knative.dev/serving/vendor/knative.dev/pkg/apiextensions/storageversion/cmd/migrate@sha256:4e44b147321c96767328f6c6551964c93c9805ae14ff9f99a01c01a02c056a38
EOF
```

### 2. 安装 istio 组件

如果不指定 ，系统默认适配的 网络插件是 istio, 可以看到 knative 对接 istio的组件（`isito-webhook`,`networking-istio`） 已经安装完成。

```text
kubectl get deployment -n knative-serving

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
activator          1/1     1            1           18s
autoscaler         1/1     1            1           18s
autoscaler-hpa     1/1     1            1           14s
controller         1/1     1            1           18s
istio-webhook      1/1     1            1           12s
networking-istio   1/1     1            1           12s
webhook            1/1     1            1           17s
```

如果还没有安装 **istio** 系统组件,可以参考上一章节，已经安装可以直接跳过。

{% page-ref page="isito-install.md" %}

### 3. 查看安装结果

```text
$ kubectl get knativeservings.operator.knative.dev -n knative-serving
NAME              VERSION   READY   REASON
knative-serving   0.23.0    True
```

## 4. 安装 Eventing

### 1. 通过 CR 安装 Eventing 组件

* 如果 `Broker` 想基于 `natss channel`，请按照 Tab `Natss channel` 中的教程（安装完成继续下一章的 Nats 安装）
* 默认是基于内存的的 `channel` \(不会持久化消息\)

{% tabs %}
{% tab title="Natss channel" %}
```yaml
cat <<-EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
 name: knative-eventing
---
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeEventing
metadata:
  name: knative-eventing
  namespace: knative-eventing
spec:
  version: 0.23.0
  defaultBrokerClass: MTChannelBasedBroker
  config:
    # 配置 Broker 默认的 channel 是natss
    br-default-channel:
      channelTemplateSpec: |
        apiVersion: messaging.knative.dev/v1beta1
        kind: NatssChannel
    # 配置 broker 默认配置 
    br-defaults:
      default-br-config: |
        clusterDefault:
          apiVersion: v1
          brokerClass: MTChannelBasedBroker
          kind: ConfigMap
          name: config-br-default-channel
          namespace: knative-eventing
EOF
  # replace images 此处可换为自己的私有镜像,这里用的原生默认的镜像，如果要自己替换，按照下面的选项一次替换即可
  #registry: 
    #override:
     # eventing-controller/eventing-controller: gcr.io/knative-releases/knative.dev/eventing/cmd/controller:v0.21.0
     # eventing-webhook/eventing-webhook: gcr.io/knative-releases/knative.dev/eventing/cmd/webhook:v0.21.0
     # imc-controller/controller: gcr.io/knative-releases/knative.dev/eventing/cmd/in_memory/channel_controller:v0.21.0
     # imc-dispatcher/dispatcher: gcr.io/knative-releases/knative.dev/eventing/cmd/in_memory/channel_dispatcher:v0.21.0
     # mt-broker-controller/mt-broker-controller: gcr.io/knative-releases/knative.dev/eventing/cmd/mtchannel_broker:v0.21.0
     # mt-broker-filter/filter: gcr.io/knative-releases/knative.dev/eventing/cmd/mtbroker/filter:v0.21.0
     # mt-broker-ingress/ingress: gcr.io/knative-releases/knative.dev/eventing/cmd/mtbroker/ingress:v0.21.0
     # pingsource-mt-adapter/dispatcher: gcr.io/knative-releases/knative.dev/eventing/cmd/mtping:v0.21.0
     # sugar-controller/controller: gcr.io/knative-releases/knative.dev/eventing/cmd/sugar_controller:v0.21.0
#EOF
```
{% endtab %}

{% tab title="默认 channel" %}
```yaml
cat <<-EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
 name: knative-eventing
---
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeEventing
metadata:
  name: knative-eventing
  namespace: knative-eventing
spec:
  version: 0.21.0
  defaultBrokerClass: MTChannelBasedBroker
EOF
  # replace images 此处可换为自己的私有镜像,这里用的原生默认的镜像，如果要自己替换，按照下面的选项一次替换即可
  #registry: 
    #override:
     # eventing-controller/eventing-controller: gcr.io/knative-releases/knative.dev/eventing/cmd/controller:v0.21.0
     # eventing-webhook/eventing-webhook: gcr.io/knative-releases/knative.dev/eventing/cmd/webhook:v0.21.0
     # imc-controller/controller: gcr.io/knative-releases/knative.dev/eventing/cmd/in_memory/channel_controller:v0.21.0
     # imc-dispatcher/dispatcher: gcr.io/knative-releases/knative.dev/eventing/cmd/in_memory/channel_dispatcher:v0.21.0
     # mt-broker-controller/mt-broker-controller: gcr.io/knative-releases/knative.dev/eventing/cmd/mtchannel_broker:v0.21.0
     # mt-broker-filter/filter: gcr.io/knative-releases/knative.dev/eventing/cmd/mtbroker/filter:v0.21.0
     # mt-broker-ingress/ingress: gcr.io/knative-releases/knative.dev/eventing/cmd/mtbroker/ingress:v0.21.0
     # pingsource-mt-adapter/dispatcher: gcr.io/knative-releases/knative.dev/eventing/cmd/mtping:v0.21.0
     # sugar-controller/controller: gcr.io/knative-releases/knative.dev/eventing/cmd/sugar_controller:v0.21.0
#EOF
```
{% endtab %}
{% endtabs %}



