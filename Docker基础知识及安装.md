### 写在前面 — What is Docker
> Docker is the world’s leading software containerization platform.

`Docker` 是引领世界的开源容器引擎，开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何可以运行`docker`容器的机器上，同时可实现虚拟化。通俗一点就是类似集装箱，软件/服务及其环境依赖就是货物，`docker`就是能装货物的集装箱。

### 基本概念
`Docker`为开发和运维人员提供了一个使用容器进行开发，部署和运行应用的平台。使用linux容器来部署应用称为容器化（`containerization`）。容器并不是一个新概念，但是用它们来部署应用非常简单。容器化带来的好处：
* Flexible: 即使再复杂的应用都可以容器化。 
* Lightweight: 轻量级且共享主机内核。
* Interchangeable: 实时部署更新你的应用。
* Portable: 一次编译，到处运行。
* Scalable: 轻松增加或创建容器副本。
* Stackable: 实时管理服务队列。
#### Images and containers

一个`image`就是运行一个应用的所有依赖集合，包括代码，运行时，基础库，环境变量和配置文件等。
`image`是一些Docker层（`layer`）的集合。
一个正在运行的`image`的实例称为一个`container`。
当我们运行一个`image`的时候，一个对应于这个`image`的`container`就产生了。
同一个`image`可能对应许多正在运行的`container`。

#### Containers and virtual machines
容器不是虚拟机！容器不是虚拟机！ 容器不是虚拟机！
容器本质上是系统下的一个独立进程，容器之间公用系统内核，因此很轻量级。
虚拟机是通过Hypervisor层抽象底层基础设施资源，提供相互隔离的虚拟环境。

### 更新/安装 Docker
查看当前系统docker信息
```
$ sudo docker --version
Docker version 17.12.0-ce, build c97c6d6

$ sudo docker version
Client:
 Version:	17.12.0-ce
 API version:	1.35
 Go version:	go1.9.2
 Git commit:	c97c6d6
 Built:	Wed Dec 27 20:10:14 2017
 OS/Arch:	linux/amd64

Server:
 Engine:
  Version:	17.12.0-ce
  API version:	1.35 (minimum version 1.12)
  Go version:	go1.9.2
  Git commit:	c97c6d6
  Built:	Wed Dec 27 20:12:46 2017
  OS/Arch:	linux/amd64
  Experimental:	false

$ sudo docker info
Containers: 5
 Running: 0
 Paused: 0
 Stopped: 5
Images: 10
Server Version: 17.12.0-ce
Storage Driver: overlay2
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
 ...
```

卸载系统docker[可选]
```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine

# 卸载Docker后,`/var/lib/docker/`目录下会保留原Docker的镜像,网络,存储卷等文件. 如果需要全新安装Docker,需要删除`/var/lib/docker/`目录

$ sudo rm -rf /var/lib/docker/
```

安装docker
```
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

# 配置阿里的docker repo
$ sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 配置epel repo
$ sudo yum -y install epel-release

# 安装/更新
$ sudo yum -y install docker-ce

# 查看docker信息
$ sudo docker -v
Docker version 18.03.1-ce, build 9ee9f40
```
### Docker服务启动
```
# 启动服务
$ sudo systemctl start docker
$ sudo systemctl status docker

# 开机启动
$ sudo systemctl enable docker
$ sudo systemctl disable docker

```
### 配置官方加速器
1. 对于使用 `systemd` 的系统，应该通过编辑服务配置文件 `docker.service` 来进行加速器的配置

找到`docker.service`所在目录：
$ rpm -qa | grep docker
docker-ce-18.03.1.ce-1.el7.centos.x86_64
$ rpm -ql docker-ce-18.03.1.ce-1.el7.centos.x86_64 | grep docker.service
/usr/lib/systemd/system/docker.service

修改`docker.service`中`ExecStart`如下：
ExecStart=/usr/bin/dockerd --registry-mirror=https://registry.docker-cn.com

2. 通过daemon.json文件配置
```
$ vim /etc/docker/daemon.json
{
 "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

### 非root用户使用docker
```
$ sudo groupadd docker

$ sudo usermod -aG docker $USER

# 退出用户重新登录

$ docker run hello-world
```
### 参考资料
[Docker官网 Get Started ](https://docs.docker.com/get-started/)

[Get Docker CE for CentOS](https://docs.docker.com/install/linux/docker-ce/centos/#upgrade-docker-ce)

[几张图帮你理解 docker 基本原理及快速入门](https://www.cnblogs.com/SzeCheng/p/6822905.html)
