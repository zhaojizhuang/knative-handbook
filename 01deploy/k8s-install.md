# kubernetes 安装

### 1. kubernetes 安装

此处不在赘述，可查看[官网安装指南](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) 

1. 安装 `docker`

配置国内的 `docker` `yum` 源

```text
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

2. 配置 `kubernetes` `yum` 源，对于国内访问不了 `kubernetes` 的问题，`centos`  可用阿里的`yum` 源,配置如下 

```text
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

 2. 安装   `kubelet`  `kubectl` `kubeadm`

```text
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

### 2. 网络插件安装

安装 网络插件，此处用的是 `cilium`

参考[官网安装指南](https://docs.cilium.io/en/v1.10/gettingstarted/k8s-install-helm/#installation-using-helm), 一行命令搞定

```text
helm install cilium cilium/cilium --version 1.10.1 \
--namespace kube-system \
--set kubeProxyReplacement=strict
```



