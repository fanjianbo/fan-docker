# Docker学习笔记

## 一、引言
### 1.1 简介
* 开源的**应用容器引擎**，基于Go语言。
* Docker让开发者**打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。**
* 完全的**沙箱机制**，互相之间不会有任何接口（类似iPhone的app），更重要的是容器**性能开销极低**

### 1.2 应用场景
* web应用的自动化打包和发布
* 自动化测试和持续集成、发布
* 在服务型环境中部署和调整数据库或其他的后台应用
* 从头编译或者扩展现有的OpenShift或Cloud Foundry平台来搭建现有PaaS<sup>①</sup>(平台即服务)环境

注①:与之类似的还有SaaS，软件即服务，Software as a Service。详情见：C:\Users\fjb19\Documents\从头开始--记录每一份知识\谁能举个通俗易懂的例子告诉我IAAS，SAAS，PAAS的区别？ - 知乎

### 1.3 Docker的优点
* **简化程序**：利用虚拟化实现方便快捷的部署，真的很方便快捷！真的很方便快捷！真的很方便快捷！
* **避免选择恐惧症**：参考Docker镜像
* **节省开支**：Docker与云结合，让云空间得到更充分的利用。


## 二、Docker架构
Docker使用C/S架构模式，使用远程API来管理和创建Docker容器
Docker容器通过Docker镜像来创建
**容器与镜像的关系类似于对象和类的关系**

|Docker|面向对象|
|--|--|
|容器|对象|
|镜像|类|

详细架构参见下表后面的示意图

