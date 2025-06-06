---
title: "kubernetes"
type: "docs"
weight: 3
---

## 常用命令

### kubectl

```shell
#列出所有支持的API资源
kubectl api-resources
#查看集群信息
kubectl cluster-info

#列出所有节点
kubectl get nodes
#列出所有命名空间
kubectl get namespaces
#创建命名空间
kubectl create namespace my-namespace

#查看集群的所有资源
kubectl get all --all-namespaces
kubectl get pods --all-namespaces
kubectl get pods --all-namespaces -o wide
kubectl get pods -n <namespace>
#查看系统pods
kubectl get pods -n kube-system

#查看 pod 的日志
kubectl logs -n <namespace> <pod-name> 
kubectl logs -n kube-system pod/kube-proxy-www59
#查看前一个容器的日志，对于调试崩溃或重启的容器特别有用。
kubectl logs -n <namespace> <pod-name> --previous

#查看详细信息
kubectl describe node ip-172-16-1-111.compute.internal
kubectl describe pod -n <namespace> <pod-name>
kubectl describe service openldap -n kubesphere-system
kubectl describe deployment nginx-deployment

#创建、删除资源
kubectl apply -f <file.yaml>
kubectl delete -f <file.yaml>

# 执行命令进入到某个 pod 的容器中
kubectl exec -it <pod-name> -n <namespace> -- /bin/bash
# 编辑 kube-proxy 配置
kubectl edit configmap kube-proxy -n kube-system

#重启pod
kubectl delete pod -n <namespace> <pod-name>
kubectl delete pods -n kube-system -l k8s-app=kube-proxy
kubectl delete pods -n kube-flannel -l k8s-app=flannel
kubectl -n kube-system rollout restart deployment coredns

#使用定时任务模版创建任务
kubectl create job --from=cronjob/test test-1 -n namespace

#flannel网络插件
kubectl get pod -n kube-flannel -l app=flannel
kubectl get daemonset -n kube-flannel kube-flannel-ds
kubectl edit configmap kube-flannel-cfg -n kube-flannel
kubectl delete pod -l app=flannel -n kube-flannel

#服务
kubectl get svc --all-namespaces

kubectl edit service openldap -n kubesphere-system
kubectl delete service openldap -n kubesphere-system

#工作负载
kubectl get deployments --all-namespaces
kubectl set image deployment/nginx-deployment nginx=nginx:1.19
kubectl rollout undo deployment/nginx-deployment
kubectl scale deployment/nginx-deployment --replicas=5
kubectl delete deployment nginx-deployment

#重启
kubectl rollout restart deployment ks-apiserver -n kubesphere-system

kubectl get deployments --all-namespaces -o yaml > all_deployments.yaml
kubectl get jobs --all-namespaces -o json > all_jobs.json
kubectl get cronjobs --all-namespaces -o json > all_cronjobs.json

#查看kubelet日志
journalctl -u kubelet -f
```

### containerd

containerd 是 k8s 默认的容器运行时，它提供了容器的生命周期管理、镜像管理等功能。

```shell
#启动和停止
sudo systemctl start containerd
sudo systemctl stop containerd
sudo systemctl status containerd
#查看日志
journalctl -u containerd

#容器管理
ctr containers list
ctr container info <container-id>

#容器操作
ctr container create <image> <container-id>
ctr run <options> <image> <container-id> <command> [<args>...]
ctr task start <container-id>
ctr task status <container-id>
ctr task kill <container-id>
ctr container delete <container-id>

#镜像管理
ctr images list
ctr images pull <image-name>
ctr images remove <image-name>
ctr images info <image-name>

#运行容器
ctr run <options> <image> <container-id> <command>
ctr run --tty --rm alpine:latest mycontainer /bin/sh
```

### crictl

crictl 是 k8s CRI（容器运行时接口）的命令行工具。

```shell
#列出正在运行的容器
sudo crictl ps

#列出所有容器（包括已停止的容器）
sudo crictl ps -a

#查看容器详细信息
sudo crictl inspect <容器ID>

#列出所有镜像
sudo crictl images

#查看镜像详细信息
sudo crictl inspecti <镜像ID或名称>

#删除容器
sudo crictl rm <容器ID>

#拉取镜像
sudo crictl pull <镜像名称>

#删除镜像
sudo crictl rmi <镜像ID或名称>

#查看容器日志
sudo crictl logs <容器ID>

#启动容器
sudo crictl start <容器ID>

#停止容器
sudo crictl stop <容器ID>

#运行一个交互式 shell
sudo crictl exec -it <容器ID> sh
```

