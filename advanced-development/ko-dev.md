# 如何利用 Google 开源工具 Ko 在 kubernetes 上构建并部署 Go 应用

`Ko` 是 `Google` 开源的一款用于构建并部署 `Go` 应用的工具。

这是一款简单、快速的 `Go` 应用镜像构建器。并与 `Kubernetes` 集成，能够将应用快速部署到 `Kubernetes` 上。是云原生时代 `Kubernetes` 应用开发的一大利器。

### 特点：

* 需要构建的 `Go` 应用对系统镜像无太多依赖（例如，无 `cgo`，无 `OS` 软件包依赖关系），最好是只有一个 `go` 二进制。
* 构建镜像的过程不需要 `Docker`   ，因此可以用在轻量化的 `CI/CD` 场景。
* 支持 `yaml` 模板，可以直接用于部署 `Kubernetes` 应用。

## 如何使用

官方地址在这 [https://github.com/google/ko](https://github.com/google/ko)

### 1. 安装 ko

此处安装的是 `v0.8.2` 版本的 `linux x86` 版本，可以根据需要自行选择版本下载

**Release 地址:** [https://github.com/google/ko/releases](https://github.com/google/ko/releases)

{% tabs %}
{% tab title="手动安装" %}
```text
curl -L https://github.com/google/ko/releases/download/v0.8.2/ko_0.8.2_Linux_x86_64.tar.gz | tar xzf - ko
chmod +x ./ko
sudo mv ko /usr/local/bin
```
{% endtab %}

{% tab title="brew 安装" %}
```bash
brew install ko
```
{% endtab %}

{% tab title="go install" %}
```
go install github.com/google/ko
```
{% endtab %}
{% endtabs %}

### 2. 镜像仓库的认证

Ko 依赖的是 Docker 的镜像仓库的认证（ Docker config 文件），即 `~/.docker/config.json`  

```text
cat ~/.docker/config.json
{
	"auths": {
		"https://index.docker.io/v1/": {},
	},
	"HttpHeaders": {
		"User-Agent": "Docker-Client/19.03.13 (darwin)"
	},
	"credsStore": "desktop",
	"experimental": "disabled",
	"stackOrchestrator": "swarm"
}
```

如果镜像仓库没有登录过，需要先进行 `Docker login` 生成 对应的认证配置文件，比如我直接用的是 `docker hub` ,那么直接 `docker login` 即可。

### 3. 设置私有仓库的地址

`Ko` 通过 环境变量 `KO_DOCKER_REPO`  配置私有仓库的地址, 这决定了 `Ko` 会将编译好的镜像推到哪个仓库。

```bash
 # 这是我的 dockerhub 账号，
 # 这里也可以设置为 docker.io/zhaojizhuang66,不过 docker 会省略掉
export KO_DOCKER_REPO = zhaojizhuang66
```

### 4. 镜像构建 Ko publish

> **首先代码一定要在本地的 `Go path` 中**，可以下载示例代码  [https://github.com/zhaojizhuang/http-helloworld](https://github.com/zhaojizhuang/http-helloworld)

```bash
mkdir -p $GOPATH/src/github.com
cd $GOPATH/src/github.com
git clone  https://github.com/zhaojizhuang/http-helloworld
```

`Ko publish` 构建镜像，执行命令 `ko publish github.com/http-helloworld/cmd/helloworld` \(main函数所在的 `GoPath` 路径\)。

```bash
$ ko publish github.com/http-helloworld/cmd/helloworld

2021/04/23 17:57:41 Using base gcr.io/distroless/static:nonroot for github.com/http-helloworld/cmd/helloworld
2021/04/23 17:57:41 No matching credentials were found for "gcr.io/distroless/static", falling back on anonymous
2021/04/23 17:57:45 Building github.com/http-helloworld/cmd/helloworld for linux/amd64
2021/04/23 17:57:46 Publishing zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648:latest
2021/04/23 17:57:52 pushed blob: sha256:a3e2b18c53ecc2aab65b3b70e5c16486e48f76490febeb68d99aa18a48265b08
2021/04/23 17:57:52 pushed blob: sha256:72164b581b02b1eb297b403bcc8fc1bfa245cb52e103a3a525a0835a58ff58e2
2021/04/23 17:57:56 pushed blob: sha256:5dea5ec2316d4a067b946b15c3c4f140b4f2ad607e73e9bc41b673ee5ebb99a3
2021/04/23 17:58:07 pushed blob: sha256:d99fea081f251cc06ce68ce7cb224e2a0f63babd03ee9dd6bb03f6bf76dcb5a5
2021/04/23 17:58:08 zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648:latest: digest: sha256:21d352ec9f9079f8da4c91cfe2461df51a404c079f262390b19fff4cb2ce15a0 size: 750
2021/04/23 17:58:08 Published zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648@sha256:21d352ec9f9079f8da4c91cfe2461df51a404c079f262390b19fff4cb2ce15a0
zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648@sha256:21d352ec9f9079f8da4c91cfe2461df51a404c079f262390b19fff4cb2ce15a0
```

也支持相对路径编译，如下

```bash
cd $GOPATH/src/github.com/http-helloworld/cmd/helloworld
ko publish ./
```

### 5. Ko resolve

 创建文件`deploy.yaml` ，yaml 内容如下

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  selector:
    matchLabels:
      foo: bar
  replicas: 1
  template:
    metadata:
      labels:
        foo: bar
    spec:
      containers:
      - name: hello-world
        # This is the import path for the Go binary to build and run.
        image: ko://github.com/http-helloworld/cmd/helloworld
        ports:
        - containerPort: 8080
```

然后执行 `ko resolve -f deploy.yaml` 结果如下，可以看到 `ko://github.com/http-helloworld/cmd/helloworld` 被替换成了 `zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648@sha256:21d352ec9f9079f8da4c91cfe2461df51a404c079f262390b19fff4cb2ce15a0`

```bash
$ ko resolve -f deploy.yaml

2021/04/23 22:09:19 Using base gcr.io/distroless/static:nonroot for github.com/http-helloworld/cmd/helloworld
2021/04/23 22:09:19 No matching credentials were found for "gcr.io/distroless/static", falling back on anonymous
2021/04/23 22:09:23 Building github.com/http-helloworld/cmd/helloworld for linux/amd64
2021/04/23 22:09:24 Publishing zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648:latest
2021/04/23 22:09:31 existing blob: sha256:5dea5ec2316d4a067b946b15c3c4f140b4f2ad607e73e9bc41b673ee5ebb99a3
2021/04/23 22:09:31 existing blob: sha256:d99fea081f251cc06ce68ce7cb224e2a0f63babd03ee9dd6bb03f6bf76dcb5a5
2021/04/23 22:09:32 existing blob: sha256:72164b581b02b1eb297b403bcc8fc1bfa245cb52e103a3a525a0835a58ff58e2
2021/04/23 22:09:32 existing blob: sha256:a3e2b18c53ecc2aab65b3b70e5c16486e48f76490febeb68d99aa18a48265b08
2021/04/23 22:09:32 zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648:latest: digest: sha256:21d352ec9f9079f8da4c91cfe2461df51a404c079f262390b19fff4cb2ce15a0 size: 750
2021/04/23 22:09:32 Published zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648@sha256:21d352ec9f9079f8da4c91cfe2461df51a404c079f262390b19fff4cb2ce15a0
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-world
spec:
  selector:
    matchLabels:
      foo: bar
  replicas: 1
  template:
    metadata:
      labels:
        foo: bar
    spec:
      containers:
        - name: hello-world
          # This is the import path for the Go binary to build and run.
          image: zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648@sha256:21d352ec9f9079f8da4c91cfe2461df51a404c079f262390b19fff4cb2ce15a0
          ports:
            - containerPort: 8080
```

### 6. Ko apply 

`ko apply -f xxx.yaml` 用法同 `kubectl apply -f xxx.yaml` ,**不同的是，`ko apply -f` 相当于先执行 `resolve` 将 `ko://xxx` 替换成镜像地址，然后再执行 `kubectl apply -f`  。**

```bash
$ cd $GOPATH/src/github.com/http-helloworld/config
$ ko apply -f ./

2021/04/23 22:18:10 Using base gcr.io/distroless/static:nonroot for github.com/http-helloworld/cmd/helloworld
2021/04/23 22:18:10 No matching credentials were found for "gcr.io/distroless/static", falling back on anonymous
2021/04/23 22:18:13 Building github.com/http-helloworld/cmd/helloworld for linux/amd64
2021/04/23 22:18:15 Publishing zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648:latest
2021/04/23 22:18:24 existing blob: sha256:a3e2b18c53ecc2aab65b3b70e5c16486e48f76490febeb68d99aa18a48265b08
2021/04/23 22:18:24 existing blob: sha256:d99fea081f251cc06ce68ce7cb224e2a0f63babd03ee9dd6bb03f6bf76dcb5a5
2021/04/23 22:18:24 existing blob: sha256:72164b581b02b1eb297b403bcc8fc1bfa245cb52e103a3a525a0835a58ff58e2
2021/04/23 22:18:24 existing blob: sha256:5dea5ec2316d4a067b946b15c3c4f140b4f2ad607e73e9bc41b673ee5ebb99a3
2021/04/23 22:18:25 zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648:latest: digest: sha256:21d352ec9f9079f8da4c91cfe2461df51a404c079f262390b19fff4cb2ce15a0 size: 750
2021/04/23 22:18:25 Published zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648@sha256:21d352ec9f9079f8da4c91cfe2461df51a404c079f262390b19fff4cb2ce15a0
deployment.apps/hello-world configured
```

### 7. 替换基础镜像

可以通过 `ko` 的 `config` 文件中的 `defaultBaseImage` 变量来设置基础镜像, 配置文件名为 `.ko.yaml` ,

ko 执行时默认会在当前目录下寻找 `.ko.yaml` 文件，也可以通过 环境变量 `KO_CONFIG_PATH` 来指定 `.ko.yaml` 的路径

```bash
# ~/.ko.yaml
defaultBaseImage: ubuntu
```

执行如下 

```bash

$ export KO_CONFIG_PATH=~/
$ ko apply  -f ./config

2021/04/23 22:28:50 Using base ubuntu for github.com/http-helloworld/cmd/helloworld
2021/04/23 22:29:00 Building github.com/http-helloworld/cmd/helloworld for linux/amd64
2021/04/23 22:29:01 Publishing zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648:latest
2021/04/23 22:29:14 existing blob: sha256:72164b581b02b1eb297b403bcc8fc1bfa245cb52e103a3a525a0835a58ff58e2
2021/04/23 22:29:14 existing blob: sha256:d99fea081f251cc06ce68ce7cb224e2a0f63babd03ee9dd6bb03f6bf76dcb5a5
2021/04/23 22:29:14 mounted blob: sha256:a70d879fa5984474288d52009479054b8bb2993de2a1859f43b5480600cecb24
2021/04/23 22:29:14 mounted blob: sha256:c4394a92d1f8760cf7d17fee0bcee732c94c5b858dd8d19c7ff02beecf3b4e83
2021/04/23 22:29:14 mounted blob: sha256:10e6159c56c084c858f5de2416454ac0a49ddda47b764e4379c5d5a147c9bf5f
2021/04/23 22:29:16 pushed blob: sha256:91d93e3477d55b71f5760478dcab690846ca5f76d92bbef874970460b3e73e5b
2021/04/23 22:29:17 zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648:latest: digest: sha256:623cb60ff10751f3f6a5f6570aaf5f49aee5fb6afc1ef5cfde4dd48a8b4d57df size: 1072
2021/04/23 22:29:17 Published zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648@sha256:623cb60ff10751f3f6a5f6570aaf5f49aee5fb6afc1ef5cfde4dd48a8b4d57df
deployment.apps/hello-world configured
```

还可以 通过`.ko.yaml` 文件中的 `baseImageOverrides` 对指定编译二进制替换基础镜像，如本示例中，设置如下：

```bash
# ~/.ko.yaml
defaultBaseImage: ubuntu
baseImageOverrides: 
  github.com/http-helloworld/cmd/helloworld: centos
```

 修改 `.ko.yaml` 后，执行命令

```bash
$ ko apply -f config

2021/04/23 22:38:14 Using base centos for github.com/http-helloworld/cmd/helloworld
2021/04/23 22:38:26 Building github.com/http-helloworld/cmd/helloworld for linux/amd64
2021/04/23 22:38:27 Publishing zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648:latest
2021/04/23 22:38:29 existing blob: sha256:72164b581b02b1eb297b403bcc8fc1bfa245cb52e103a3a525a0835a58ff58e2
2021/04/23 22:38:29 existing blob: sha256:d99fea081f251cc06ce68ce7cb224e2a0f63babd03ee9dd6bb03f6bf76dcb5a5
2021/04/23 22:38:29 mounted blob: sha256:7a0437f04f83f084b7ed68ad9c4a4947e12fc4e1b006b38129bac89114ec3621
2021/04/23 22:38:32 pushed blob: sha256:e909db5555a2c7310e605e60a47a98661c9a5c54d567f741a8d677f84c8a887f
2021/04/23 22:38:32 zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648:latest: digest: sha256:dc009335be23a9eee0235e425218f18a68c3210b00adac7cfe736d9bf38f4121 size: 752
2021/04/23 22:38:32 Published zhaojizhuang66/helloworld-cbcbba9849adcc25ce56a08dfd597648@sha256:dc009335be23a9eee0235e425218f18a68c3210b00adac7cfe736d9bf38f4121
deployment.apps/hello-world configured
```



