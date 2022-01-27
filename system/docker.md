# Docker

## 1. 安装Docker

### 1.1 设置yum仓库

~~~shell
# 安装yum-utils
sudo yum install -y yum-utils

# 添加仓库
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
~~~

### 1.2 用yum安装Docker Engine

#### 1.2.1 安装最新版Docker Engine

~~~shell
sudo yum install docker-ce docker-ce-cli containerd.io
~~~

#### 1.2.2 安装指定版本Docker Engine

~~~shell
# 列出所有可用版本
yum list docker-ce --showduplicates | sort -r

# 安装指定版本
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
~~~

### 1.3 启动Docker

~~~shell
systemctl start docker
~~~

### 1.4 验证安装

~~~shell
sudo docker run hello-world
~~~

## 2. 离线安装docker

### 2.1 准备一台联网&没安装docker的CentOS机器

~~~shell
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
~~~

### 2.2 使用yumdowmloader下载所有rpm包

~~~shell
mkdir ~/docker-ce && cd ~/docker-ce

yumdownloader --resolve docker-ce docker-ce-cli containerd.io

cd .. && zip -r docker-ce.zip ~/docker-ce
~~~

### 2.3 离线机器安装

~~~shell
unzip docker-ce.zip && cd docker-ce

rpm -ivh --replacefiles --replacepkgs *.rpm
~~~

### 2.4 设置docker

~~~shell
systemctl enable docker && systemctl start docker
~~~

## 2. Docker命令

## 3. Dockerfile命令