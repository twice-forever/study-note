# Docker安装

## CentOS在线安装

### 删除旧版本

~~~shell
sudo yum remove docker \
                docker-client \
                docker-client-latest \
                docker-common \
                docker-latest \
                docker-latest-logrotate \
                docker-logrotate \
                docker-engine
~~~

### 设置官方仓库

~~~shell
sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
sudo yum makecache fast
~~~

### 安装docker

~~~shell
sudo yum install docker-ce docker-ce-cli containerd.io
~~~

### 开启docker

~~~shell
sudo systemctl enable docker && systemctl start docker
~~~

### 验证安装

~~~shell
sudo docker run hello-world
~~~

## CentOS离线安装

### 下载离线rpm包

~~~
// 在全新安装的CentOS系统中运行
mkdir docker-ce && cd ./docker-ce && sudo yumdownloader docker-ce docker-ce-cli containerd.io && zip -qr docker-ce.zip *
~~~

### 在离线CentOS中安装docker

~~~
unzip docker-ce.zip

rpm -Uvh --force *.rpm
~~~

### 开启docker

~~~shell
sudo systemctl enable docker && systemctl start docker
~~~