|Docker组成|详细|
|---|---|
|Docker镜像(Images)|Docker镜像用于创建Docker容器的模板|
|Docker容器(Container)|容器是独立运行的一个或一组应用|
|Docker客户端(Client)|Docker客户端通过命令或者其他工具使用Docker API与Docker的守护进程通信|
|Docker主机(Host)|一个物理或虚拟(VMware、阿里云ECS等)的机器，用于执行Docker守护进程和容器|
|Docker仓库(Registry)|Docker仓库用来保存镜像，可以理解为代码控制中的代码仓库。<br>Docker Hub([https://hub.docker.com](https://hub.docker.com))上提供了庞大的镜像集合供使用|
|Docker Machine|Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure|

## 三、CentOS上安装Docker
### 3.1 前提条件
* 目前，CentOS仅发行版中的内核支持Docker
* Docker运行在CentOS7上时，要求系统为64位，系统内核版本为3.10以上
* Docker运行在CentOS6.5或更高版本上时，要求系统为64位，且系统内核版本为2.6.32-431 或者更高版本。

### 3.2 使用yum安装(这里使用阿里云的CentOS7)
使用```uname -r```检查你当前的内核版本，我的是```3.10.0-327.el7.x86_64```

#### 3.2.1 安装Docker
* ```yum -y install docker```
* 安装完成后，使用```service docker start```启动Docker后台服务
* **自己运行hello world时，第一次报了一个错(没注意是什么错误)，什么也没动下，第二次执行就不报错了....mark一下**
* 运行结果如下：

由于本地没有hello-world这个镜像，所以会下载一个hello-world的镜像，并在容器内运行

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://cloud.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

## 四、Docker Hello World
### 4.1 HelloWorld
Docker允许你在容器内部运行应用程序，使用```docker run```命令来在容器内运行一个应用程序。如下命令：
```
docker run ubuntu:15.10 /bin/echo "hello world"
```
这个命令执行比较长，我本人是给Ctrl+C了。因为它的作用是让docker以Ubuntu15.10镜像创建一个新容器，然后在容器里执行/bin/echo "Hello World"。(后记：使用docker镜像加速器就不会这么慢了，因为大部分时间在下载镜像)

上面命令指定了运行的镜像Ubuntu15.10后，**首先从本地主机上查找惊险是否存在，如果不存在，Docker就会从镜像仓库Docker hub上下载公共镜像**

### 4.1.1 实际操作的小插曲
最开始按照菜鸟教程的方法，用

```
yum -y install docker
```
安装docker。但因为下载镜像的速度实在是太慢啦。所以开始寻找类似maven mirror的方式加速下载。
因为使用的是阿里云的服务器，所以发现下面这个地址[安装／升级你的Docker客户端](https://cr.console.aliyun.com/?spm=5176.100239.blogcont29941.13.F3XUNG#/accelerator)。
本人手一直都很贱，因为很顺利的发现了解决方案，所以也没仔细阅读，就脑抽随手```yum -y remove docker```并执行了上面```curl xxx```命令。这下好了....报错了。**错误1如下：**

```
Warning: the "docker" command appears to already exist on this system.

If you already have Docker installed, this script can cause trouble, which is
why we're displaying this warning and provide the opportunity to cancel the
installation.

If you installed the current Docker package using this script and are using it
again to update Docker, you can safely ignore this message.

You may press Ctrl+C now to abort this script.
+ sleep 20
+ sh -c 'sleep 3; yum -y -q update; if [ -z "" ] ; 
+ then yum -y -q install docker-engine; else yum -y -q install docker-engine-; fi'
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
^C

Exiting on user cancel
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
```

看错误的第一行，这个错误是因为之前安装的docker失败或者删除时没删干净导致的。

**解决1办法**：
* 看看哪些包没有删干净
```
yum list installed | grep docker
```
* 将列出的包分别用全名删掉

接着继续执行curl命令下载加速器(**这一步又错啦，下面这条命令千外别随便执行啦!!!....叫你不好好看文档！！！**)

```
curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
```

又报错。**错误2如下**：

```
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
newaliases: fatal: parameter inet_interfaces: no local interface found for ::1
warning: /etc/profile created as /etc/profile.rpmnew
warning: /etc/shadow created as /etc/shadow.rpmnew
warning: /etc/nsswitch.conf created as /etc/nsswitch.conf.rpmnew
warning: /etc/security/limits.conf created as /etc/security/limits.conf.rpmnew
...
...
```
**错误2解决方案**：
不要执行那个命令，至于“为什么”，等有空详细研究Linux的各种常用工具(如curl)再说吧....

**接下来，正确的加速姿势**：
* 重新使用yum安装docker，且保证docker版本高于1.10，否则下面的命令无效
* 通过下面的一系列命令，修改/etc/docker/daemon.json，并重启deamon,其中[https://t566lnce.mirror.aliyuncs.com](https://t566lnce.mirror.aliyuncs.com)是我的阿里云服务器分配的加速镜像地址，不知道非阿里云用户能不能用呢？闲来无事时试一试

```
1、sudo tee /etc/docker/daemon.json <<-'EOF'
2、
{
  "registry-mirrors": ["https://t566lnce.mirror.aliyuncs.com"]
}
3、EOF
4、sudo systemctl daemon-reload
5、sudo systemctl restart docker
```


好啦，接下来试试那个最开始下载很慢的hello-world：
```
docker run ubuntu:15.10 /bin/echo "hello world"
```

看到这飞一样的下载速度是不是很开森呢？！！


### 4.2 运行交互式的容器
通过docker的两个参数 ```-i -t```,让docker运行的容器实现**“对话”**的能力

```
docker run -i -t ubuntu:15.10 /bin/bash
```
执行上面命令后，成功得到下面的结果：
```
root@63b496737b8e:/# 
```

第一次看到这结果，我只能惊讶的说了一句**“我操！”**。嗯，就是**屌**！

**参数解析**

|参数|含义|
|--|--|
|-t|在新容器内指定一个伪终端或终端|
|-i|允许你对容器内的标准输入(STDIN)进行交互|


### 4.3 启动容器(后台模式)

使用以下命令创建一个以进程方式运行的容器

```
docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

输出结果：
```
2b1b7a428627c51ab8810d541d759f072b4fc75487eed05812646b8534a2fe63
```
输出结果不是"hello world"，而是一个长的字符串，它是**容器的ID**，对每个容器都是唯一的。

**查看有哪些容器在运行**：
```
docker ps
```
运行结果：
```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
0f43b51fafe9        ubuntu:15.10        "/bin/sh -c 'while tr"   4 seconds ago       Up 3 seconds                            high_heyrovsky
```

部分字段解析：

|字段|含义|
|--|--|
|CONTAINER ID|容器ID|
|NAMES|自动分配的容器名称|

在容器内使用下面命令显示容器内的标准输出：
```
docker logs  ${CONTAINER ID}
这里是：docker logs 0f43b51fafe9
```

运行结果与脚本期望输出一致：
```
hello world
hello world
hello world
hello world
hello world
.....
```


### 4.4 停止容器

在容器外的本机上执行下面命令：
```
docker stop ${CONTAINER ID}/${NAMES}
```


## 五、Docker容器使用

### 5.0 后记补充
还有一些非常实用的命令《菜鸟教程》没有列出，这里简单列举如下

```java
docker restart id/name

//使用docker stop只是停止容器，相当于关机，想要删除则需要使用docker rm xxx
docker rm id/name  可以接多个id/name，批量删除

//已经stop的容器，docker ps命令不会列出，需要使用-a让他们现形
docker ps -a
```

### 5.1 Docker客户端
* 使用
```
docker
```
查看可用命令

* 使用
```
docker command --help
```
查看命令使用方法/帮助

### 5.2 运行一个Web应用
下面尝试使用docker构建一个web应用程序

我们将在docker容器中运行一个Python Flask应用来运行一个web应用。
```
docker run -d -P training/webapp python app.py
```
**参数含义**：

|参数|含义|
|--|--|
|-d|后台运行|
|-P|将容器内部使用的网络端口映射到我们使用的主机上|

**运行结果：**
```
Unable to find image 'training/webapp:latest' locally
Trying to pull repository docker.io/training/webapp ... 
latest: Pulling from docker.io/training/webapp

e190868d63f8: Pull complete 
909cd34c6fd7: Pull complete 
0b9bfabab7c1: Pull complete 
a3ed95caeb02: Pull complete 
10bbbc0fc0ff: Pull complete 
fca59b508e9f: Pull complete 
e7ae2541b15b: Pull complete 
9dd97ef58ce9: Pull complete 
a4c1b0cb7af7: Pull complete 
Digest: sha256:06e9c1983bd6d5db5fba376ccd63bfa529e8d02f23d5079b8f74a616308fb11d
2f8a2eeb0e2b072a93f36397f2fd4a259445cac7b1a33c7ad4d362ed89745306
```

```docker ps```一下，可以看到一个新的docker容器启动了：
```
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
2f8a2eeb0e2b        training/webapp     "python app.py"     3 minutes ago       Up 3 minutes        0.0.0.0:32768->5000/tcp   sleepy_newton
```
**上面的结果中，重点看这里：**PORTS=0.0.0.0:32768->5000/tcp

**含义：**Docker开放了5000端口(默认Python Flask端口)映射到主机端口的32768上。

这是我们可以通过浏览器访问——[主机:32768](#)——的方式访问WEB应用

**访问结果：**

![](https://raw.githubusercontent.com/fanjianbo/picture/3ca5547944d19fe4c325fe5f324e8f7b6e3f9d70/res/QQ%E6%88%AA%E5%9B%BE20170329122517.png)

**使用```-p```标识来绑定指定端口**

使用-P的映射端口是随机分配的，我们也可以使用```-p```来指定分配的端口

```
docker run -d -p 5000:5000 training/webapp python app.py
```



### 5.3 网络端口的快捷方式
除了使用```docker ps```查看容器的映射端口外，还可以通过如下命令，查看给定容器id的映射端口
```
docker port id
```

### 5.4 登陆连接docker容器
在第四章中，我们使用```docker run xxx```运行一个容器后，会自动连接到该容器中。但假设该容器是一deamon方式运行的，退出到主机用户上后，我们还可以使用多种方式连接到docker容器，这只列出推荐的做法：
```
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```
**注意到，至少包含两个参数[   container和command   ]**
例如：
```
docker exec -it ${id} /bin/bash
```

### 5.5 查看web应用程序的日志
```
docker logs -f ID或者名字
```
使用**-f**参数可以像使用tail -f命令一样输出日志

### 5.6 查看容器内部运行的进程
```
docker top CONTAINER名字/id [ps OPTIONS]
```
同理还有相对应的
```
docker kill name/id
```

### 5.7 检查web应用程序
```
docker inspect [OPTIONS] CONTAINER|IMAGE|TASK
```
上述命令将以**JSON**的格式输出container的信息


## 六、Docker镜像使用
当容器运行时，使用的镜像如果本地不存在，则会自动从docker镜像仓库下载，默认是从docker hub下载，自己的阿里云服务器已经改成从阿里的docker镜像mirror地址下载。

接下来看看**管理和使用本地Docker主机镜像**和**创建镜像**

### 6.1 列出镜像列表
```
docker images
```

### 6.2 使用镜像
就像前面说的那样
```
docker run [options] image [command]
```

* 一般option都会使用-i参数，使得容器内可以使用标准输入
* 一般command都会使用 /bin/bash，使得启动容器后进入容器内(启动容器后你也许想进去干点什么？) 
* **image**参数使用```
```
docker images
```
列出的镜像名称，如果不指定版本的话，会使用最新lastest版本。使用“：”指定版本，如ubuntu:15.10

### 6.3 获取新镜像
```
docker pull image
```
同样的，image如果不指定版本，将下载最新的latest版本

### 6.4 查找镜像
```
docker search imagename
```

### 6.5 创建、更新镜像
当我们从docker镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改
* 从已经创建的容器中更新镜像，并且提交这个镜像
* 使用Dockerfile指令来创建一个新的镜像


## 七、Docker安装jenkins
不废话了，**直接上干货**，樊小贱，你要是看不懂就去吃屎啦！！！
```
docker pull jenkins
mkdir -p /jenkins_home && chmod -R 777 /jenkins_home
docker run -it -d --restart always --name jenkins -p 8080:8080 -p 50000:50000 -v /jenkins_home:/var/jenkins_home jenkins
```

好啦，直接浏览器访问ip:8080就可以啦，然后docker log看下日志吧



## 八、Docker安MySQL
以下命令中my.cnf自行创建
```
docker pull mysql:5.6
mkdir -p /usr/local/mysql_docker && chmod -R 777 /usr/local/mysql_docker
cd /usr/local/mysql_docker
mkdir conf && chmod -R 777 conf
mkdir data && chmod -R 777 data
mkdir logs && chmod -R 777 logs
docker run -p 3306:3306 --name mymysql -v $PWD/conf/my.cnf:/etc/mysql/my.cnf -v $PWD/data:/mysql_data -e MYSQL_ROOT_PASSWORD=123qwe -d mysql:5.6
```

## 九、Docker安装Apache

```
docker pull httpd
mkdir -p /usr/local/httpd && chmod -R 777 /usr/local/httpd
mkdir -p /usr/local/httpd/conf && chmod -R 777 /usr/local/httpd/conf
mkdir -p /usr/local/httpd/logs && chmod -R 777 /usr/local/httpd/logs
mkdir -p /usr/local/httpd/www && chmod -R 777 /usr/local/httpd/www
cp /etc/httpd/conf/httpd.conf /usr/local/httpd/conf
//学习时，提前使用yum安装了httpd，并把生成的httpd.conf文件挂在到了docker的apache下
cd /usr/local/httpd
docker run -p 80:80 -v $PWD/www/:/usr/local/apache2/htdocs -v $PWD/conf/httpd.conf:/usr/local/apache2/conf/httpd.conf -v $PWD/logs/:/usr/local/apache2/logs/ -d httpd
```






