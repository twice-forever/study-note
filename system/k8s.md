# k8s

## 1. k8s安装

### 1.1 修改docker daemon.json

~~~shell
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors" : [
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com",
    "https://cr.console.aliyun.com/"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

systemctl daemon-reload && systemctl restart docker
~~~

### 1.2 设置系统环境

~~~shell
# 关闭防火墙
systemctl disable firewalld && systemctl stop firewalld

# 关闭selinux
setenforce 0 && \
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# 禁用交换分区
swapoff -a && sed -i 's/.*swap.*/#&/' /etc/fstab

# 修改内核参数
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
~~~

### 1.3 安装kubeadm

~~~shell
# 配置k8s阿里源
cat > /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

# 下载rpm包
mkdir ~/k8s && cd ~/k8s
yumdownloader --resolve kubelet kubeadm kubectl

# 安装kubelet kubeadm kubectl
sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# 离线安装
rpm -ivh --replacefiles --replacepkgs *.rpm

# 开启kubelet
sudo systemctl enable --now kubelet
~~~

### 1.4 准备docker镜像

~~~shell
# 输出默认配置到yaml文件
kubeadm config print init-defaults > kubeadm-config.yaml

# 列出所需镜像
kubeadm config images list --config kubeadm-config.yaml

# 拉取镜像
docker pull registry.aliyuncs.com/google_containers/kube-apiserver:v1.23.0
docker pull registry.aliyuncs.com/google_containers/kube-controller-manager:v1.23.0
docker pull registry.aliyuncs.com/google_containers/kube-scheduler:v1.23.0
docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.23.0
docker pull registry.aliyuncs.com/google_containers/pause:3.6
docker pull registry.aliyuncs.com/google_containers/etcd:3.5.1-0
docker pull registry.aliyuncs.com/google_containers/coredns:v1.8.6

# 导出镜像
docker save registry.aliyuncs.com/google_containers/kube-apiserver:v1.23.0 > kube-apiserver:v1.23.0.tar
docker save registry.aliyuncs.com/google_containers/kube-controller-manager:v1.23.0 > kube-controller-manager:v1.23.0.tar
docker save registry.aliyuncs.com/google_containers/kube-scheduler:v1.23.0 > kube-scheduler:v1.23.0.tar
docker save registry.aliyuncs.com/google_containers/kube-proxy:v1.23.0 > kube-proxy:v1.23.0.tar
docker save registry.aliyuncs.com/google_containers/pause:3.6 > pause:3.6.tar
docker save registry.aliyuncs.com/google_containers/etcd:3.5.1-0 > etcd:3.5.1-0.tar
docker save registry.aliyuncs.com/google_containers/coredns:v1.8.6 > coredns:v1.8.6.tar
~~~

### 1.5 安装master节点

~~~shell
kubeadm init --image-repository registry.aliyuncs.com/google_containers
~~~

### 1.6 安装**Flannel** Network

~~~shell
# 下载flannel配置文件
wget https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

# 安装
kubectl apply -f kube-flannel.yml

# 导出flannel所需镜像
docker save rancher/mirrored-flannelcni-flannel-cni-plugin:v1.0.1 > mirrored-flannelcni-flannel-cni-plugin:v1.0.1.tar
docker save rancher/mirrored-flannelcni-flannel:v0.16.1 > mirrored-flannelcni-flannel:v0.16.1.tar
~~~

