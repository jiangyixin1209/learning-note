> Docker支持全平台安装，Windows、Mac以及Linux。Docker在Windows和Mac上的安装都比较便捷，因此不再叙述，现在主要来介绍在Linux环境中安装Docker。我选择的Linux版本为Centos7。

# 安装Centos

Centos系统的安装有很多种方法，例如直接安装、虚拟机安装等等。我选用的是使用vagrant来安装centos。

## 安装Vagrant

Vagrant的安装比较方便，访问[Vagrant](https://www.vagrantup.com/downloads.html) 下载自己所在平台的安装包进行安装即可。除了Vagrant之外，还需要安装VirtualBox。

## 安装Centos7

在安装完成Vagrant之后，我们就可以来安装Centos7了，安装的方式也比较方便，只需要以下两行命令即可启动一个Centos系统。

```shell
# 在一个新建的目录下输入
vagrant init centos/7
vagrant up
```

启动后输入以下命令即可连接虚拟机。

```shell
vagrant ssh
```

## 安装Docker

1、删除之前安装的Docker

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

2、安装必要的依赖

```shell
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

3、添加repo

```shell
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

4、安装Docker

```shell
sudo yum install docker-ce
```

5、启动Docker

```shell
 sudo systemctl start docker
```

6、验证Docker的启动

```shell
sudo docker run hello-world
```

7、配置加速器

```shell
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://58ab2c05.m.daocloud.io
```

8、添加当前用户到Docker用户组

```python
sudo groupadd docker
sudo gpasswd -a 用户名 docker
```

