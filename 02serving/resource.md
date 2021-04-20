# 资源对象

`Knative Serving` 定义了一组 `CRDs`（Custom Resource Definitions），通过这些对象来控制 `Serverless` 工作负载的行为。

这些 `CRDs` 包括：

* Service:  `services.serving.knative.dev`
* Route:  `routes.serving.knative.dev`
* Configuration: `configurations.serving.knative.dev`
* Revision: `revisions.serving.knative.dev`
* Ingress: `ingresses.networking.internal.knative.dev`
* ServerlessService: `serverlessservices.networking.internal.knative.dev`
* Image: `images.caching.internal.knative.dev`
* Kpa: `podautoscalers.autoscaling.internal.knative.dev`

下面会一一介绍这些自定义资源

