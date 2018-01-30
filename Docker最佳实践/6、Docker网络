# 容器对外服务

当容器内运行一些网络应用，要让外部访问这些应用时，可以通过 -P 或 -p 参数来指定端口映射。
使用 -P 映射时，Docker会随机映射一个30000 ～ 49900 的端口至容器内部开放的端口：

``` shell
docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=root mysql:5.6
```

通过docker ps可以看到端口映射关系。可以通过映射在宿主机的端口来访问对应容器内的服务。

``` shell
# 映射到指定宿主机的端口
docker run -d -p 3306:3306 --name mysql mysql:5.6
# 映射到指定地址的指定端口
docker run -d -p 127.0.0.1:3306:3306 --name mysql mysql:5.6
# 映射到指定地址的任意端口
docker run -d -p 127.0.0.1::3306 --name mysql mysql:5.6
# 查看映射端口配置
docker port mysql 3306
```

# 容器相互通信

通过映射宿主机的端口实现容器的互联。

容器的连接(link)除了端口映射外的另一种可以与容器中应用进行交互的方式。

使用 --link 参数可以让容器之间安全的进行交互。

``` shell
# 创建一个数据库容器：
docker run -d --name mysqldb mysql:5.6
# 创建一个web容器并和数据库容器建立连接：
docker run -d --name Webapp –p 8000:8080 --link mysqldb:MySQL tomcat
# mysqldb容器和web容器建立互联关系。
# --link参数的格式为--link name:alias,其中name是要连接的容器名称，alias是这个连接的别名。
```

可以使用docker ps（PORT字段）来查看容器的连接。
Docker在两个容器之间创建了安全隧道，而且不用映射它们的端口到宿主机上。在启动mysqldb的时候并没有使用-p和-P标记，从而避免的了暴露数据库的端口到外部的网络上。
Docker通过两种方式为容器公开连接信息：
1 环境变量：
使用env命令来查看。
EX:
docker run --rm --name test --link dblink:dblink ubuntu env
2 更新/etc/hosts文件
查看/etc/hosts文件。