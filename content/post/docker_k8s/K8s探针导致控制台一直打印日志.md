---
title: "K8s探针导致控制台一直打印日志"
date: 2019-05-08T15:34:00+08:00
---
[Pod 的生命周期](https://kubernetes.io/zh/docs/concepts/workloads/pods/pod-lifecycle/)中介绍了容器探针，
根据[Configure Liveness and Readiness Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/)配置探针时，发现在容器的控制台一直在打印日志

    2019-05-08 15:41:16 - com.alibaba.dubbo.remoting.transport.AbstractServer.disconnected(AbstractServer.java:205) -  [DUBBO] All clients has discontected from /192.168.0.45:20001. You can graceful shutdown now., dubbo version: 2.6.2, current host: 192.168.0.45
    2019-05-08 15:41:16 - com.alibaba.dubbo.rpc.protocol.dubbo.DubboProtocol$1.disconnected(DubboProtocol.java:127) -  [DUBBO] disconnected from /x.x.x.x:5445
    ...
且每隔10s出现一次，端口也一直在加大，探针配置如下

        readinessProbe:
          tcpSocket:
            port: 20001
          initialDelaySeconds: 15
          periodSeconds: 10

服务也在正常运行，可以看出是探针每隔10s建立链接，然后就关闭，导致控制台一直在打印
