# 在Knative中使用HPA

Knative支持HPA的弹性能力。您可以在Knative Service中设置CPU指标阈值，满足在突发高负载的场景下，自动扩缩容资源的诉求。本文介绍如何在Knative中使用HPA。

### 原理

Knative autoscaller 通过 读取  Knative Service annotations获取 cpu 和 memory 的相关配置，然后生成 Kubernetes 资源 `HPA，`具体扩缩容逻辑由 Kubernetes 的 autoscaller 来完成。

### 操作步骤 <a id="title-xiq-wtj-s1w"></a>

1. 创建ksvc.yaml。

   在Knative Service指定使用HPA弹性策略，以下为配置示例。

   ```text
   apiVersion: serving.knative.dev/v1
   kind: Service
   metadata:
     name: helloworld-nodejs
   spec:
     template:
       metadata:
         labels:
           app: helloworld-nodejs
         annotations:
           autoscaling.knative.dev/class: "hpa.autoscaling.knative.dev"
           autoscaling.knative.dev/metric: "cpu"
           autoscaling.knative.dev/target: "75"
           autoscaling.knative.dev/minScale: "1"
           autoscaling.knative.dev/maxScale: "10"
       spec:
         containers:
           - image: registry.cn-hangzhou.aliyuncs.com/knative-samples/helloworld-go:160e4dc8
             resources:
               requests:
                 cpu: '200m'              
   ```

   * 通过`autoscaling.knative.dev/class: "hpa.autoscaling.knative.dev"`指定HPA弹性插件。
   * 通过`autoscaling.knative.dev/metric`设置HPA CPU指标。
   * 通过`autoscaling.knative.dev/target`设置HPA CPU指标的阈值。
   * 通过`autoscaling.knative.dev/minScale: "1"`设置弹性策略实例数的最小值。
   * 通过`autoscaling.knative.dev/maxScale: "10"`设置弹性策略实例数的最大值。

2. 执行HPA弹性策略。

   ```text
   kubectl apply -f ksvc.yaml
   ```

### 执行结果 <a id="title-49d-6wl-491"></a>

Knative Service开启HPA弹性后，实例变化趋势如下图所示

![](../../.gitbook/assets/image%20%281%29.png)

### 查看 HPA资源

```bash
# kubectl get hpa
NAME                      REFERENCE                                       TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
helloworld-nodejs-00001   Deployment/helloworld-nodejs-00001-deployment   <unknown>/75%   1         10        1          15h
```

