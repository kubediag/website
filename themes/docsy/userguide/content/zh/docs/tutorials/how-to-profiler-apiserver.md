---
title: "如何使用 Go Profiler 剖析 APIServer"
linkTitle: "如何使用 Go Profiler 剖析 APIServer"
weight: 4
description: >
  创建 Go Profiler 帮助分析 APIServer 的运行状况。
---

在本教程中，你将部署一个配置为收集性能剖析文件数据的 Go 应用剖析集群的 APIServer，并将使用浏览器界面查看性能剖析文件数据。

## 准备工作

在教程开始前，您需要确定 kubernetes 集群中已经正确安装 KubeDiag，并且确认有以下资源

* ServiceAccount：`apiserver-profiler-sa`
* ClusterRole：`apiserver-profiler-role`
* ClusterRoleBinding：`apiserver-profiler-rolebinding`

您可以通过执行以下命令，确认以上资源的存在：

```bash
kubectl get serviceaccount apiserver-profiler-sa -n kubediag 
NAME                    SECRETS   AGE
apiserver-profiler-sa   1         79m

kubectl get clusterrole apiserver-profiler-role
NAME                      CREATED AT
apiserver-profiler-role   2021-07-13T06:55:48Z

kubectl get clusterrolebinding apiserver-profiler-rolebinding
NAME                             ROLE                                  AGE
apiserver-profiler-rolebinding   ClusterRole/apiserver-profiler-role   79m
```

如果不存在，执行以下命令创建：

```bash
kubectl apply -f https://raw.githubusercontent.com/kubediag/kubediag/master/config/rbac/apiserver_profiler_viewer_serviceaccount.yaml  -n kubediag 

kubectl apply -f https://raw.githubusercontent.com/kubediag/kubediag/master/config/rbac/apiserver_profiler_viewer_clusterrole.yaml  -n kubediag 

kubectl apply -f https://raw.githubusercontent.com/kubediag/kubediag/master/config/rbac/apiserver_profiler_viewer_serviceaccount.yaml  -n kubediag 
```

## 教程目标

阅读本教程后，你将熟悉以下内容：

* 如何创建 Go Profiler 剖析集群的 APIServer 的性能
* 怎样查看性能剖析结果

## 创建 Go Profiler

首先创建一个引用 Go Profiler Operation 的 Operationset

```yaml
apiVersion: diagnosis.kubediag.org/v1
kind: OperationSet
metadata:
  name: go-profiler
spec:
  adjacencyList:
  - id: 0
    to:
    - 1
  - id: 1
    operation: go-profiler
```

下载上面的例子并保存为文件 `go-profiler-operationset.yaml`。

使用 `kubectl apply` 来创建这个Go Profiler Operationset

```bash
kubectl apply -f go-profiler-operationset.yaml
```

然后创建 Diagnosis，这个过程需要获取由 `apiserver-profiler-sa` ServiceAccount 生成的 Secret 资源，用以下命令获取 Secret 的名字：

```bash
kubectl get secret -n kubediag | grep apiserver-profiler-sa-token | awk '{print $1}'
```

使用如下示例创建一个 Go Profiler Diagnosis。注意替换其中的 `<secret-name>`、`<node-name>`、`<apiserver-url>` 地址，其中 `<secret-name>` 是由 `apiserver-profiler-sa` ServiceAccount 生成的 Secret 对象的名字，`<node-name>` 是 KubeDiag Agent 所运行的节点名字，`<apiserver-url>` 是当前集群的 APIServer URL。

```yaml
apiVersion: diagnosis.kubediag.org/v1
kind: Diagnosis
metadata:
  name: go-profiler
spec: 
  parameters:
    param.diagnoser.runtime.go_profiler.expiration_seconds: "7200"
    param.diagnoser.runtime.go_profiler.type: Heap
    param.diagnoser.runtime.go_profiler.source: <apiserver-url>
    param.diagnoser.runtime.go_profiler.tls.secret_reference.namespace: kubediag
    param.diagnoser.runtime.go_profiler.tls.secret_reference.name: <secret-name>
  operationSet: go-profiler
  nodeName: <node-name>
```

下载上面的例子并保存为文件 `apiserver-profiler.yaml`。

使用 `kubectl apply` 来创建这个Go Profiler Diagnosis。

```bash
kubectl apply -f apiserver-profiler.yaml
```

上面这个示例中创建了一个引用 `go-profiler` Operationset 的 Go Profiler Diagnosis，它将被 `<node-name>` 上运行的 KubeDiag Agent 监测到并进行同步。`tls.secret_reference.namespace` 与 `tls.secret_reference.name` 共同指定的 Secret 将被用于获取 `token` 和 `ca.crt` 进行连接到 `<apiserver-url>` 的认证。Agent 在认证通过后下载 Go Profiler 性能剖析文件于本地保存。这里性能剖析的 Type 是 `Heap`，代表 APIServer 的堆信息将在此刻收集。

查看 Diagnosis 的状态，性能剖析的执行结果以键值对的形式被同步到 `.status.operationResults` 字段，其中键为 `diagnoser.runtime.go_profiler.result.endpoint`：

```yaml
status:
  operationResults:
      diagnoser.runtime.go_profiler.result.endpoint: Visit http://10.0.2.15:39941,
        this server will expire in 7200 seconds.
```

## 查看性能剖析结果

在浏览器中打开 `10.0.2.15:45771`，显示 Profiler 界面，即可查看 APIServer 的堆分析结果与火焰图等。
