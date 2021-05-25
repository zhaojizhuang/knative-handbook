# 自动扩缩容 KPA 配置



Knative中提供了开箱即用、基于流量请求的自动扩缩容KPA（Knative Pod Autoscaler）功能。本文介绍如何基于流量请求数实现服务自动扩缩容。

### 背景信息 <a id="h2-url-1"></a>

Knative Serving为每个Pod注入QUEUE代理容器（queue-proxy），该容器负责向Autoscaler报告业务容器的并发指标。Autoscaler接收到这些指标之后，会根据并发请求数及相应的算法，调整Deployment的Pod数量，从而实现自动扩缩容。[![&#x6269;&#x7F29;&#x5BB9;](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/6032790261/p172886.png)](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/6032790261/p172886.png)

并发数和QPS

* 并发数是同一时刻Pod的接收的请求数。
* QPS指的是Pod每秒响应的请求数，也就是最大吞吐能力。

并发数的增加并不一定会导致QPS增加。应用在访问压力较大的情况下，如果并发数增加，系统的QPS反而会下降。因为系统超负荷工作，CPU、内存等其他消耗导致系统性能下降，从而导致响应延迟。

### 算法 <a id="title-9d9-afp-irj"></a>

Autoscaler基于每个Pod的平均请求数（并发数）进行自动扩缩容，默认并发数为100。Pod数=并发请求总数/容器并发数。如果服务中并发数设置为10，并且加载了50个并发请求的服务，则Autoscaler就会创建5个Pod。Autoscaler实现了两种操作模式的缩放算法，Stable稳定模式和Panic恐慌模式：

* Stable稳定模式。

  在稳定模式下，Autoscaler调整Deployment的大小，以实现每个Pod所需的平均并发数。Pod的并发数是根据60秒窗口内接收所有数据请求的平均数来计算的。

* Panic恐慌模式。Autoscaler计算60秒窗口内的平均并发数，系统需要1分钟稳定在所需的并发级别。但是，Autoscaler也会计算6秒的恐慌窗口，如果该窗口达到目标并发的2倍，则会进入恐慌模式。在恐慌模式下，Autoscaler在更短、更敏感的紧急窗口上工作。一旦紧急情况持续60秒后，Autoscaler将返回初始的60秒稳定窗口。

  ```text
                                                         |
                                    Panic Target--->  +--| 20
                                                      |  |
                                                      | <------Panic Window
                                                      |  |
         Stable Target--->  +-------------------------|--| 10   CONCURRENCY
                            |                         |  |
                            |                      <-----------Stable Window
                            |                         |  |
  --------------------------+-------------------------+--+ 0
  120                       60                           0
                       TIME
  ```

### KPA配置介绍 <a id="title-ixk-3vz-gna"></a>

* 配置KPA，需要配置config-autoscaler，该参数默认已配置，以下为重点参数介绍。

  执行以下命令，查看config-autoscaler。

  ```text
  kubectl -n knative-serving get cm config-autoscaler
  ```

  预期输出：

  ```text
  apiVersion: v1
  kind: ConfigMap
  metadata:
   name: config-autoscaler
   namespace: knative-serving
  data:
   container-concurrency-target-default: "100"
   container-concurrency-target-percentage: "0.7"
   enable-scale-to-zero: "true"
   max-scale-up-rate: "1000"
   max-scale-down-rate: "2"
   panic-window-percentage: "10"
   panic-threshold-percentage: "200"
   scale-to-zero-grace-period: "30s"
   scale-to-zero-Pod-retention-period: "0s"
   stable-window: "60s"
   target-burst-capacity: "200"
   requests-per-second-target-default: "200"
  ```

  * 为KPA配置缩容至0。
    * scale-to-zero-grace-period：表示在缩为0之前，`inactive revison`保留的运行时间（最小是30s）。

      ```text
      scale-to-zero-grace-period: 30s
      ```

    * stable-window：当在Stable模式运行中，Autoscaler在稳定窗口期下平均并发数下的操作。

      ```text
      stable-window: 60s
      ```

      stable-window也可以在Revision注释中配置。

      ```text
      autoscaling.knative.dev/window: 60s
      ```

    * enable-scale-to-zero：设置`enable-scale-to-zero`参数为`true`。

      ```text
      enable-scale-to-zero: "true"
      ```
  * 配置Autoscaler的并发数。
    * `targettarget`定义在给定时间（软限制）需要多少并发请求，是Knative中Autoscaler的推荐配置。在ConfigMap中默认配置的并发target为100。

      ```text
      `container-concurrency-target-default: 100`
      ```

      这个值可以通过Revision中的`autoscaling.knative.dev/target`注释进行修改。

      ```text
      autoscaling.knative.dev/target: 50
      ```

    * `containerConcurrencycontainerConcurrency`限制在给定时间允许并发请求的数量（硬限制），并在Revision模板中配置。**说明**

      使用`containerConcurrency`，需要满足以下条件：

      * 只有在明确需要限制在给定时间有多少请求到达应用程序时，才应该使用`containerConcurrency`。
      * 只有当应用程序需要强制的并发约束时，才建议使用`containerConcurrency`。

      ```text
      containerConcurrency: 0 | 1 | 2-N
      ```

      * 1：确保一次只有一个请求由Revision给定的容器实例处理。
      * 2-N：请求的并发值限制为2或更多。
      * 0：表示不作限制，有系统自身决定。

    * container-concurrency-target-percentage

      这个参数为并发百分比或称为并发因子，并且会直接参与扩缩容并发数计算。例如，如果并发数target或者`containerConcurrency`设置值为100，并发因子container-concurrency-target-percentage为0.7。那么实际担当并发数达到70（100\*0.7）时就会触发扩容操作。

      因此，实际扩缩容并发数=target（或者`containerConcurrency`）\*container-concurrency-target-percentage

* 配置扩缩容边界，通过`minScale`和`maxScale`可以配置应用程序提供服务的最小和最大Pod数量。通过这两个参数配置可以控制服务冷启动或者控制计算成本。**说明**

  * 如果未设置`minScale`注释，Pods将缩放为零.。如果设置config-autoscaler的`enable-scale-to-zero`为`false`，则缩放为1。
  * 如果未设置`maxScale`注释，则创建的Pod数量将没有上限。

  `minScale`和`maxScale`可以在Revision模板中按照以下方式进行配置：

  ```text
  spec:
    template:
      metadata:
        autoscaling.knative.dev/minScale: "2"
        autoscaling.knative.dev/maxScale: "10"
  ```

### 场景一：设置并发请求数实现自动扩缩容 <a id="title-4b5-6v0-880"></a>

设置并发请求数，通过KPA实现自动扩缩容。

1. 创建autoscale-go.yaml。

   设置当前最大并发请求数为10。

   ```text
   apiVersion: serving.knative.dev/v1alpha1
   kind: Service
   metadata:
     name: autoscale-go
     namespace: default
   spec:
     template:
       metadata:
         labels:
           app: autoscale-go
         annotations:
           autoscaling.knative.dev/target: "10"
       spec:
         containers:
           - image: registry.cn-hangzhou.aliyuncs.com/knative-sample/autoscale-go:0.1
   ```

2. 执行以下命令，获取访问网关。

   ```text
   kubectl -n knative-serving get svc
   ```

   预期输出：

   ```text
   NAME              TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
   ingress-gateway   LoadBalancer   172.19.1*.***   121.199.19*.***   80:32185/TCP,443:31137/TCP   69d
   ```

3. 使用Hey压测工具，执行30s内保持50个并发请求。**说明** Hey压测工具的详细介绍，请参见[Hey](https://github.com/rakyll/hey)

   ```text
   hey -z 30s -c 50   -host "autoscale-go.default.example.com"   "http://121.199.194.150?sleep=100&prime=10000&bloat=5"
   ```

   预期输出：[![hey](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/9444562061/p172988.gif)](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/9444562061/p172988.gif)结果正如所预期的，扩容了5个Pod。

### 场景二：设置扩缩容边界实现自动扩缩容 <a id="title-785-drb-hps"></a>

扩缩容边界指应用程序提供服务的最小和最大Pod数量。通过设置应用程序提供服务的最小和最大Pod数量实现自动扩缩容。

1. [开启Knative](https://help.aliyun.com/document_detail/185057.htm#task-1956927)。
2. 创建autoscale-go.yaml。

   设置最大并发请求数为10，`minScale`最小保留实例数为1，`maxScale`最大扩容实例数为3。

   ```text
   apiVersion: serving.knative.dev/v1alpha1
   kind: Service
   metadata:
     name: autoscale-go
     namespace: default
   spec:
     template:
       metadata:
         labels:
           app: autoscale-go
         annotations:
           autoscaling.knative.dev/target: "10"
           autoscaling.knative.dev/minScale: "1"
           autoscaling.knative.dev/maxScale: "3"   
       spec:
         containers:
           - image: registry.cn-hangzhou.aliyuncs.com/knative-sample/autoscale-go:0.1
   ```

3. 使用Hey压测工具，执行30s内保持50个并发请求。**说明** Hey压测工具的详细介绍，请参见[Hey](https://github.com/rakyll/hey)。

   ```text
   hey -z 30s -c 50   -host "autoscale-go.default.example.com"   "http://121.199.194.150?sleep=100&prime=10000&bloat=5"
   ```

   预期输出：[![&#x81EA;&#x52A8;&#x6269;&#x7F29;&#x5BB9;](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/9444562061/p172995.gif)](https://static-aliyun-doc.oss-accelerate.aliyuncs.com/assets/img/zh-CN/9444562061/p172995.gif)结果正如所预期，最多扩容出来了3个Pod，并且即使在无访问请求流量的情况下，保持了1个运行的Pod。

