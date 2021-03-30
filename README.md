# Introduction

Kubernetes 是谷歌开源的容器集群管理系统，是 Google 多年大规模容器管理技术 Borg 的开源版本，也是 CNCF 最重要的项目之一，主要功能包括：

* 基于容器的应用部署、维护和滚动升级
* 负载均衡和服务发现
* 跨机器和跨地区的集群调度
* 自动伸缩
* 无状态服务和有状态服务
* 广泛的 Volume 支持
* 插件机制保证扩展性

Kubernetes 发展非常迅速，已经成为容器编排领域的领导者。Kubernetes 的中文资料也非常丰富，但系统化和紧跟社区更新的则就比较少见了。《Kubernetes 指南》开源电子书旨在整理平时在开发和使用 Kubernetes 时的参考指南和实践总结，形成一个系统化的参考指南以方便查阅。欢迎大家关注和添加完善内容。

### 在线阅读 <a id="&#x5728;&#x7EBF;&#x9605;&#x8BFB;"></a>

* 中文：
  * Gitbook: [zhaojizhuang.gitbook.io/knative](https://zhaojizhuang.gitbook.io/knative)
  * [Github](https://github.com/zhaojizhuang/knative-handbook)

### 项目源码 <a id="&#x9879;&#x76EE;&#x6E90;&#x7801;"></a>

项目源码存放于 Github 上，[https://github.com/feiskyer/kubernetes-handbook](https://github.com/feiskyer/kubernetes-handbook)。

#### 本书版本更新记录 <a id="&#x672C;&#x4E66;&#x7248;&#x672C;&#x66F4;&#x65B0;&#x8BB0;&#x5F55;"></a>

如无特殊说明，本指南所有文档仅适用于 Knative v0.21 及以上版本。详细更新记录见 [CHANGELOG](https://github.com/feiskyer/kubernetes-handbook/blob/master/CHANGELOG.md)。

### 微信公众号 <a id="&#x5FAE;&#x4FE1;&#x516C;&#x4F17;&#x53F7;"></a>

扫码关注微信公众号，回复关键字即可在微信中查看相关章节。

![](.gitbook/assets/knative.png)

### 贡献者 <a id="&#x8D21;&#x732E;&#x8005;"></a>

欢迎参与贡献和完善内容，贡献方法参考 [CONTRIBUTING](https://github.com/feiskyer/kubernetes-handbook/blob/master/CONTRIBUTING.md)。感谢所有的贡献者，贡献者列表见 [contributors](https://github.com/feiskyer/kubernetes-handbook/graphs/contributors)。

### LICENSE <a id="license"></a>

![](https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png)

[署名-非商业性使用-相同方式共享 4.0 \(CC BY-NC-SA 4.0\)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)。

