## 概念

### LXC
Docker的前生 LXC 是Linux Container的简写。是一种轻量级的虚拟化手段，提供了在单一的主机上支持多个相互隔离的server container同时执行的机制。LXC可以将进程沙盒化，提供了一个拥有自己的进程和网络空间的虚拟环境。

### docker
docker是一个开源的应用容器引擎，docker可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的linux服务器，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口，并且容器开销极其低。

### docker 和 LXC
docker并不是LXC替代品，docker底层使用了LXC来实现，在LXC的基础之上，docker提供了一系列更强大的功能。

### docker host 主机
一个物理的或者虚拟的机器， 用于执行docker守护进程和容器。docker本质就是宿主机的一个进程。

### docker client 客户端
docker client 是docker架构中用户用来和docker server建立通信的客户端，即命令行工具docker cli

### docker daemon 守护进程
docker daemon 是docker架构中一个常驻在后台的系统进程。该守护进程在后台启动一个server。

### docker server 
专门服务于docker client的server，接收处理docker client发送的请求。接受请求后，server通过路由与分发调度，找到相应的handler来执行请求。

### docker images 镜像
docker镜像就是一个只读模板，可以用来创建容器， 像面向对象的类。

### docker container 容器
容器是独立运行的一个或一组应用，是从镜像创建的运行实例，它可以被启动，开始、停止、删除。每个容器都是互相隔离的，保证安全的平台。可以把容器看做是一个简易版的linux环境（包括root用户权限、镜像空间、用户空间和网络空间等）和运行在其中的应用程序。

