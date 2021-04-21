# 如何基于 Knative 开发 自定义controller

## 1. 为什么要开发 自定义 controller？

开源版本的 `Knative` 提供了扩缩容及事件驱动的架构，对于大部分场景的 `Serverless` 已经满足了，不过对于商业版本的 `Serverless` 平台来说，免不了要添加一些增强特性。

通常情况下，`In-Tree` 形式的增强不推荐，而且这种方式也会因开源版本升级带来不小的适配工作量。

`Out-Of-Tree` 形式的 自定义 `controller` 是一种很好特性增强方式，而且社区本身对于周边组件的解耦也是通过 controller 来对接的。比如：

* \*\*\*\*[**net-contour**](https://github.com/knative-sandbox/net-contour) **: 对接  Contour 七层负载的网络插件**
* \*\*\*\*[**net-kourier**](https://github.com/knative-sandbox/net-kourier) **: 对接  Kourier 七层负载的网络插件**
* \*\*\*\*[**net-istio**](https://github.com/knative-sandbox/net-istio) **: 对接  Istio 七层负载的网络插件**

> 上述提到的几个网络插件都是通过 自定义 `Controller` 结合  `Kingress` 这个 `CRD`资源来实现

![](../.gitbook/assets/knativegate.png)

## 2. How？

对于 有过 `Kubernetes` `operator` 开发经验的同学来说，可能对 [**Kubebuilder**](https://cloudnative.to/kubebuilder/introduction.html) ****更熟悉一些，其实 `Knative` 自定义 控制器的开发更简单，下面一步一步介绍怎么开始

### 2.1  Fork 社区 Template

社区 项目地址在  [https://github.com/knative-sandbox/sample-controller](https://github.com/knative-sandbox/sample-controller)，直接 fork 到个人仓库。

### 2.2 sample-controller介绍

代码下载到本地,目录如下，如下（此处省略掉不重要的文件）：

```bash
sample-controller
├── cmd
│   ├── controller 
│   │   └── main.go # controller 的启动入口文件
│   ├── schema
│   │   └── main.go # 生成 CRD 资源的 工具
│   └── webhook
│       └── main.go # webhook 的入口文件
├── config # controller 和webhook 的部署文件（deploy role clusterrole 等等，此处省略）
│   ├── 300-addressableservice.yaml
│   ├── 300-simpledeployment.yaml
├── example-addressable-service.yaml # CR 资源的示例yaml
├── example-simple-deployment.yaml # CR 资源的示例yaml
├── hack
│   ├── update-codegen.sh # 生成 informer，clientset，injection，lister 等
│   ├── update-deps.sh
│   ├── update-k8s-deps.sh
│   └── verify-codegen.sh
├── pkg
│   ├── apis
│   │   └── samples 
│   │       ├── register.go
│   │       └── v1alpha1 # 此处需编写 CRD 资源的types
│   ├── client # 执行 hack/update-codegen.sh 后自动生成的文件
│   │   ├── clientset
│   │   ├── informers
│   │   ├── injection
│   │   └── listers
│   └── reconciler # 此处是控制器的主要逻辑，示例中实现了两个控制器，每个控制器包含主控制器入口（controller.go） 和对应的 reconcile 逻辑
│       ├── addressableservice
│       │   ├── addressableservice.go
│       │   └── controller.go
│       └── simpledeployment
│           ├── controller.go
│           └── simpledeployment.go

```

**目录介绍：**

* **cmd**: 包含 `controller` 和`webhook` 的入口 `main` 函数,以及生成 crd  的 schema 工具（这也是笔者的社区贡献之一）
* **config**： controller 和webhook 的部署文件（本文只关注 controller）
* **hack**：是 程序自动生成代码的脚本，其中的 `update-codegen.sh` 最常用，是生成 `informer，clientset，injection，lister` 的工具
* **pkg/apis**: 此处是 CRD 定义的 types 文件
* **pkg/client:** 这里是 执行 `hackupdate-codegen.sh` 后自动生成的
* **pkg/reconciler**: 这里是控制器的主要逻辑，包括控制器主入口 `controller.go` 和对应的 `reconciler`逻辑

### 2.3 CRD 资源定义

#### 1. 确定 `GKV`，即资源的 `Group、Kind、Version`    

\`\`

\`\`



