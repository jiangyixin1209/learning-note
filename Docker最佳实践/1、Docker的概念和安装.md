# Docker的概念

## 镜像

Docker 镜像就是一个只读的模板。  

例如：一个镜像可以包含一个完整的 ubuntu 操作系统环境，里面仅安装了 Apache 或用户需要的其它应用程序。  
镜像可以用来创建 Docker 容器。

创建Docker镜像有几种方式，多数是在一个现有镜像基础上创建新镜像，因为几乎你需要的任何东西都有了公共镜像，包括所有主流Linux发行版，你应该不会找不到你需要的镜像。不过，就算你想从头构建一个镜像，也有好几种方法。  

要创建一个镜像，你可以拿一个镜像，对它进行修改来创建它的子镜像 。

## 容器

Docker 利用容器来运行应用。  

容器是从镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。

可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

> 注：镜像是只读的，容器在启动的时候创建一层可写层作为最上层。

## 仓库

仓库是集中存放镜像文件的场所。

有时候会把仓库和仓库注册服务器（Registry）混为一谈，并不严格区分。实际上，仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。

***

# Docke的安装

> 要求：Linux内核版本最低尾3.10  (uname -r 查看当前内核版本)

## 通过yum方式安装docker

更新yum源

``` shell
sudo yum update
```

增加docker的yum源

``` shell
sudo vim /etc/yum.repos.d/docker.repo
```

``` shell
[dockerrepo]
name=Docker Repository baseurl=https://yum/dockerproject/org/repo/main/centos/$releasever/
enabled=1
gpcheck=1
gpgkey=https://yum.dockerproject.org/gpg
```

通过yum安装docker

``` shell
sudo yum install docker-engine
```

启动docker服务

``` shell
sudo service docker start
```

验证docker是否安装成功

``` shell
sudo docker version
```

测试docker镜像

``` shell
docker run hello-world
```

***

# Docker的卸载

查看安装包

``` shell
yum list installed | grep docker
```

移除安装包

``` shell
sudo yum -y remove docker-engine.x86_64
```

清除所有docker依赖的文件

``` shell
rm -rf /var/lib/docker
```

***

# Docker的配置

新建docker组

``` shell
sudo groupadd docker
```

将当前用户添加到docker组

``` shell
sudo usermod -aG docker <用户名>
```

退出当前用户再次登录

``` shell
exit
service docker restart
```

验证docker是否安装成功

``` shell
docker run hello-wrold
```

设置docker开机启动

``` shell
sudo chkconfig docker on
```