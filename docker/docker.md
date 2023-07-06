# docker是一个快速交付、运行应用的技术


## docker解决依赖兼容问题

```
1. 将应用的libs（函数库）、deps（依赖）、配置与应用一起打包，形成可移植现象。

2. 将每个应用放到一个隔离容器中去运行，避免相互干扰，使用沙箱机制，相互隔离。
```

ubuntu和centos都是基于linux内核，只是系统应用不同，提供的函数库有差异。

## docker如何解决不同系统环境的问题

```
1. docker将用户程序与所需要调用的系统函数库一起打包。
2. docker运行到不同操作系统时，直接基于打包的函数库，借助于操作系统的linux来运行。
//docker镜像中包含完整的与性能环境，包括系统函数库，可以认为docker打包好的程序包可以运行在任何一个linux内核的操作系统中。
```
启动和移除都可以通过一行命令完成，方便快捷。


# docker与虚拟机

```
1.虚拟机是在操作系统中模拟硬件设备，然后运行另一个操作系统.
2.docker不需要模拟出整个操作系统，只需要模拟出一个小规模的环境。
```
### 性能区别

```
1. docker是一个系统进程，虚拟机是在操作系统中的操作系统；
2.  docker体积小、启动速度快，性能好，接近原生，磁盘占用一般为MB，启动也是秒级；
3  虚拟机体积小，启动速度快、性能好，虚拟机体积大速度慢，性能一般；
1.  虚拟机性能较差，磁盘占用一般为分钟级，相当于把操作系统重启。
```

# 镜像和容器

### 镜像
```
docker将应用程序及其所需的依赖、函数库、环境、配置等文件打包在一起，称为镜像。
//本质上，docker镜像是一个linux的文件系统，包含可以运行在Linux内核的程序及相应的数据。
```
#### 镜像特征
```
1. 分层：一个镜像可以由多个中间层组成，多个镜像可以共享同一中间层；镜像和中间层是多对多的关系。
2. 只读：镜像在构建完成后不可修改。
```

### 容器
```
镜像中的应用程序运行后形成的进程就是容器。docker会给容器做隔离，对外不可见。
```
### 镜像和容器的关系
```
容器是通过镜像来创建的，所以必须现有镜像才能创建容器。生成的容器是一个独立于宿主机的隔离进程，并且有属于容器自己的网络和命名空间。镜像构建完成后不可修改，容器是可以动态改变的。
```

# Docker和DockerHub
```
DockerHub是一个Docker镜像托管平台，统称为Docker Registry。
```

# Docker架构
```
Docker是一个CS架构的程序，由服务端和客户端组成。
服务端（Server）：Docker守护进程，负责处理Docker指令，管理镜像、容器等。
客户管（Client）：通过命令或RestAPI向Docker服务端发送指令，可以在本地或远程向服务端发送指令。
```
### 构建镜像

1. 在本地通过命令形式:
   
Client上通过 **docker build** 命令构建一个镜像，这个命令到达DockerServer之后，被 **docker daemon  守护进程** 接收和处理，利用提供的数据，构建一个镜像。

2. 从Registry拉去镜像：
Client上通过 **docker pull** 命令，DockerServer的守护进程会去从Registry中拉去指定的镜像

### 运行镜像创建容器
Clinet上通过 **docker run** 告诉Server要创建进程。

# Docker 安装（centos）
>1.docker分为ce（社区版）和ee（企业版）

> docker ce 支持64位版本的centos7，并且要求内核版本不低于3.10

### 1.1 卸载docker
```
yum remove docker \
        docker-client \
        docker-client-latest \
        docker-common \
        docker-latest \
        docker-latest-logrotate \
        docker-logrotate \
        docker-engine

删除镜像、容器、配置文件等内容：
rm -rf /var/lib/docker
```

### 1.2 安装docker
首先虚拟机需要联网，安装yum工具
```
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2 --skip-broken
```
然后更新本地镜像源

阿里云:
```
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
清华大学源:
```
yum-config-manager \
    --add-repo \
    https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
