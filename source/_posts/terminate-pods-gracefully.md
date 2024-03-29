---
title: "如何优雅地关闭 Pods"
date: 2021-11-23T09:41:31+08:00
typora-root-url: ../../static
categories:
series:
  - Kubernetes
  - CloudNative
tags:
  - Kubernetes
  - CloudNative
  - Pod
  - SIGTERM
  - timeout
keywords:
  - Kubernetes
  - 终止宽限期
  - termination grace period
  - 超时
  - timeout
  - 等待
  - Pod
---

## 理论

当我们在 Kubernetes 中触发停止一个 Pod 时，它会向每个容器中的进程发送一个 SIGTERM 信号，等待一段时间，这段时间被称为**终止宽限期**（termination grace period）。如果超过宽限期 Pod 中仍有容器进程没有被终止，会发送 SIGKILL 强行终止该进程。

所以作为开发者要注意配置：

1. 正确处理应用对 SIGTERM 信号的响应方式
2. 指定更加合理的宽限期

## 设置宽限期

在配置文件和命令行中都可以设置宽限期。

### 配置文件

使用 `kubectl explain` 配置我们可以知道默认的终止宽限期为30秒。

```text
$ kubectl explain pod.spec.terminationGracePeriodSeconds
KIND:     Pod
VERSION:  v1

FIELD:    terminationGracePeriodSeconds <integer>

DESCRIPTION:
     Optional duration in seconds the pod needs to terminate gracefully. May be
     decreased in delete request. Value must be non-negative integer. The value
     zero indicates delete immediately. If this value is nil, the default grace
     period will be used instead. The grace period is the duration in seconds
     after the processes running in the pod are sent a termination signal and
     the time when the processes are forcibly halted with a kill signal. Set
     this value longer than the expected cleanup time for your process. Defaults
     to 30 seconds.
```

通过配置 `pod.spec.terminationGracePeriodSeconds` 可以修改终止宽限期。例如：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  labels:
    app: busybox
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-ec", "sleep 1000"]
  terminationGracePeriodSeconds: 10 // change termination grace period
```

### 命令行

我们还可以在终止一个 Pod（或者 Deployment）的时候设置 `--grace-period` 来指定终止宽限期[^command-termination]。

```bash
kubectl delete pod <name> --grace-period=<seconds>
```

## 对无法优雅关闭的应用进行处理

如果有一些应用不是我们开发的，那么可能无法修改它处理 SIGTERM 信号的方式。这时该如何优雅地关闭该应用呢？这时我们需要用到配置 `pod.spec.containers.lifecycle.preStop`。我们先看一下[官方文档](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/#hook-handler-execution)的解释：

> `PreStop` hooks are not executed asynchronously from the signal to stop the Container; the hook must complete its execution before the TERM signal can be sent. If a `PreStop` hook hangs during execution, the Pod's phase will be `Terminating` and remain there until the Pod is killed after its `terminationGracePeriodSeconds` expires.

所以在 `preStop` hook 执行完之前，不会向容器发送 SIGTERM 信号，并且 Pod 会处于 Terminating 状态。

例如 Nginx 接收到 SIGTERM 信号会立即退出进程，必须通过在 `preStop` 中执行命令 `/usr/sbin/nginx -s quit` 来优雅地结束进程。指定触发 Pod 停止的时候容器中的，来执行命令关闭 Nginx。示例代码[^nginx-termination]：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
    lifecycle:
      preStop:
        exec:
          # SIGTERM triggers a quick exit; gracefully terminate instead
          command: ["/usr/sbin/nginx","-s","quit"]
```

## 总结

从运维角度我们可以在部署时通过配置文件指定 Pods 的终止宽限期，也可以在手动终止时指定宽限期。

从开发者的角度，我们应该保证自己的应用可以正确处理 Kubernetes 对 Pods 的终止动作。一般情况下我们需要正确处理 SIGTERM 信号；如果有些服务无法正确处理 SIGTERM 信号，但是提供了终止命令，我们可以在 `preStop` 中指定。

[^command-termination]: 细节可以参考[官方文档](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination-forced)。
[^nginx-termination]: 参考文章 [Graceful shutdown of pods with Kubernetes](https://pracucci.com/graceful-shutdown-of-kubernetes-pods.html)。
