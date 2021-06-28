# Nats 安装（选装）

### 1. Nats 安装

```text
kubectl create ns natss
kubectl apply -f https://raw.githubusercontent.com/knative-sandbox/eventing-natss/release-0.23/config/broker/natss.yaml
```

{% hint style="info" %}
可参考 `knative` 社区 `natss` 的安装 [readme](https://github.com/knative-sandbox/eventing-natss/blob/release-0.21/config/broker/README.md)
{% endhint %}

因为 `pod` 需要 `pv`，所以正常情况下 `pod` 应该是在 `pending`

```text
[root@k8s-master ~]# kubectl get po -n natss
NAME               READY   STATUS    RESTARTS   AGE
nats-streaming-0   0/1     Pending   0          19s
```

### 2. 准备PV

下面列出来一个 `pv` 的例子，注意 `path` 要提前创建好 `/mnt/disks/ssd2`，`mod` 最好改为 `777`

```yaml
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume2
  labels:
    type: local
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/disks/ssd2"
EOF
```

### 3. 查看 Pod 状态

可以看到 `Pod` 状态已经变为 `Running` 了

### 4. 安装 Nats controller，Nats dispatcher

`Nats controller` 和 `Nats dispatcher` 是  `Knative Eventing` 对接 `Natss` 的组件

```bash
kubectl apply -f https://github.com/knative-sandbox/eventing-natss/releases/download/v0.21.0/eventing-natss.yaml
```

安装成功之后会看到下面几个组件

```text
$ kubectl get po -n knative-eventing |grep natss
natss-ch-controller-7dd86fdbdf-tmhh8                    1/1     Running     0          7m43s
natss-ch-dispatcher-6587686545-xvpnh                    1/1     Running     0          7m43s
natss-webhook-d59f4fb6f-5x9w2                           1/1     Running     0          7m43s
```

### 5.最后一步！重要

安装到这一步还没完，还需要修改下 `nats controller` 和 `nats dispatcher` 中对接 `nats` 的配置: **nats url** 和 **cluster id**

* 默认情况下 对接 `NATS Streamin` 的 url 是  `nats://nats-streaming.natss.svc:4222`, 
* `NATS  Streaming` 的 `cluster id` 是 `knative-nats-streaming`           这个值是[ Nats 安装时](https://github.com/knative-sandbox/eventing-natss/blob/main/config/broker/natss.yaml#L74)指定的

`nats controller`   和  `nats dispatcher` 中添加如下 env：

```yaml
env:
  - name: DEFAULT_NATSS_URL
    value: nats://natss.custom-namespace.svc.cluster.local:4222
  - name: DEFAULT_CLUSTER_ID
    value: knative-nats-streaming
  - name: SYSTEM_NAMESPACE
    valueFrom:
      fieldRef:
        fieldPath: metadata.namespace
```

