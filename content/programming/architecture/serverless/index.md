---
title: "Serverless"
type: "docs"
weight: 1
---

serverless 是资源抽象的最高形式，屏蔽硬件、操作系统、运维，让开发者只需关注业务。AWS Lambda 使用 FireCracker，相对于容器更贴近硬件，运行效率更高。

## Serverless 和 FaaS

serverless 因 FaaS（Function as a service）才流行起来，但是 serverless 概念比 FaaS 更大，比如 腾讯云的 PostgreSQL for Serverless、阿里云的容器服务 Serverless。

## 冷启动

serverless 具有冷启动的天生缺陷，当服务容量已满，新进请求触发动态扩容，函数实例启动耗时比较长会影响当前请求，可以通过预热等方案解决。目前 AWS Lambda 冷启动时间缩减 80%，几乎没有影响。

## Flutter+Serverless 架构

架构演进

![image.png](image.png)

Dart 三端合一架构图

![image 1.png](image_1.png)

Dart 三端合一职能

![image 2.png](image_2.png)

渲染层实例

![image 3.png](image_3.png)