### docker registry 镜像仓库
docker hub， [docker hub](https://hub.docker.com) 提供了很多镜像源

### 容器和虚拟机
容器在主机上运行，并与其他容器共享主机的内核，它运行的一个独立的进程，不占用其他任何可执行文件的内存，非常轻量。

虚拟机运行的是一个完成的操作系统，通过虚拟机管理程序对主机资源进行虚拟访问，相比之下需要的资源更多。

Docker 不是虚拟机，容器就是进程。

### 镜像和容器
通过镜像启动一个容器，一个镜像是一个可执行的包，其中包括运行应用程序所需要的所有内容。包含代码，运行时间，库、环境变量、和配置文件。容器是镜像的运行实例，当被运行时有镜像状态和用户进程，可以使用docker ps 查看。

实际上你可以简单的把image理解为可执行程序，container就是运行起来的进程。

### dockerfile
dockerfile是一个用来构建镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

docker build可以根据dockerfile生成image。之后就可以运行这个image了，这就是docker run命令，image运行起来后就是docker container。

## 安装和配置registry镜像源
### 1. 根据操作系统安装对应的doker
### 2. 配置国内镜像加速
* 对于 Mac， 在docker desktop中配置
* 对于 Centos 7，在 /etc/docker/daemon.json 中添加 `"registry-mirrors": ["https://reg-mirror.qiniu.com/"]`

之后重新启动服务：
```sh
$ sudo systemctl enable docker # 开机自启
$ sudo systemctl start docker

# 如果不是第一次安装就重启
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```
通过 `docker info`命令来查看registry是否替换成功

## 使用

```sh
# 搜索镜像
docker search nginx

# 拉取镜像
docker pull centos:centos7

# 查看本地镜像
docker images

# 删除镜像
docker image rm image-id

# 运行镜像， 生成容器
docker run -it centos:centos7

--name="" 指定容器名字，后续可以通过名字进行容器管理
-t: 在新容器内指定一个伪终端或终端
-i: 允许你对容器内的标准输入 (STDIN) 进行交互
-p 8080:80, 端口进行映射，将本地 8080 端口映射到容器内部的 80 端口
-P: 随机端口映射
-d, 后台运行容器，并返回容器id
--rm：这个参数是说容器退出后随之将其删除
-v path1:path2, 将主机中的path1目录挂载到容器的path2目录，作为其存储路径

# 查看本地运行的容器
docker ps

# 停止某容器
docker stop e22d4c3accb4

# 启动停止的容器
docker container start

# 删除不需要的容器
docker rm 8652b9f0cb4c

# 查看运行的容器的资源占用
docker stats 

# 查看容器内部的标准输出
docker logs container-id

# 进入容器bash，因为容器都是基于linux内核的
docker exec -it 34d0 bash

# 将容器内文件拷贝到宿主机
docker cp 容器名:容器内路径 宿主机路径

# 导出容器
docker export 容器ID > xxx.tar

# 把镜像保存成 tar 文件
docker save -o xxx.tar image

# 退出当前交互
exit
```

Docker 不是虚拟机，容器中的应用都应该以前台执行，而不是像虚拟机、物理机里面那样，用 systemd 去启动后台服务，容器内没有后台服务的概念。

容器运行时应该尽量保持容器存储层不发生写操作，对于数据库类需要保存动态数据的应用，其数据库文件应该保存于卷(volume)。保证容器存储层的无状态化。

当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括：

* 检查本地是否存在指定的镜像，不存在就从公有仓库下载
* 利用镜像创建并启动一个容器
* 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
* 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
* 从地址池配置一个 ip 地址给容器
* 执行用户指定的应用程序
* 执行完毕后容器被终止

推荐run的时候后台运行， -dit，然后需要进入容器的时候再用exec。 `docker exec -it 69d1 bash`

## 构建镜像
使用Dockerfile来定制镜像。

`docker build -t runoob/ubuntu:v1 .`

使用当前目录的 Dockerfile 创建镜像，标签为 runoob/ubuntu:v1。

```sh
-t: 镜像的名字及标签，通常为 name:tag 或者 name 格式
-f: 指定Dockerfile的路径，默认是当前目录
. 表示当前目录，指定镜像构建上下文
```

## Dockerfile
### FROM
所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 nginx 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 FROM 就是指定 基础镜像，因此一个 Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。

### RUN
RUN 指令是用来执行命令行命令的。由于命令行的强大能力，RUN 指令在定制镜像时是最常用的指令之一。

格式：RUN <命令>，就像直接在命令行中输入的命令一样。

每一个 RUN 都是启动一个容器、执行命令、然后提交存储层文件变更。第一层 RUN cd /app 的执行仅仅是当前进程的工作目录变更，一个内存上的变化而已，其结果不会造成任何文件变更。而到第二层的时候，启动的是一个全新的容器，跟第一层的容器更完全没关系。

### CMD
在启动容器的时候，需要指定所运行的程序及参数。CMD 指令就是用于指定默认的容器主进程的启动命令的。

### COPY
COPY <源路径> <目标路径>

源路径 是 构建上下文目录中的相对路径。

目标路径 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 WORKDIR 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

### ADD
ADD 指令和 COPY 的格式和性质基本一致。但是在 COPY 基础上增加了一些功能。ADD 会自动解压缩。

### ENV
设置环境变量。无论是后面的其它指令，如 RUN，还是运行时的应用，都可以直接使用这里定义的环境变量。

定义 `ENV VERSION=1.0` ， $+变量来使用 `$VERSION`

### ARG
格式：ARG <参数名>[=<默认值>]

ARG 指令是定义参数名称，以及定义其默认值。该默认值可以在构建命令 docker build 中用 --build-arg <参数名>=<值> 来覆盖。

### VOLUME
VOLUME <路径>

为了防止运行时用户忘记将动态文件所保存目录挂载为卷，在 Dockerfile 中，我们可以事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

### EXPOSE
EXPOSE 指令是声明运行时容器提供服务端口，这只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

要将 EXPOSE 和在运行时使用 -p <宿主端口>:<容器端口> 区分开来。-p，是映射宿主端口和容器端口，换句话说，就是将容器的对应端口服务公开给外界访问，而 EXPOSE 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。

### WORKDIR
使用 WORKDIR 指令可以来指定工作目录（或者称为当前目录）。

比如连续两次RUN 命令， 都是基于这个WORKDIR的， 并不跟shell脚本一样。
