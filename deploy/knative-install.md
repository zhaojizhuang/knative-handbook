# Knative 安装

主要介绍通过 `Operator` 来安装 **`Knative Serving`** 和 **`Eventing`** 系统组件。

## 1. Knative Operator 安装 

通过下面的命令安装  v0.21.0 版本的 **Knative  operator**

```text
kubectl apply -f https://github.com/knative/operator/releases/download/v0.21.0/operator.yaml
```

## 2. 安装 Serving 

### 1. Serving

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

## 4. 安装 Eventing