```

```
yum makecache fast
```
>yum makecache fast命令是将软件包信息提前在本地索引缓存，用来提高搜索安装软件的速度,建议执行这个命令可以提升yum安装的速度。

开始下载
```
yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
#### CentOS8安装docker并解决docker和podman冲突问题
1. 更新yum
>yum -y update
2. centos8默认使用podman代替docker，所以需要containerd.io
>yum install https://download.docker.com/linux/fedora/30/x86_64/stable/Packages/containerd.io-1.2.6-3.3.fc30.x86_64.rpm
3. 安装其他依赖
>yum install -y yum-utils device-mapper-persistent-data lvm2

> yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
4. 安装docker 
>yum install docker-ce docker-ce-cli containerd.io
5. 若报错，
因为centos8默认使用podman代替docker，直接安装docker会产生冲突，因此：
> yum erase podman buildah

> yum install docker-ce docker-ce-cli containerd.io

### 1.3 启动docker
docker应用需要用到各种端口，需逐一修改防火墙设置；
若后续使用中出现docker无法访问的情况，可对防火墙进行检查。

- 关闭防火墙：
>systemctl stop firewalld

- 禁止开机启动防火墙：
>systemctl disable firewalld
  
- 启动docker
>systemctl start docker

- 停止docker
>systemctl stop docker

- 重启docker
>systemctl restart docker

- 查看docker版本
>docker -v

### 1.4 配置国内镜像加速
参考阿里云的镜像加速

配置镜像加速器
```
#mkdir -p /etc/docker
#tee /etc/docker/daemon.json<<-'EOF'
{
   "registry-mirrors":["https://n0dwemtq.mirror.aliyuncs.com]
}
EOF
#systemctl daemin-reload
#systemctl restart docker
```

# Docker基本操作
### 镜像
- 镜像名称[repository]:[tag]
- 未指定tag时，默认为latest，代表最新版本的镜像

##### 构建镜像
1. 本地构建
   local使用Dockerfile文件，利用 **docker build**  命令构建镜像
2. 镜像服务器拉取
    可以使用 **docker pull**  从Docker Registry中拉取镜像
##### 常用命令
> docker images 查看镜像

> docker rmi 删除镜像 （缩写remove images）

> docker push 推送镜像到服务

> docker save 保存镜像为压缩文件 

> docker load 加载压缩包为镜像

> docker --help 查看帮助文档

>  docker images --help 查看命令详情

### 从DockerHub中拉去一个nginx镜像并查看
1. 去镜像仓库搜索nginx镜像[hub.docker.com]挂梯子
2. 拉去nginx
> docker pull nginx
3. 查看镜像 
> docker images

### 利用docker save将nginx镜像导出磁盘，再通过load加载回来
1. 利用 docker xx --help 命令查看 docker save 和 docker load的语法
2. 使用 docker tag 创建新镜像mynginx1.0  
3. 使用 docker save 导出镜像到磁盘
 > docker save --help

 > docker save -o nginx.tar nginx:latest # 导出到压缩文件

 > ll #查看

 4. 删除镜像
 > docker rmi nginx:latest
 5. 加载
 > docker load --help
 
 > docker load -i nginx.tar

### 容器命令
- docker run #让容器处于运行状态
- docker pause #暂停运行
- docker unpause #恢复运行
- docker stop #停止
- docker start #开始
- docker ps #查看容器运行状态
- docker logs #查看容器日志
- docker exec #进入容器执行命令
- docker rm #删除容器

### 创建运行一个nginx容器
1. 去 docker hub上查看nginx运行命令
> docker run --name containeName -p 80:80 -d nginx
//docker run #创建并运行一个容器
//--name #给容器起一个名字
//-p #将宿主机端口与容器端口映射，左侧是宿主机端口，右侧是容器端口
//-d #后台运行容器
//nginx #镜像名称
生成一个唯一id（长字符型）
> docker run --name mn -p 80:80 -d nginx
2. 查看容器（默认查看运行中的容器）
> docker ps
3. 访问端口
在浏览器访问虚拟机ip：端口
4. 查看docker日志
> docker logs mn
5. 停止docker
> docker stop -t=60 mn
//docker stop 容器ID或容器名
参数 -t：关闭容器的限时，如果超时未能关闭则用kill强制关闭，默认值10s，这个时间用于容器的自己保存状态
docker stop -t=60 容器ID或容器名

