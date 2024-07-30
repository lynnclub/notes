---
title: "kubernetes"
type: "docs"
weight: 3
---

### 安装依赖

VERSION="v1.30.1"
wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin

### 安装

https://mirrors.aliyun.com/kubernetes-new/core/stable/v1.29/rpm/aarch64/

kubectl
kubelet
kubeadm

初始化控制节点
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

移除控制节点标签，使其同时作为工作节点
kubectl taint nodes --all node-role.kubernetes.io/control-plane-

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

安装 Pod 网络插件
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

加入工作节点
kubeadm join <control-plane-endpoint> --token <token> --discovery-token-ca-cert-hash <hash>

设置默认存储
kubectl apply -f default-storageclass.yaml.yaml
kubectl patch storageclass gp3 -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'

kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.4.0/kubesphere-installer.yaml
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.4.0/cluster-configuration.yaml

# 删除 KubeSphere 安装
kubectl delete -f https://github.com/kubesphere/ks-installer/releases/download/v3.4.0/kubesphere-installer.yaml
kubectl delete -f https://github.com/kubesphere/ks-installer/releases/download/v3.4.0/cluster-configuration.yaml

http://ip:30880
admin/P@88w0rd