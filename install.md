```
$ docker --version
Docker version 17.09.1-ce, build 19e2cf6
```

$ sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine

卸载Docker后,/var/lib/docker/目录下会保留原Docker的镜像,网络,存储卷等文件. 如果需要全新安装Docker,需要删除/var/lib/docker/目录
rm -fr /var/lib/docker/

$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

  $ sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
sudo yum -y install docker-ce

$ docker -v
Docker version 17.12.0-ce, build c97c6d6


### 加速器
http://www.jb51.net/article/116873.htm

http://www.jb51.net/article/112921.htm

$ vim /etc/docker/daemon.json
 
{
 "registry-mirrors": ["https://registry.docker-cn.com"]
}

### Manage Docker as a non-root user

$ sudo groupadd docker

$ sudo usermod -aG docker $USER

$ docker run hello-world

### Configure Docker to start on boot
$ sudo systemctl enable docker

$ sudo systemctl disable docker

## Define a container with Dockerfile