## 安装kubernetes

安装请参考文档 [镜像](../mirror/#kubernetes)。

### 作为控制节点

```shell
#前置：calico依赖内核模块
lsmod | grep -E 'ip_tables|iptable_nat|ip6_tables|ip6table_nat|ip_set|xt_set|ipip|nf_conntrack|ip6_tunnel|tun|br_netfilter'
#临时启用
modprobe ip_tables iptable_nat ip6_tables ip6table_nat ip_set xt_set ipip nf_conntrack ip6_tunnel tun br_netfilter
#长期启用
sudo tee /etc/modules-load.d/k8s.conf > /dev/null <<EOF
ip_tables
iptable_nat
ip6_tables
ip6table_nat
ip_set
xt_set
ipip
nf_conntrack
ip6_tunnel
tun
br_netfilter
EOF
sudo systemctl restart systemd-modules-load.service
#启用IP转发
echo -e "net.ipv4.ip_forward = 1\nnet.ipv6.conf.all.forwarding = 1\nnet.bridge.bridge-nf-call-iptables = 1\nnet.bridge.bridge-nf-call-ip6tables = 1" | sudo tee -a /etc/sysctl.conf
sysctl -p

#推荐使用IPVS
#calico默认使用ip_tables，如果是最新系统可能只有nf_tables，需要在集群统一使用nf_tables
#calico还需要注意网卡名称
lsmod | grep -E 'ip_vs|ip_tables|iptable_nat|ip6_tables|ip6table_nat|nf_tables|nf_nat|ip_set|xt_set|ipip|nf_conntrack|nf_defrag_ipv4|nf_defrag_ipv6|ip6_tunnel|tun|br_netfilter'
#临时启用
modprobe ip_tables iptable_nat ip6_tables ip6table_nat nf_tables nf_nat ip_set xt_set ipip nf_conntrack nf_defrag_ipv4 nf_defrag_ipv6 ip6_tunnel tun br_netfilter
#长期启用
sudo tee /etc/modules-load.d/k8s.conf > /dev/null <<EOF
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
ip_tables
iptable_nat
ip6_tables
ip6table_nat
nf_tables
nf_nat
nf_conntrack
nf_defrag_ipv4
nf_defrag_ipv6
ip_set
xt_set
ipip
ip6_tunnel
tun
br_netfilter
EOF
sudo systemctl restart systemd-modules-load.service
#启用IP转发
echo -e "net.ipv4.conf.all.arp_ignore=1\nnet.ipv4.conf.all.arp_announce=2\nnet.ipv4.ip_forward = 1\nnet.ipv6.conf.all.forwarding = 1\nnet.bridge.bridge-nf-call-iptables = 1\nnet.bridge.bridge-nf-call-ip6tables = 1" | sudo tee -a /etc/sysctl.conf
sysctl -p

yum install -y ipset ipvsadm

# 旧系统从iptables切换到nftables兼容模式（不推荐）
# sudo yum install -y iptables nftables iptables-nft
# 手动创建备选方案配置
# sudo update-alternatives --install /usr/sbin/iptables iptables /usr/sbin/iptables-nft 100
# sudo update-alternatives --install /usr/sbin/ip6tables ip6tables /usr/sbin/ip6tables-nft 100
# sudo update-alternatives --install /usr/sbin/arptables arptables /usr/sbin/arptables-nft 100
# sudo update-alternatives --install /usr/sbin/ebtables ebtables /usr/sbin/ebtables-nft 100
# 设置默认选项
# sudo update-alternatives --set iptables /usr/sbin/iptables-nft
# sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-nft
# sudo update-alternatives --set arptables /usr/sbin/arptables-nft
# sudo update-alternatives --set ebtables /usr/sbin/ebtables-nft
# 如果有问题切换回来
# sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
# sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
# sudo update-alternatives --set arptables /usr/sbin/arptables-legacy
# sudo update-alternatives --set ebtables /usr/sbin/ebtables-legacy
# 验证配置
# sudo iptables -V
# 应输出: iptables v1.8.x (nf_tables)

# 新系统从nftables切换到iptables模式
# sudo yum install iptables-legacy -y
# sudo systemctl enable iptables --now

#初始化控制节点
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

#配置文件
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

#安装网络插件flannel，简单适合初学者，依赖少，大多数环境都能跑起来，默认网段10.244.0.0/16
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
#或者安装网络插件calico，性能好，功能强大
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

#移除控制节点标签，使其同时作为工作节点（分布式集群不推荐）
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

#设置存储（配置文件见 存储 章节）
kubectl apply -f storageclass.yaml
#设置默认存储
kubectl patch storageclass hostpath -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'

#重置，危险！
sudo kubeadm reset -f
rm -rf ~/.kube /etc/kubernetes /var/lib/etcd /var/lib/kubelet/* /etc/cni/net.d
#清理Calico残留（可选）
rm -rf /var/run/calico /etc/calico
#重置iptables
iptables -P INPUT ACCEPT && iptables -P FORWARD ACCEPT && iptables -P OUTPUT ACCEPT && iptables -F && iptables -X && iptables -t nat -F && iptables -t nat -X && iptables -t mangle -F && iptables -t mangle -X && iptables -t raw -F && iptables -t raw -X && iptables -Z && ip6tables -P INPUT ACCEPT && ip6tables -P FORWARD ACCEPT && ip6tables -P OUTPUT ACCEPT && ip6tables -F && ip6tables -X && ip6tables -t nat -F && ip6tables -t nat -X && ip6tables -t mangle -F && ip6tables -t mangle -X && ip6tables -t raw -F && ip6tables -t raw -X && ip6tables -Z
```

calico兼容多种网卡
```
env:
  - name: IP_AUTODETECTION_METHOD
    value: "interface=ens.*|eth.*"
```

### 工作节点

```shell
#生成加入命令（控制节点）
kubeadm token create --print-join-command

#加入集群（工作节点）
kubeadm join <control-plane-endpoint> --token <token> --discovery-token-ca-cert-hash <hash>

#删除工作节点（控制节点）
kubectl delete node <节点名>
```

## 架构

![k8s](k8s.png)

Kubernetes（简称K8s）是一个用于自动化部署、弹性伸缩、负载均衡、服务发现和管理容器化应用程序的开源系统。

机器类型：

1. Control Plane（控制平面，奇数数量）：
    - API Server：k8s 集群的入口点，接收所有操作请求（kubectl、dashboard、controller等），并更新etcd中的状态。
    - Controller Manager：运行各种控制器，包括Node Controller（节点监控）、Replication Controller（副本管理）、Service Account Controller（权限分配）等，负责确保集群状态与预期状态的一致性。
    - Scheduler：根据资源、约束和策略为新建 Pod 选择最佳工作节点。
    - etcd：一个高可用的键值存储系统，保存所有集群状态和配置数据（如资源对象信息、配置项、服务发现等）。

2. Worker Node（工作节点/计算节点）：
    - Kubelet： Pod 的生命周期管理，与 API Server 通信，负责本节点的 Pod 创建、监控与回收。
    - Kube-Proxy：实现 Service 的通信逻辑，支持 iptables/IPVS 模式，处理请求转发、负载均衡。
    - Container Runtime：运行容器的底层引擎，常见的有 Docker、containerd（默认） 等。

3. Edge Node（边缘节点）：控制平面在中心云，工作负载运行在物理/弱网边缘设备，KubeEdge/SuperEdge 等边缘计算框架。

资源类型：

1. Namespace：用于将集群中的资源划分为多个虚拟集群，实现资源的隔离。

2. Pod：k8s 中的基本调度单元，包含一个或多个容器，这些容器共享网络和存储。

3. Deployment：用于管理 Pod 的副本和更新策略，确保应用程序的高可用性和可扩展性。

4. DaemonSet：用于确保某些 Pod 在多个节点上都运行，例如日志收集、监控、代理等。

5. ReplicaSet：负责维护一定数量的 Pod 副本。Deployment 的底层就是通过 ReplicaSet 实现副本管理的，但通常用户不会直接使用。

6.  Job/CronJob：一次性任务/定时任务。

7.  Service：定义了一组 Pod 的访问策略，提供负载均衡和服务发现。

8.  Ingress：管理外部访问 k8s 集群中服务的策略，通常与负载均衡器结合使用。

9.  ConfigMap/Secret：用于存储和管理应用程序配置/敏感数据。

10. Volume：提供持久化存储解决方案，确保数据在容器重启或迁移时不丢失。

## 系統 Pod

k8s 集群初始化后，控制平面（Control Plane）和工作节点（Worker Node）上会自动运行一些关键的系统容器（通常称为 系统 Pod，而非独立容器），这些容器支撑着 k8s 的核心功能。以下是典型的初始容器及其作用：

### 1. 控制平面（Control Plane）的组件
控制平面的组件通常以 Pod 形式运行在 `kube-system` 命名空间下（如果是通过 `kubeadm` 部署的集群）。

#### (1) `etcd`
- 作用：分布式键值存储数据库，保存整个集群的状态（如 Pod、Service、ConfigMap 等资源对象的元数据）。
- 关键性：集群的“大脑”，所有组件通过它获取和更新状态。
- 运行方式：通常以单 Pod 运行（生产环境建议多副本实现高可用）。

#### (2) `kube-apiserver`
- 作用：k8s 的 API 入口，接收并验证所有 REST 请求（如 `kubectl` 命令），处理集群的增删改查操作。
- 关键性：唯一直接与 `etcd` 通信的组件，是集群控制平面的核心。

#### (3) `kube-controller-manager`
- 作用：运行一系列控制器（Controller），确保集群状态与预期一致。例如：
  - Node Controller（监控节点状态）
  - Replication Controller（维护 Pod 副本数）
  - Endpoints Controller（维护 Service 的 Endpoints）
- 关键性：自动化修复集群状态的核心组件。

#### (4) `kube-scheduler`
- 作用：负责将新创建的 Pod 调度到合适的节点上（基于资源需求、节点负载、亲和性等策略）。
- 关键性：决定 Pod 在哪个节点运行。

#### (5) `cloud-controller-manager`（仅限云环境）
- 作用：与云服务商（如 AWS、GCP、Azure）交互，管理负载均衡、存储卷、节点实例等云资源。
- 关键性：解耦 k8s 与底层云平台的依赖。

---

### 2. 工作节点（Worker Node）的组件
工作节点上运行的组件负责与控制平面通信，并管理容器生命周期。

#### (1) `kubelet`
- 作用：
  - 管理节点上的 Pod 生命周期（启动、停止、监控容器）。
  - 向控制平面报告节点和 Pod 的状态。
- 关键性：节点上最重要的进程，直接与容器运行时（如 Docker、containerd）交互。
- 注意：`kubelet` 通常以系统服务（非容器）形式运行，但会被视为节点的核心组件。

#### (2) `kube-proxy`
- 作用：
  - 维护节点上的网络规则（如 iptables/IPVS），实现 Service 的负载均衡和流量转发。
  - 确保 Pod 可以通过 Service 的虚拟 IP 或 DNS 名称相互通信。
- 关键性：实现 k8s 服务发现和负载均衡的核心。

#### (3) 容器运行时（Container Runtime）
- 常见实现：Docker Engine、containerd、CRI-O。
- 作用：负责拉取镜像、启动/停止容器、管理容器生命周期。
- 关键性：k8s 依赖容器运行时执行底层容器操作。

---

### 3. 其他可能的附加组件
以下组件可能在集群初始化时自动部署（取决于部署工具，如 `kubeadm`、托管集群等）：

#### (1) `CoreDNS`
- 作用：为集群提供 DNS 服务，解析 Service 和 Pod 的域名。
- 部署方式：通常以 Pod 形式运行在 `kube-system` 命名空间。

#### (2) `metrics-server`
- 作用：收集集群资源使用指标（如 CPU、内存），供 `kubectl top` 和 HPA（自动扩缩容）使用。
- 部署方式：可能需要手动部署（部分工具会默认安装）。

### CNI网络插件

CNI网络插件负责Pod网络的创建和管理。

1. Flannel：支持 VXLAN/host-gw/IPIP，简单稳定，默认使用 VXLAN 封装，性能较差、兼容性好。
2. Calico：支持 BGP/VXLAN/IPIP/WireGuard/eBPF（v3.20+），支持网络策略和安全性等高级网络功能，比较复杂。
3. Cilium：依赖eBPF，高性能，安全性强，对内核要求较高，最复杂。

| 技术        | 优点                                                         | 缺点                                                           | 常见应用/备注                                |
|-------------|--------------------------------------------------------------|----------------------------------------------------------------|----------------------------------------------|
| **iptables** | ✅ 成熟稳定、广泛支持                                         | ❌ 表规则多时效率低，配置复杂                                 | kube-proxy 默认模式                          |
| **nftables** | ✅ 替代 iptables，结构更清晰                                  | ❌ 一些旧软件兼容性差                                          | Debian / Ubuntu 等现代发行版默认使用         |
| **IPVS**     | ✅ 高性能，支持连接追踪，适合高并发                         | ❌ 配置和调试复杂                                               | kube-proxy 推荐在高并发集群使用             |
| **eBPF**     | ✅ 内核态执行、超高性能、灵活编程                          | ❌ 依赖高版本内核、学习曲线陡峭                                | Cilium、网络策略、Service Mesh              |
| **VXLAN**    | ✅ 支持跨主机通信，网络可扩展                               | ❌ 基于 UDP 封装，性能损耗较高，可能丢包                     | Flannel、Calico 的 overlay 模式             |
| **IPIP**     | ✅ 相比 VXLAN 封装效率更高                                  | ❌ 云环境常有限制，容易遇到 MTU 问题                          | Flannel、Calico（轻量 overlay）             |
| **WireGuard**| ✅ 安全、轻量、现代加密方式                                 | ❌ 不支持负载分发                                               | Calico 加密节点通信、集群间 VPN              |
| **BGP**      | ✅ 真正的 L3 路由互通，无需封装                             | ❌ 配置复杂、依赖网络控制器                                   | Calico 发布路由，裸机集群或公网互通场景     |

- **小型测试集群**：kube-proxy: iptables + Flannel VXLAN/IPIP  简单、兼容性好
- **中型生产集群**：kube-proxy: IPVS + Calico BGP  无封装、高性能、支持直连
- **高性能场景**：kube-proxy: none + Cilium eBPF  极限性能、复杂微服务通信，用eBPF替代kube-proxy
- **安全优先**：kube-proxy: IPVS + Calico WireGuard  加密通信、路由灵活性
- **操作系统使用nftables**：Flannel、Calico不支持nftables，建议切换成 iptables-legacy 或使用 eBPF绕过iptables/nftables

### 重要组件

除了核心组件之外，k8s 中还有许多重要的组件和工具，它们在不同的场景和需求下发挥着重要作用：

1. Helm：
    - k8s 的包管理工具，用于简化应用程序的部署和管理。Helm 使用“charts”来定义、安装和升级 k8s 应用程序。

2. Prometheus：
    - 用于监控和告警的系统，可以与 k8s 集成，以收集集群和应用程序的性能数据。

3. Grafana：
    - 开源的可视化工具，与 Prometheus 配合使用，提供丰富的监控和可视化能力。

4. kube-state-metrics：
    - 用于暴露 k8s 集群状态指标，供 Prometheus 采集。

5. Fluentd：
    - 日志收集和聚合工具，可以将 k8s 集群中的日志收集并发送到不同的存储后端。

6. Jaeger 或 Zipkin：
    - 分布式追踪系统，用于监控和调试微服务架构中的延迟和性能问题。

7. Cert-Manager：
    - 用于在 k8s 集群中自动化管理 TLS 证书，常用于自动申请和续订 Let’s Encrypt 证书。

8.  Kubeflow：
    - 用于在 k8s 上运行和部署机器学习工作负载的工具包。

9.  Velero：
    - 用于备份和恢复 k8s 集群资源和持久化卷的工具。

10. Kubernetes Dashboard：
    - 一个基于 Web 的用户界面，用于管理和监控 k8s 集群。

11. Operators：
    - 用于将复杂的应用程序管理自动化的 k8s 扩展，使用自定义控制器来管理特定的应用程序或服务。

12. Kustomize：
    - 原生于 k8s 的配置管理工具，允许用户声明性地管理 k8s 对象配置。

这些组件和工具在不同的应用场景下发挥着重要作用，可以极大地增强 k8s 的功能和用户体验。

### 其它组件

除了之前提到的组件和工具，还有一些其他的重要组件和工具，它们在特定的用例中也非常有用：

1. Minikube：
    - 本地运行 k8s 的工具，适用于开发和测试环境。

2. Kind (Kubernetes IN Docker)：
    - 在 Docker 容器中运行本地 k8s 集群的工具，常用于测试和 CI/CD 环境。

3. Linkerd：
    - 轻量级服务网格，用于提供服务间通信的可观察性、安全性和可靠性。

4. Istio：
    - 功能丰富的服务网格，提供流量管理、安全性、策略控制和可观察性。

5. Keda (Kubernetes Event-driven Autoscaling)：
    - 允许基于事件的自动扩展，为应用程序提供基于事件的自动缩放能力。

6.  Argo CD：
    - 声明式的 GitOps 继续交付工具，帮助管理 k8s 应用程序的持续交付。

7.  Tekton：
    - 云原生的 CI/CD 系统，提供构建、测试和部署应用程序的流水线。

8.  Velero：
    - k8s 的备份和恢复工具，用于保护和恢复集群资源和持久化存储。

9.  OpenEBS：
    - 云原生的持久化存储解决方案，提供基于容器的存储编排。

10. Longhorn：
    - CNCF 孵化的分布式块存储系统，用于 k8s 环境。

11. Kong for Kubernetes：
    - 云原生 API 网关，提供服务管理、路由、负载均衡和安全性。

12. Harbor：
    - 云原生的容器镜像仓库，提供镜像管理、安全扫描和访问控制功能。

13. Kiali：
    - 服务网格（如 Istio）的可观察性和可管理性工具，提供仪表盘和可视化功能。

这些组件和工具可以根据具体需求选择和使用，进一步增强 k8s 的功能和管理能力。

## 推荐容量

根据 k8s 官方文档，集群的推荐容量上限：

1. 节点数（Nodes）：5000 个节点
2. Pod 数（Pods）：每个集群最多 150,000 个 Pod
3. 每节点 Pod 数：每个节点最多 110 个 Pod
4. 每节点容器数：每个节点最多 300 个容器
5. 每个节点的服务（Services）数：每个节点最多 10,000 个服务
6. 每个集群的 ConfigMap 数：最多 10,000 个 ConfigMap
7. 每个集群的 Secrets 数：最多 10,000 个 Secrets
8. 每个节点的 PersistentVolumes 数：最多 2000 个 PersistentVolumes

这些限制是为了确保集群的稳定性和可扩展性，同时这些数字是经过严格测试和验证的，适用于大多数生产环境。

## 控制器

在 k8s 中，有几种常用的控制器用于管理和部署容器化应用程序。以下是主要的控制器及其用途：

### 1. Deployment

- 用途：用于声明和管理一组 Pod 的副本。
- 功能：支持滚动更新、回滚、扩展和缩减副本数。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
name: nginx-deployment
spec:
replicas: 3
selector:
  matchLabels:
    app: nginx
template:
  metadata:
    labels:
      app: nginx
  spec:
    containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
          - containerPort: 80
```

### 2. ReplicaSet

- 用途：确保指定数量的 Pod 副本始终在运行。
- 功能：主要用于 Deployment 内部，直接使用的场景较少。

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

### 3. StatefulSet

- 用途：用于管理有状态应用程序，提供稳定的标识、持久存储和有序部署。
- 功能：保证 Pod 的顺序启动和停止，持久卷绑定到特定的 Pod。

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

### 4. DaemonSet

- 用途：在集群的每个节点上运行一个 Pod 的副本。
- 功能：确保所有（或特定节点上的）节点都运行一个 Pod 实例，适用于日志、监控等系统服务。

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      name: fluentd
  template:
    metadata:
      labels:
        name: fluentd
    spec:
      containers:
        - name: fluentd
          image: fluentd:v1.2.0
```

### 5. Job

- 用途：用于一次性任务，即执行完成后终止的任务。
- 功能：保证任务的完成，支持并行任务。

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
        - name: pi
          image: perl
          command: [ "perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)" ]
      restartPolicy: Never
```

### 6. CronJob

- 用途：用于定时任务，类似于 Linux 的 cron 任务。
- 功能：按照预定的时间表运行任务。

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: hello
              image: busybox
              args:
                - /bin/sh
                - -c
                - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

### 7. HorizontalPodAutoscaler (HPA)

- 用途：根据 CPU 利用率或其他指标自动扩展或缩减 Pod 数量。
- 功能：动态调整 Pod 副本数以应对负载变化。

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

这些控制器各有用途，选择合适的控制器可以更好地管理和部署你的应用程序。

## 流量

在 k8s 中处理流量的方式主要有以下几种：

1. Service：
    - ClusterIP：默认类型，只能在集群内部访问，提供内部服务的负载均衡。
    - NodePort：在每个节点上开放一个端口，允许外部流量访问集群中的服务。
    - LoadBalancer：创建一个外部负载均衡器，允许外部流量通过负载均衡器访问集群中的服务。
    - Headless Service：不分配 ClusterIP，直接将流量发送到后端 Pod，适用于需要直接访问 Pod 的场景，如状态服务。

2. Ingress：
    - Ingress 是一种 API 对象，提供 HTTP 和 HTTPS 路由功能。通过配置 Ingress Controller，可以定义路由规则，将流量从外部路由到集群内部的服务。

3. Ingress Controller：
    - Ingress Controller 是实现 Ingress 资源的控制器。常用的 Ingress Controller 有 NGINX Ingress Controller、Traefik、HAProxy 等。

4. NetworkPolicy：
    - 用于定义网络访问控制规则，以限制不同 Pod 之间的流量。可以实现细粒度的安全策略，控制哪些 Pod 可以访问其他 Pod。

5. Service Mesh：
    - 通过 Service Mesh（如 Istio、Linkerd）提供更高级的流量管理功能，包括流量路由、负载均衡、故障恢复、监控和安全控制等。

6. Custom Resource Definitions (CRDs)：
    - 通过自定义资源定义扩展 k8s 的功能，可以定义自定义的流量管理策略，满足特定需求。

这些方式可以单独使用，也可以结合使用，根据实际需求和场景选择最合适的流量处理方式。

### API入口

1. Ingress Controller：最常用的解决方案
   - Nginx Ingress Controller
   - Traefik
   - HAProxy
   - Kong

2. Service Mesh：
   - Istio
   - Linkerd

3. API Gateway：
   - Kong
   - Ambassador
   - Gloo

4. Gateway API：k8s新一代的入口资源，比Ingress更强大

Ingress Controller是最推荐的方案，它通过主机名和路径规则将流量转发到不同的服务，支持多域名配置，还能集成SSL/TLS证书自动管理。

```
# 安装ingress-nginx
helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --version 4.2.5
# 监听宿主机端口，ingress默认运行在隔离环境内，需要手动绑定
# 在spec.template.spec下添加hostNetwork: true
kubectl edit deployment ingress-nginx-controller -n ingress-nginx
```

## 存储

StorageClass、PersistentVolume（PV）和 PersistentVolumeClaim（PVC）是用于管理存储资源的关键概念。

StorageClass（存储类）

StorageClass 用于定义存储的类型和质量，例如性能特征和存储类型（如 SSD、HDD）。它允许用户指定存储的要求和动态提供存储卷。通过不同的存储类，你可以选择不同的存储提供商和配置。

storageclass.yaml

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: hostpath
  labels:
    storage.kubesphere.io/storagetype: FileStorage
  annotations:
    storageclass.kubernetes.io/is-default-class: 'true'
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

aws的ebs磁盘

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: aws-ebs
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: ebs.csi.aws.com
parameters:
  type: aws-ebs
volumeBindingMode: WaitForFirstConsumer
```

ebs.csi.aws.com 是 AWS 提供的更现代的、基于 CSI 的存储解决方案，推荐在新项目中使用，支持更多的功能和未来发展。kubernetes.io/aws-ebs 是传统的 in-tree 驱动，功能有限且逐步被淘汰，建议尽早迁移到 CSI 驱动。

PersistentVolume（持久卷）

PersistentVolume 是一个集群范围的资源，表示存储资源的实际实现。它由管理员配置，提供了持久的存储卷。持久卷是存储的实际实例，可以是
NFS、iSCSI、云存储等。

persistentvolume.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: host
  finalizers:
    - kubernetes.io/pv-protection
spec:
  capacity:
    storage: 50Gi
  hostPath:
    path: /app/k8s
    type: ''
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: hostpath
  volumeMode: Filesystem
```

aws的ebs磁盘

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: aws-ebs
spec:
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: aws-ebs
  hostPath:
    path: /app/k8s_pv
```

PersistentVolumeClaim（持久卷声明）

PersistentVolumeClaim 是用户用来请求存储资源的对象。它描述了应用所需的存储容量、访问模式等。k8s会根据持久卷声明的要求选择适当的持久卷，或者根据 StorageClass 动态创建新的 PersistentVolume。

persistentvolumeclaim.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-example
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: aws-ebs
```

使用这些组件，k8s 可以自动化存储的管理，并根据需求提供适当的存储资源。

```shell
# 查看存储类
kubectl get sc
# 删除存储类
kubectl delete sc aws-ebs
# 查看持久卷      
kubectl get pv
kubectl delete pv aws-ebs
# 查看持久卷声明
kubectl get pvc
kubectl delete pvc aws-ebs  
```

### 通过helm安装kubesphere4.x（推荐）

kubesphere是k8s的开源平台，提供了完整的云原生应用管理、监控、日志、配置、服务发现、网关、身份认证、访问控制、策略管理、工作负载编排、Helm应用管理、多集群管理、多租户管理、多环境管理、多语言支持等功能。

[https://helm.sh/zh/docs/intro/install/](https://helm.sh/zh/docs/intro/install/)

```shell
# 如果没有helm，安装
brew install helm

# 如果无法访问 charts.kubesphere.io, 可将 charts.kubesphere.io 替换为 charts.kubesphere.com.cn
helm upgrade --install -n kubesphere-system --create-namespace ks-core https://charts.kubesphere.io/main/ks-core-1.1.4.tgz --debug --wait
# 如果访问 Docker Hub 受限，请在命令后添加如下配置，修改默认的镜像拉取地址。
--set global.imageRegistry=swr.cn-southwest-2.myhuaweicloud.com/ks
--set extension.imageRegistry=swr.cn-southwest-2.myhuaweicloud.com/ks

# 卸载，危险！
helm -n kubesphere-system uninstall ks-core
```

KubeSphere默认端口与密码

http://ip:30880  
admin/P@88w0rd

### 通过helm安装kubesphere3.x

```shell
# 如果没有helm，安装
brew install helm

# 添加 KubeSphere 的 Helm 仓库
helm repo add kubesphere https://charts.kubesphere.io/main
helm repo update

# 安装 KubeSphere
helm install kubesphere kubesphere/ks-installer --namespace kubesphere-system --create-namespace

# 自定义安装 KubeSphere，修改 values.yaml 文件中的配置
helm show values kubesphere/ks-installer > values.yaml
helm install kubesphere kubesphere/ks-installer --namespace kubesphere-system --create-namespace -f values.yaml
```

### 安装镜像仓库

```shell
#创建用户名和密码
sudo yum install httpd-tools
mkdir -p /root/auth
htpasswd -Bc /root/auth/htpasswd default

#将 htpasswd 文件创建为 Kubernetes Secret
kubectl create secret generic registry-auth-secret --from-file=/root/auth/htpasswd -n registry

#将配置文件创建为 Kubernetes ConfigMap
kubectl create configmap registry-config --from-file=config.yml -n registry

#为了更好地管理资源，创建专用 Namespace（可选）
kubectl create namespace registry

#Deployment 会启动一个包含 Docker Registry 的容器，并将容器的端口 5000 暴露出来
#Service 将 Deployment 中的容器端口 5000 暴露为一个 ClusterIP 服务，使集群内部的其他容器可以访问这个 Docker Registry。
kubectl apply -f registry-deployment.yaml
kubectl apply -f registry-service.yaml

# 为了将镜像推送到 Kubernetes 内部的 Docker Registry，需要将 Docker 客户端配置为信任这个私有仓库。
# 可以通过 Kubernetes 集群节点的 IP 地址和 NodePort（例如 32000）访问 Docker Registry。
# 集群内部的其他容器可以直接通过 http://registry.registry:5000 访问 Docker Registry。
kubectl apply -f registry-nodeport.yaml
```

config.yml

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  secret: registry-auth-secret
  headers:
    X-Content-Type-Options: [ nosniff ]
  auth:
    htpasswd:
      realm: basic-realm
      path: /auth/htpasswd
```

registry-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: registry
  namespace: registry
spec:
  replicas: 1
  selector:
    matchLabels:
      app: registry
  template:
    metadata:
      labels:
        app: registry
    spec:
      containers:
        - name: registry
          image: registry:2
          ports:
            - containerPort: 5000
          volumeMounts:
            - name: registry-storage
              mountPath: /var/lib/registry
            - name: registry-auth
              mountPath: /root/auth
            - name: registry-config
              mountPath: /etc/docker/registry
      volumes:
        - name: registry-storage
          emptyDir: { }
        - name: registry-auth
          secret:
            secretName: registry-auth-secret
        - name: registry-config
          configMap:
            name: registry-config
```

registry-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: registry
  namespace: registry
spec:
  selector:
    app: registry
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
```

registry-nodeport.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: registry-nodeport
  namespace: registry
spec:
  type: NodePort
  selector:
    app: registry
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 32000
```
