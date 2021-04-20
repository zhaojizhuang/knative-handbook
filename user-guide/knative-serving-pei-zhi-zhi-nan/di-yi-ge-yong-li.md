# 第一个 Serving 用例 Hello world

### 1. 创建一个名为 `service.yaml` 的文件，文件内容如下

```yaml
apiVersion: serving.knative.dev/v1 # Current version of Knative
kind: Service
metadata:
  name: helloworld-go # The name of the app
  namespace: default # The namespace the app will use
spec:
  template:
    spec:
      containers:
        - image: gcr.io/knative-samples/helloworld-go # Reference to the image of the app
          env:
            - name: TARGET # The environment variable printed out by the sample app
              value: "Go Sample v1"
```

### 2. 执行以下脚本

```text
kubectl apply --filename service.yaml
```

查看 Knative 工作负载的命令

```text
kubectl get ksvc helloworld-go
```

结果如下

```text
NAME            URL                                                LATESTCREATED         LATESTREADY           READY   REASON
helloworld-go   http://helloworld-go.default.34.83.80.117.xip.io   helloworld-go-96dtk   helloworld-go-96dtk   True
```

你的环境里如果没有修改过默认域名，URL 这一列应该显示的是 `http://helloworld-go.default.example.com` 

测试方式

{% tabs %}
{% tab title="istio gateway 配置了 xip.io" %}
istio gateway 的 external ip配置了 xip.io 的话, 其中 

```text
# curl http://helloworld-go.default.34.83.80.117.xip.io
Hello World: Go Sample v1!
```
{% endtab %}

{% tab title="默认情况下" %}


```text
curl -H "helloworld-go.default.example.com" http://<istio gateway的服务地址>:<istio gateway的服务端口>
```
{% endtab %}
{% endtabs %}