6. 删除指定容器
> docker rm -f mn
### 访问失败：
1. 检查虚拟机防火墙并关闭
> systemcyl stop firewalld
1. 查看虚拟机ip
> ifconfig
ens:192.168.44.128     docker0:172.17.0.1
1. Docker容器内部端口映射到外部宿主机端口
> ROUTE -p add 172.17.0.0 mask 255.255.0.0 192.168.44.128
1. 访问docker端口
   [http://172.17.0.2/80]

### 持续监控日志
> docker logs -f mn

#### 启动容器报错
Error response from daemon: driver failed programming external connectivity on endpoint mn 
1. 重启docker服务 
> systemctl restart docker   
2. 查看所有容器（-all）
> docker ps -a 
1. 强制删除
> docker rm -f mn
1. 重新运行镜像
> docker run --name mn -p 80:80 -d nginx

//docker -rm mn不能删除运行中的容器,而强制删除-f可以
### 进入nginx容器，修改html文件内容，添加helloworld
1. 进入容器
> docker exec -it mn bash
// docker exec #进入容器内部执行命令
// -it #创建输入输出标准，允许与容器交互
// mn #指定容器名称
// bash #进入容器后执行的命令，bash是一个Linux终端交互命令
root@c21d79e73790:/# 
在根目录/中
2. 进入静态文件目录中(nginx的html所在目录)
> cd /usr/share/nginx/html
3. 修改index.html内容(替换内容)
>sed -i 's#Welcome to nginx#hello world#g' index.html
>sed -i 's#<head>#</head><meta charset="utf-8">#g' index.html
//vi index.html会出现vi:command not found
镜像封装时，只是应用程序所需要的必备函数库和一些命令


docker exec -it mr redis-cli
等价于
docker exec -it mr bash
redis-cli

### 数据卷
容器与数据耦合的问题
1. 不便于修改
   - 当要修改nginx的html内容时要进入容器内部才能修改。
2. 数据不可复用
   - 在容器内的修改对外是不可见的，所有修改对新创建的容器是不可复用的。
3. 升级维护困难
   - 数据在容器内，如果要升级容器则必然删除旧容器，而数据也随之被删除。

数据卷是一个虚拟目录，指向宿主机文件系统中的某个目录。
此时在宿主机上修改文件会立即反映到挂载到数据卷的容器里，就不用进入容器内部再进行修改了，解决了不便于修改的问题；
新创建的容器也可以挂载到同一数据卷上，对数据卷的一切修改，新容器也可以看到，解决了数据共享问题；
对于删除的容器，数据卷不被删除，所有数据还在，新版本的容器可以接着挂载到数据卷上，解决了升级维护困难的问题；

#### 操作数据卷
docker volume create 创建一个数据卷
docker volume inspect 显示一个或多个volume的信息
docker volume ls 列出所有的volume
docker volume prune 删除未使用的volume
docker volume rm 删除一个或多个指定的volume

### 创建一个数据卷，查看数据卷在宿主机的目录位置
1. 创建数据卷
   > docker volume create html
2. 查看所有数据
   > docker volume ls
3. 查看数据卷详细信息卷
   > docker volume inspect html

### 挂载数据卷
-v 数据卷:容器内的目录
>例：docker run --name mn -v html:/usr/share/nginx/html -p 80:80 -d nginx
若挂载的数据卷未创建，数据卷在容器创建时自动创建。
查看数据卷位置
> docker volume inspect html
进入html数据卷所在位置
> cd /var/lib/docker/volumes/html/_data
修改文件
> vi index.html














