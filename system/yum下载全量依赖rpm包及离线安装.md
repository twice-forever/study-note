# yum下载全量依赖rpm包及离线安装

## 1. 原因

由于内外网隔离,所以需要进行离线安装,从方便程度上来说,CentOS下使用rpm包安装会比较快

## 2. 使用

### 2.1 yum镜像使用

参考[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/help/centos/):

该文件夹只提供 CentOS 7 与 8，架构仅为 `x86_64` ，如果需要较早版本的 CentOS，请参考 centos-vault 的帮助，若需要其他架构，请参考 centos-altarch 的帮助。

建议先备份 `/etc/yum.repos.d/` 内的文件（CentOS 7 及之前为 `CentOS-Base.repo`，CentOS 8 为`CentOS-Linux-*.repo`）

然后编辑 `/etc/yum.repos.d/` 中的相应文件，在 `mirrorlist=` 开头行前面加 `#` 注释掉；并将 `baseurl=` 开头行取消注释（如果被注释的话），把该行内的域名（例如`mirror.centos.org`）替换为 `mirrors.tuna.tsinghua.edu.cn`。

以上步骤可以被下方的命令一步完成

```shell
$ sudo sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org|baseurl=https://mirrors.tuna.tsinghua.edu.cn|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-*.repo
```

注意其中的`*`通配符，如果只需要替换一些文件中的源，请自行增删。

注意，如果需要启用其中一些 repo，需要将其中的 `enabled=0` 改为 `enabled=1`。

最后，更新软件包缓存

```shell
$ sudo yum makecache
```

### 2.1 安装yum-util

~~~shell
$ sudo yum install yun-install
~~~

### 2.2 下载全量包(如docker)

~~~shell
$ sudo repotrack docker-ce docker-ce-cli containerd.io
~~~

repotrack命令可以下载依赖以及依赖的依赖

### 2.3 离线安装rpm

~~~shell
$ rpm -Uvh --force --nodeps *.rpm
~~~

命令详解:

~~~
-U 更新包
-v 显示指令执行过程
-h 套件安装时列出标记
--force 强行安装
--nodeps 忽略依赖
~~~

