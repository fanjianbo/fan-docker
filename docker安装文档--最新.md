# fan-docker

docker安装经历了两个很不爽的阶段。  
* 第一次是在公司的老旧服务器上安装，记得当时鼓捣了大概一整天吧，怎么也无法编译成功。最后才发现，原来是linux内核版本太低导致的...**kernel version的最低要求为3.10**，而公司老旧的服务器版本才2.6.x.....嗯....别笑!!!

* 发现问题后，因为各种原因没有升级服务器的内核，于是跑到自己的阿里云ECS上搞。最初还写了一个版本笔记，中间也因为更换docker镜像地址等问题卡主过，但最后总算是成功了，而且顺利的用了几个月。***那为什么还是会不爽呢？*** 因为某个偶然的机会，我发现在**gitbooks**上，早就有了成熟安装教程....而且针对当初卡住我的 *镜像加速* 也有针对阿里云的非常方便的解决方案.


* **so,这里记录更好的解决方案，旧版笔记移植```fan-history```**
* 参考地址：[《Docker——从入门到实践》](https://yeasy.gitbooks.io/docker_practice/content/install/centos.html)
* 旧版笔记链接:[docker学习笔记]()


## 一、CentOS 安装 Docker CE
PS：CE是免费版，EE是收费版

### 1.1 准备工作
#### 1.1.1 系统要求
**Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10**。 CentOS 7 满足最低内核的要求，但由于内核版本比较低，部分功能（如 overlay2 存储层驱动）无法使用，并且部分功能可能不太稳定。

#### 1.1.2 卸载旧版本
旧版本的 Docker 称为 ```docker``` 或者 ```docker-engine```，使用以下命令卸载旧版本：
```
$ sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine
```

### 1.2 使用yum源 安装
执行以下命令安装依赖包：
```
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

#### 1.2.1 国内源
执行下面的命令添加 yum 软件源：
```
$ sudo yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

> 以上命令会添加稳定版本的 Docker CE yum 源。从 Docker 17.06 开始，edge test 版本的 yum 源也会包含稳定版本的 Docker CE。

#### 1.2.2 官方源

```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

如果需要最新版本的 Docker CE 请使用以下命令：
```
$ sudo yum-config-manager --enable docker-ce-edge
```
```
$ sudo yum-config-manager --enable docker-ce-test
```

#### 1.2.3 安装 Docker CE
更新 ```yum``` 软件源缓存，并安装 ```docker-ce```。
```
$ sudo yum makecache fast
$ sudo yum install docker-ce
```

### 1.3 使用脚本自动安装
在测试或开发环境中 Docker 官方为了简化安装流程，提供了一套便捷的安装脚本，CentOS 系统上可以使用这套脚本安装：

```
$ curl -fsSL get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
```

执行这个命令后，脚本就会自动的将一切准备工作做好，并且把 Docker CE 的 edge 版本安装在系统中。

### 1.4 启动 Docker CE
```
$ sudo systemctl enable docker
$ sudo systemctl start docker
```

### 1.5 建立 docker 用户组
默认情况下，```docker``` 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 ```root``` 用户和 ```docker``` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root 用户。因此，更好地做法是将需要使用 ```docker``` 的用户加入 ```docker``` 用户组。

建立 docker 组：
```
$ sudo groupadd docker
```
将当前用户加入 docker 组：
```
$ sudo usermod -aG docker $USER
```

### 1.6 镜像加速
鉴于国内网络问题，后续拉取 Docker 镜像十分缓慢，强烈建议安装 Docker 之后配置 [国内镜像加速](https://yeasy.gitbooks.io/docker_practice/content/install/mirror.html)。

### 1.7 添加内核参数
默认配置下，如果在 CentOS 使用 Docker CE 看到下面的这些警告信息：

```
WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```
请添加内核配置参数以启用这些功能。
```
$ sudo tee -a /etc/sysctl.conf <<-EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
然后重新加载 sysctl.conf 即可
```
$ sudo sysctl -p
```