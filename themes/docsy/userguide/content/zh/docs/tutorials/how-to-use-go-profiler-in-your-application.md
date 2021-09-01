---
title: "剖析 Go 应用的性能"
linkTitle: "剖析 Go 应用的性能"
weight: 3
description: >
  创建 Go Profiler 来剖析应用的性能并查看结果。
---

本教程介绍如何修改 Go 应用以捕获性能剖析数据，并查看剖析数据。

## 教程目标

阅读本教程后，你将熟悉以下内容：

* 如何创建 Go Profiler 剖析你的应用的性能
* 怎样查看性能剖析结果

## 创建 Go Profiler

假如你已经有一个 HTTP Server 的 Go 应用，你可以在代码中加入如下引用。

```go
import _ "net/http/pprof"
```

如果你想使用一个全新的 Go 应用，也可以复制如下代码生成一个：

```go
package main

import (
    "net/http"
    _ "net/http/pprof"
)

func main() {
    http.ListenAndServe(":9090", nil)
}
```

运行这个应用，然后使用如下示例创建一个 Operationset 与 Diagnosis 进行性能剖析。注意创建前修改你的 `<source-ip>` 与 `<node-name>`，其中 `<source-ip>` 是你的 Go 应用访问 IP，`<node-name>` 是运行了 KubeDiag Agent的节点 Name。

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
---
apiVersion: diagnosis.kubediag.org/v1
kind: Diagnosis
metadata:
  name: go-profiler
spec: 
  parameters:
    param.diagnoser.runtime.go_profiler.type: Heap
    param.diagnoser.runtime.go_profiler.source: <source-ip>:9090
  operationSet: go-profiler
  nodeName: <node-name>
```

查看 Diagnosis 的状态，性能剖析的执行结果被同步到 `.status.profilers` 字段：

```yaml
status:
  operationResults:
    diagnoser.runtime.go_profiler.result.endpoint: Visit http://10.0.2.15:45778,
      this server will expire in 7200 seconds.
```

## 查看性能剖析结果

在浏览器中打开 `10.0.2.15:45778`，显示 Profiler 界面，即可查看此应用的堆分析结果与火焰图等。
