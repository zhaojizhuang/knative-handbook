# 序言

![Github Last Commit](https://img.shields.io/github/last-commit/zhaojizhuang/knative-handbook)![Github Contributors](https://img.shields.io/github/contributors/zhaojizhuang/knative-handbook)



[knative](https://knative.dev/) 是谷歌开源的 serverless 架构方案，旨在提供一套简单易用的 serverless 方案，把 serverless 标准化。目前参与的公司主要是 Google、Pivotal、IBM、Red Hat，2018年7月24日才刚刚对外发布，当前还处于快速发展的阶段。主要功能包括：

* 基于容器的应用部署，支持多版本发布
* 基于流量百分比的蓝绿发布
* 根据请求自动扩缩容，支持缩容到零
* 基于 CloudEvents 的事件驱动
* 丰富的事件源
* 插件机制保证扩展性

Knative 发展非常迅速，旨在称为 Serverless 领域的事实标准。Kubernetes 的中文资料比较少，但系统化和紧跟社区更新的则就更少了。《Knative 指南》开源电子书旨在整理平时在开发和使用 Knative 时的参考指南和实践总结，形成一个系统化的参考指南以方便查阅。欢迎大家关注和添加完善内容。

### 在线阅读 <a id="&#x5728;&#x7EBF;&#x9605;&#x8BFB;"></a>

* 中文：
  * Gitbook: [zhaojizhuang.gitbook.io/knative](https://zhaojizhuang.gitbook.io/knative)

### 项目源码 <a id="&#x9879;&#x76EE;&#x6E90;&#x7801;"></a>

项目源码存放于 Github 上，[https://github.com/zhaojizhuang/knative-handbook](https://github.com/zhaojizhuang/knative-handbook)。

#### 本书版本更新记录 <a id="&#x672C;&#x4E66;&#x7248;&#x672C;&#x66F4;&#x65B0;&#x8BB0;&#x5F55;"></a>

如无特殊说明，本指南所有文档仅适用于 Knative v0.21 及以上版本。详细更新记录见 CHANGELOG。

### 微信公众号 <a id="&#x5FAE;&#x4FE1;&#x516C;&#x4F17;&#x53F7;"></a>

扫码关注微信公众号，回复关键字即可在微信中查看相关章节。

![](.gitbook/assets/knative.png)

### 贡献者 <a id="&#x8D21;&#x732E;&#x8005;"></a>

欢迎参与贡献和完善内容，贡献方法参考 [CONTRIBUTING](https://github.com/feiskyer/kubernetes-handbook/blob/master/CONTRIBUTING.md)。感谢所有的贡献者，贡献者列表见 [contributors](https://github.com/zhaojizhuang/knative-handbook/graphs/contributors)。

### LICENSE <a id="license"></a>

![](https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png)

[署名-非商业性使用-相同方式共享 4.0 \(CC BY-NC-SA 4.0\)](https://creativecommons.org/licenses/by-nc-sa/4.0/deed.zh)。

