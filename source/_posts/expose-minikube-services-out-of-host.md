---
title: "将 minikube 的服务暴露到宿主机外"
date: 2021-06-24T22:45:45+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - Kubernetes
  - minikube
---

## 背景

[minikube](https://minikube.sigs.k8s.io/) 是一款基于 Kubernetes 的定位于快速验证功能的小型容器编排环境。

由于它的定位特性，我们在使用中会发现 minikube 虚拟出了一个 IP 作为自身的节点 IP，该 IP 和宿主机不同。对于 `NodePort` 类型的 Service 也没有办法通过 `127.0.0.1` 访问。

## Host 内访问

> The minikube VM is exposed to the host system via a host-only IP address, that can be obtained with the `minikube ip` command.

我们必须通过 `minikube ip` 找到 minikube 的 IP 并通过它来[访问 `NodePort` 类型的 service](https://minikube.sigs.k8s.io/docs/handbook/accessing/#getting-the-nodeport-using-kubectl)。

另外可以留意到 `LoadBalancer` 类型的 service 在默认情况下 external IP 为 `<pending>`，为了要能够[访问到 service](https://minikube.sigs.k8s.io/docs/handbook/accessing/#loadbalancer-access)，需要通过 `minikube tunnel` 建立隧道和服务通信。在执行 `minikube tunnel` 后，可以发现 external IP 被设置成和 cluster IP 一样的值，这时候就可以通过 `<minikube-ip>:<service-port>` 来访问 service 了。

## Host 外访问

通过 minikube 可以很方便地在本机访问，同时避免了对宿主机端口的占用。但是也带来了另一个问题：无法直接通过访问宿主机的端口来访问 services 进行调试。

比如我的实验机是一台云服务虚拟机，我在自己的电脑上可以访问这台虚拟机，但是不能访问虚拟机上 minikube 暴露的服务。

虽然没有办法让 minikube 直接通过宿主机端口对外暴露 services，但是如果我们把问题换个角度思考，就很容易找到解决办法：如何将一个 service（不特定类型）暴露到本机。

这时候最简单的办法就是通过 `kubectl port-forward` 转发端口。

假设我有 `service/istio-ingressgateway`，service 监听 80 端口，我希望暴露在宿主机的 31303 端口，就可以使用以下命令[^ref]：·

```bash
kubectl port-forward --address 0.0.0.0 -n istio-system service/istio-ingressgateway 31303:80
```

[^ref]: 参考问答：[virtualbox - Expose host-only minikube for local network - Super User](https://superuser.com/questions/1634658/expose-host-only-minikube-for-local-network?newreg=d9bd6e22b82f4a0fbce2a9af14fddafb)

