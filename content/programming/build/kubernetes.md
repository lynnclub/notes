---
title: "kubernetes"
type: "docs"
weight: 3
---

### 安装依赖

VERSION="v1.30.1"
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-arm64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin

### 安装k8s命令

https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.29/rpm/aarch64/

kubectl
kubelet
kubeadm

arm平台最新的1.30依赖有问题装不上，只能安装1.29

### 安装k8s

```shell
#初始化控制节点

sudo kubeadm init --pod-network-cidr=10.244.0.0/16

#移除控制节点标签，使其同时作为工作节点（分布式集群不推荐）

kubectl taint nodes --all node-role.kubernetes.io/control-plane-

#配置文件

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

#安装网络插件

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

#加入工作节点（分布式集群）

kubeadm join <control-plane-endpoint> --token <token> --discovery-token-ca-cert-hash <hash>

#设置默认存储

kubectl apply -f default-storageclass.yaml.yaml
kubectl patch storageclass gp3 -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'
```


### 安装kubesphere

```shell
#安装KubeSphere

kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.4.0/kubesphere-installer.yaml
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.4.0/cluster-configuration.yaml

#删除KubeSphere

kubectl delete -f https://github.com/kubesphere/ks-installer/releases/download/v3.4.0/kubesphere-installer.yaml
kubectl delete -f https://github.com/kubesphere/ks-installer/releases/download/v3.4.0/cluster-configuration.yaml
```

默认端口密码

http://ip:30880  
admin/P@88w0rd
