### 写在前面
使用`Dockerfile`来定义并构建`image`,加载镜像来运行容器，可以说`Dockerfile`定义了后面的玩耍范围和规则。`Dockerfile`定义了容器内部运行的一切环境，很好的进行了资源隔离，包括访问网络和磁盘，因此需要映射网络端口和指定需要拷贝的文件到镜像中。

### Dockerfile
> 官方Dockerfile实例

创建一个空目录，然后新建一个`Dockerfile`文件，写入以下内容
```
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```
requiremtns.txt内容如下：
```
Flask
Redis
```

app.py内容如下：
```
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```
> 其实不懂python也没关系，这段代码其实创建了一个http服务，打印一些信息。

构建app
```
$ ls
Dockerfile		app.py			requirements.txt

# 构建image
$ docker build --no-cache=true -t hello:v1 .
Sending build context to Docker daemon  4.608kB
Step 1/7 : FROM python:2.7-slim
 ---> 46ba956c5967
Step 2/7 : WORKDIR /app
Removing intermediate container ad518bdc4496
 ---> 920491c55526
Step 3/7 : ADD . /app
 ---> af2d62a1bf4c
Step 4/7 : RUN pip install --trusted-host pypi.python.org -r requirements.txt
 ---> Running in bbf77f6bac2b
Collecting Flask (from -r requirements.txt (line 1))
  Downloading https://files.pythonhosted.org/packages/7f/e7/08578774ed4536d3242b14dacb4696386634607af824ea997202cd0edb4b/Flask-1.0.2-py2.py3-none-any.whl (91kB)
Collecting Redis (from -r requirements.txt (line 2))
  Downloading https://files.pythonhosted.org/packages/3b/f6/7a76333cf0b9251ecf49efff635015171843d9b977e4ffcf59f9c4428052/redis-2.10.6-py2.py3-none-any.whl (64kB)
  ......
Successfully built itsdangerous MarkupSafe
Installing collected packages: Werkzeug, click, MarkupSafe, Jinja2, itsdangerous, Flask, Redis
Successfully installed Flask-1.0.2 Jinja2-2.10 MarkupSafe-1.0 Redis-2.10.6 Werkzeug-0.14.1 click-6.7 itsdangerous-0.24
Removing intermediate container bbf77f6bac2b
 ---> 23cb2ad03ed4
Step 5/7 : EXPOSE 80
 ---> Running in 308bcbdb0098
Removing intermediate container 308bcbdb0098
 ---> 36dfd57d84fa
Step 6/7 : ENV NAME World
 ---> Running in ccdf64f09cce
Removing intermediate container ccdf64f09cce
 ---> 86c75d7f9e80
Step 7/7 : CMD ["python","app.py"]
 ---> Running in ebb5dc46646f
Removing intermediate container ebb5dc46646f
 ---> 8c301f504e06
Successfully built 8c301f504e06
Successfully tagged hello:v1

# 查看images
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED                  SIZE
python              2.7-slim            46ba956c5967        Less than a second ago   140MB
hello               latest              5e76397922e5        2 hours ago              151MB
```
语法形式 :  `docker build [OPTIONS] PATH | URL |`

1. 根据`Dockerfile`和镜像构建上下文（`Context`）来构建一个镜像。构建上下文是指`PATH`或`URL`里面的一系列文件。构建过程中可以使用上下文中的任何文件，比如`COPY`和`ADD`命令。
这里的`PATH`指的是上下文路径，比较容易和`Dockerfile`路径混淆。实际上 `Dockerfile` 的文件名并不要求必须为 `Dockerfile`，而且并不要求必须位于上下文目录中，可以用`--file , -f`来指定某个文件作为 `Dockerfile`。
一般大家习惯性的会使用默认的文件名 Dockerfile，以及会将其置于镜像构建上下文目录中。

2. 从上述过程看`Dockerfile`中的每一条语句都相当于创建了一个临时镜像, 执行了所要求的命令，最后提交了这一层并且移除了临时镜像。可以设置不移除临时镜像，请自行参考命令行格式。

3. `.dockerignore`文件可以忽略上下文中的文件打包到镜像里面。
  
运行app
```
$ docker run -d -p 4000:80 hello

$ curl http://localhost:4000
<h3>Hello World!</h3><b>Hostname:</b> b8b9457455b6<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```
> 整个流程比较清晰，重点是`Dockerfile`的语法格式

### Dockerfile 基础语法（部分）

`FROM` 指定基础镜像

`FROM` 必须 是 Dockerfile 中第一条非注释命令。可以从`docker hub`上找到很多正式官方的基础镜像，比如`nginx`，`redis`或者系统镜像 `centos`,`ubuntu`等。
除了这些现有基础镜像外，还存在一个特殊的镜像 `scratch`，它表示一个空白的镜像,直接 `FROM scratch` 会让镜像体积更加小巧。

`COPY` 复制文件

`COPY` 指令将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径>

语法形式：
* COPY <源路径>... <目标路径>
* COPY ["<源路径>",... "<目标路径>"]

1. <源路径>支持通配符；
2. 如果 <目标路径> 不存在会在复制文件前自动创建，不需要手动重建；
3. <目标路径> 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 WORKDIR 指令来指定)

`ADD` 复制文件

`ADD` 指令和 `COPY` 的格式和性质基本一致。但是`ADD`增加了一些功能，比如文件自动解压缩。所有的文件复制均使用 `COPY` 指令，仅在需要自动解压缩的场合使用 `ADD`

`RUN` 执行命令

`RUN` 指令是用来执行命令行命令的,有两种形式：
* shell 格式：RUN <命令>
* exec 格式：RUN ["可执行文件", "参数1", "参数2"]

写 `Dockerfile` 其实就是在定义每一层该如何构建,每个 `RUN`会创建一层`layer`,结果就是产生很多层的镜像，极大增加了构建部署的时间,可以使用下面这中形式：
```
RUN xxx \
    && yyy \
    && zzz
```
`CMD` 指令

用于指定默认的容器主进程的启动命令。
* shell 格式：CMD <命令>
* exec 格式：CMD ["可执行文件", "参数1", "参数2"...]

对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。比如启动`nginx`进程，正确的做法是直接执行 nginx 可执行文件，并且要求以前台形式运行:
```
CMD ["nginx", "-g", "daemon off;"]
```
`ENV` 设置环境变量

指令可以支持环境变量展开： ADD、COPY、ENV、EXPOSE、LABEL、USER、WORKDIR、VOLUME、STOPSIGNAL、ONBUILD

示例：
```
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
```

`WORKDIR` 指定工作目录

该目录不存在，后台会自动建立目录

`EXPOSE` 声明端口

`EXPOSE` 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。在运行时使用 `-p <宿主端口>:<容器端口>` 映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 `EXPOSE` 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。


### 参考资料
1. [Docker Get Started,Part 2](https://docs.docker.com/get-started/part2/)
2. [构建上下文](https://yeasy.gitbooks.io/docker_practice/content/image/build.html)