# 利用Dockerfile创建镜像

## 什么是Dockerfile

定义：Dockerfile是一个文本格式的配置文件，用户可以使用Dockerfile快速创建自定义镜像。

## Dockerfile的基本结构
Dockerfile由一行行的命令语句组成。并且支持以#开头的注释行。一般Dockerfile分为四个部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时的指令。

Dockerfile示例:

``` dockerfile
#This is a Dockerfile
#Author:chen
#第一行必须指定基础镜像
FROM ubuntu
#维护者信息
MAINTAINER <15221613915@163.com>
#镜像的操作指令
RUN apt-get update && apt-get install –y nginx
RUN echo “\ndaemon off” >> /etc/nginx/nginx.conf
#容器启动时的指令
CMD /usr/sbin/nginx
```

创建命令

``` shell
docker build -t="jiangyx/first_build" .
```

执行此命令要和Dockerfile在同级目录，文件名称必须为Dockerfile。命令后面的”.”表示在当前目录下执行。

# Dockerfile指令集

## FROM

``` text
格式为 FROM <image>或<image>:<tag>。
第一条指令必须为FROM指令，用于指定基础镜像。
```
## MAINTAINER

```text
格式为 MAINTAINER <name>,指定维护者信息。
```

## RUN

```text
格式为 RUN <command>,会在shell终端运行命令。
```

## EXPOSE

``` text
格式为 EXPOSE <port> [<port> ...],容器需要暴露的端口号。镜像启动可以通过 –P 或 -p 进行端口映射的绑定。
```

## ENV

``` text
格式为 ENV <key> <value>。
指定一个环境变量，可以被后续的RUN引用，并且在容器中记录该环境变量。
```

## ADD

``` text
格式为 ADD <src> <dest>。
该命令将复制指定的<src>到容器中的<dest>。其中<src>可以是Dockerfile所在目录的一个相对路径；也可以是url，还可以是tar文件（自动解压）。
```

## VOLUME

``` text
格式为 VOLUME [path]。
创建一个可以从本地主机或其他容器挂载点，一般用来存放需要保持的数据。
```

## USER

``` text
格式为 USER <username>。
指定运行容器时的用户名，后续的RUN也会指定该用户。
```

## WORKDIR

``` text
格式为 WORKDIR <work_path>。
指定工作空间，运行完WORKDIR后，后续执行的RUN、CMD、ENTRYPOINT都会在此目录下执行。
```

## COPY

``` text
格式为 COPY <src> <dest>。
复制本地主机的<src>到容器中的<dest>,目标路径不存在时，会自动创建。
当使用本地目录为源目录时，推荐使用COPY。
```

## CMD

``` text
推荐格式为 CMD [“executable”,”param1”,”param2”] 。
作为ENTRYPOINT的默认参数为 CMD［”param1”,”param2”］。
指定容器的启动命令，每个Dockerfile只能有一条CMD命令，如果指定多条，只有最后一条会执行。
用户启动容器时指定运行命令，会覆盖掉Dockerfile中的CMD命令。
```

## ENTRYPOINT

``` text
格式为 ENTRYPOINT [“executable”,”param1”,”param2”]。
配置容器启动后的命令，可被docker run提供的--entrypoint参数覆盖。
每个Dockerfile只能有一条ENTRYPOINT命令，如果指定多条，只有最后一条会执行。
```

``` docker
# 这是一个Dockerfile文件
# Author: jiangyx

# 第一行必须指定基础镜像
FROM ubuntu
# 维护者信息
MAINTAINER <15221613915@163.com>
# 奖项操作质量
RUN apt-get update && apt-get install -y nginx
RUN echo "\ndaemon off" >> /etc/nginx/nginx.conf

# 暴露端口
# 只有暴露的端口后才能使用端口映射
EXPOSE 80 443

# 设置环境变量
ENV NGINX_BIN_PATH /use/sbin

# 拷贝文件
ADD hello.txt /hello.txt

# 挂在目录
VOLUME /root

# 设置工作空间
WORKDIR /root

# 容器启动时的指令
CMD /usr/sbin/nginx
```

# Dockerfile最佳时间

## 错误定位

每个Dockerfile的指令可以生成新的一层镜像，如果通过Dockerfile创建镜像出错，可以根据出错所在步骤的上一层启动容器，然后手工执行出错层的命令，以达到调试目的。

## 好的使用习惯

[http://dockone.io/article/131](http://dockone.io/article/131)  
[http://dockone.io/article/132](http://dockone.io/article/132)