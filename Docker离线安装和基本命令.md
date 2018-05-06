### 写在前面
本来这一篇打算直接写`Dockerfile`，但是觉得安装部署实在是初学者的一大难题，自己也是踩过各种坑啊。简单的知识点积累起来其实就很厉害了，这一篇主要涉及三个方面：

1. `Docker` 的离线安装；
2. `Docker` 的主要组成部分;
3. `image` 和 `container` 的基础命令

如果后面有机会的话还会涉及`Dockfile`，`docker compose`, `docker machine`和`k8s`等，挑战下自己吧。

### Docker的离线安装
上一篇主要是使用 `yum` 源在线安装 `Docker`，比较简单，一直认为`Linux`下的`yum`源安装方式比win下要方便的多。但是有的时候去实际环境部署的时候宿主机器往往是不能联网的，要能快速方便的离线部署。

这里使用的也是比较常见通用的方式：在线环境下下载对应的所需所有`rpm`包，构建`repo`，然后拷贝到离线环境，建立离线环境下的本地`yum`源。这种方式有一个极大的好处就是在实际部署的时候和在线是一样的，重点是构建的本地`repo`必须要包含所有依赖的`rpm`包。

#### 准备环境
可以使用虚拟机来创建一个和离线部署机器相同系统版本的环境，比如这里使用的是`VirtualBox`创建一个基于`CentOS-7-x86_64-DVD-1708.iso`版本的一个镜像。最好使用最小化安装的方式，因为这样镜像里面的依赖项比较少。设置好网络连接，最好使用桥接网卡的形式。

```
# 配置epel yum源
$ yum -y install epel-release yum-utils createrepo

# 配置docker yum源
$ yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 创建保存rpm包的目录
$ mkdir /home/docker-repo

# 下载docker-ce依赖的`rpm`包
# 这一步非常重要，如果系统中已经安装了`docker-ce`就无法下载所有的依赖或者下载的包不全，默认忽略掉已经安装的包，目前好像没有办法能支持下载已经安装的依赖。
$ yum install --downloadonly --downloaddir=/home/docker-repo docker-ce

# 查看下载的rpm包
$ ls /home/docker-repo/*.rpm
/home/docker-repo/audit-libs-python-2.7.6-3.el7.x86_64.rpm
/home/docker-repo/checkpolicy-2.5-4.el7.x86_64.rpm
/home/docker-repo/container-selinux-2.42-1.gitad8f0f7.el7.noarch.rpm
/home/docker-repo/docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm
/home/docker-repo/libcgroup-0.41-13.el7.x86_64.rpm
/home/docker-repo/libsemanage-python-2.5-8.el7.x86_64.rpm
/home/docker-repo/libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm
/home/docker-repo/pigz-2.3.4-1.el7.x86_64.rpm
/home/docker-repo/policycoreutils-python-2.5-17.1.el7.x86_64.rpm
/home/docker-repo/python-IPy-0.75-6.el7.noarch.rpm
/home/docker-repo/setools-libs-3.3.8-1.1.el7.x86_64.rpm
```
#### 构建repo
```
$ createrepo -v /home/docker-repo/

# 可以看到多了一个repodata目录
$ ls /home/docker-repo/
audit-libs-python-2.7.6-3.el7.x86_64.rpm            
libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm
checkpolicy-2.5-4.el7.x86_64.rpm                    
pigz-2.3.4-1.el7.x86_64.rpm
container-selinux-2.42-1.gitad8f0f7.el7.noarch.rpm 
policycoreutils-python-2.5-17.1.el7.x86_64.rpm
docker-ce-18.03.1.ce-1.el7.centos.x86_64.rpm        
python-IPy-0.75-6.el7.noarch.rpm
libcgroup-0.41-13.el7.x86_64.rpm                    
repodata
libsemanage-python-2.5-8.el7.x86_64.rpm             
setools-libs-3.3.8-1.1.el7.x86_64.rpm

# 查看下一般repo目录的基本结构
ls /home/docker-repo/repodata/
01dd2b461e1a2b4103e0c200f351c6346d473c755ddbcfcde96d3a97442a2d7e-other.xml.gz
434df4cf5480ce435357b14b63d88d64f3cea85c15f6d4f0b1c8130c38a3beb3-other.sqlite.bz2
6b6f5d2d3232aa54a66b19da0b3711afb8a6dcee564f9f71a815909d97564e34-filelists.sqlite.bz2
8cf9a98dfcfe4b99a7e804c66413edfd2f8b0bd5c25fbf159c7e28a4d3e2abfc-primary.xml.gz
96dc3b9b3b1644c4ed4b2e25019815cfc58a4b75204ccd6cac90f1237d0701f4-filelists.xml.gz
b86f30c74baa378fbc23114110b5cdf631c49be7a1816eec9e4acf6da4c83eac-primary.sqlite.bz2
repomd.xml
```
#### 离线安装部署
创建好`repo`之后基本上就完成了一半了，现在只需要在离线环境下配置本地yum源就可以了
```
# 拷贝创建好的repo（即整个docker-repo目录）到需要部署的离线宿主机器

# 配置本地yum源
$ mkdir /etc/yum.repos.d/bak
$ mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak
$ vi /etc/yum.repos.d/local_docker_ce.repo

# 写入以下内容
[local_docker-ce]
name=local docker-ce
baseurl=file:///home/docker-repo
enable=1
gpgcheck=0

$ yum clean all
$ yum makecache
$ yum search docker

已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
============================================= N/S matched: docker ==============================================
docker-ce.x86_64 : The open-source application container engine

# 安装docker-ce
$ yum install docker-ce
$ docker -v
Docker version 18.03.1-ce, build 9ee9f40
```
整个流程思路还是很清晰的，也比较简单。这样配置好的repo可以多次使用，而且不限于`docker`的安装，很多依赖项多的服务都可以使用这样方式。前期准备好，到了离线环境下就非常顺利了。

