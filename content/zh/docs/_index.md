---
title: "欢迎使用 KubeDiag"
linkTitle: "文档"
menu:
  main:
    weight: 20
    pre: <i class='fas fa-book'></i>
---

Kubernetes 是一个生产级的容器编排引擎，但是 Kubernetes 仍然存在稳定性管理难和故障诊断成本高等问题。故障运维和集群检查操作在云原生体系下当前缺乏统一的管理标准，运维和配置管理的混乱导致了 Kubernetes 集群稳定性管理的成本居高不下，这种情况在多云和异构环境中尤为明显。

KubeDiag 为 Kubernetes 集群中的诊断运维管理提供了一套统一的编排框架。用户通过 Kubernetes 自定义资源可以定义运维操作、执行复杂的诊断运维流水线、通过报警自动触发诊断运维流水线。该系统通过下列自定义资源为用户提供了运维操作的自动化管理能力：

* Operation 用于定义故障运维和集群检查等操作。
* OperationSet 用于定义诊断运维流水线。
* Trigger 支持用户通过 Prometheus、Kafka 等系统自动触发诊断运维流水线。
* Diagnosis 中记录了一次诊断运维流水线的结果和状况。
