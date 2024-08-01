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

kubectl apply -f default-storageclass.yaml
kubectl patch storageclass gp3 -p '{"metadata": {"annotations": {"storageclass.kubernetes.io/is-default-class": "true"}}}'
```

default-storageclass.yaml

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
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
    X-Content-Type-Options: [nosniff]
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
        emptyDir: {}
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