`docker-ce-18.03.1` 的repo压缩包已经上传到网盘，需要的点击[阅读原文](https://pan.baidu.com/s/1MOSTKMESPDzDaEfJxEsgww)获取。

### Docker的主要组成部分
`Docker`主要可以分为`Docker daemon`和`Docker client`:
* Docker daemon : Docker守护进程，或者叫Docker服务。用户通过`client`与该守护进程交互。
* Docker client : Docker客户端，与Docker daemon交互通信返回结果给用户。Docker client同时提供`socket`或者`RESTful api`的远程访问形式。

`image`中包含了应用需要运行的环境，只读的。某一个正在运行的`image`的实例称为一个`container`。`image`可以自己使用`Dockerfile`创建,也可以从`Docker hub`上下载已经制作好的`image`。

`container`是一个隔离的环境，多个`container`之间不会相互影响。

`Docker hub`是官方的`image`托管网站，地址为：`https://hub.docker.com/explore/`，比较出名的`image`都可以从中找到,类似`github`代码托管。

### image基本命令
```
# 运行一个Hello World，如果本地不存在该`image`，默认会去拉取改`image`然后执行

$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
9bb5a5d4561a: Pull complete 
Digest: sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/

# 查看本地所有images : docker images 或者 docker image ls
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              e38bc07ac18e        3 weeks ago         1.85kB

# 删除docker image : docker rmi [image id] 或者 docker image rm [image id]
# 必须先移除运行过该image的container
$ docker rmi e38
Error response from daemon: conflict: unable to delete e38bc07ac18e (must be forced) - image is being used by stopped container d07947ffb63a

$ docker rm d07
d07
[root@localhost ~]# docker rmi e38
Untagged: hello-world:latest
Untagged: hello-world@sha256:f5233545e43561214ca4891fd1157e1c3c563316ed8e237750d59bde73361e77
Deleted: sha256:e38bc07ac18ee64e6d59cf2eafcdddf9cec2364dfe129fe0af75f1b0194e0c96
Deleted: sha256:2b8cbd0846c5aeaa7265323e7cf085779eaf244ccbdd982c4931aef9be0d2faf
```
### container基本命令
```
# 查看当前所有container ：docker ps -a 或者 docker container ls -a
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
8c3b78b50987        hello-world         "/hello"            2 minutes ago       Exited (0) 2 minutes ago                       eager_golick

# 查看所有正在运行的container : docker ps 或者 docker container ls

# 查看所有正在运行的container的ID : docker ps -q

# 停止所有正在运行的container : docker stop $(docker ps -q)

# 停止并移除所有（正在运行的和已经停止的）container : docker rm -f $(docker ps -qa)

# 重新启动container : docker restart [container id]

# 清理掉所有处于终止状态的容器 ： docker container prune
```
