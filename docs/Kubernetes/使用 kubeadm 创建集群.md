# 使用 kubeadm 创建集群

官网地址：[https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/](https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)


### 准备工作
#### 确保每个节点上的 MAC 地址和 product_uuid 的唯一性
- 你可以使用命令 ip link 或 ifconfig -a 来获取网络接口的 MAC 地址
- 可以使用 sudo cat /sys/class/dmi/id/product_uuid 命令对 product_uuid 校验

#### 允许 iptables 检查桥接流量
确保 br_netfilter 模块被加载。这一操作可以通过运行 lsmod | grep br_netfilter 来完成。若要显式加载该模块，可执行 sudo modprobe br_netfilter。

为了让你的 Linux 节点上的 iptables 能够正确地查看桥接流量，你需要确保在你的 sysctl 配置中将 net.bridge.bridge-nf-call-iptables 设置为 1。例如：
```shell
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

#### 检查端口
确保 6443 端口没有被占用

#### 安装 runtime
docker/container

#### 修改docker daemon.jsaon
- 修改cgroup类型，要和k8s统一
- 修改国内镜像源
- 添加本地仓库地址
```json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": [
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn"
  ],
  "insecure-registries": [],
  "debug": true,
  "experimental": false,
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  }
}
```
修改后重启dokcer

### 安装 kubeadm、 kubelet、 kubectl

```shell
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# # 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# master
sudo yum install -y kubelet kubeadm kubectl
# node 
sudo yum install -y kubelet kubeadm

# master/node
sudo systemctl enable --now kubelet
```

### master上执行 kubeadm init
kubeadm init --pod-network-cidr=192.168.0.0/16 --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers


### 问题1
`kubectl get nodes` 报错

The connection to the server localhost:8080 was refused - did you specify the right host or port?

处理方法：
```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 安装 Calico
50 nodes or less
`curl https://projectcalico.docs.tigera.io/manifests/calico.yaml -O`
`kubectl apply -f calico.yaml`

more than 50 nodes
`curl https://projectcalico.docs.tigera.io/manifests/calico-typha.yaml -o calico.yaml`

修改文件
把 `Deployment` 的 name 字段改成 `calico-typha`
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: calico-typha
  ...
spec:
  ...
  replicas: <number of replicas>
```

安装
`kubectl apply -f calico.yaml`

### 查看安装结果
`kubectl get pods --all-namespaces`
可以看到coreDNS容器正常运行
