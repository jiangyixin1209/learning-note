# 安装Gitlab前系统预配置准备工作

## 关闭firewalled防火墙

```shell
# 关闭防火墙服务
systemctl stop firewalld
# 禁用防火墙服务开机启动
systemctl disable firewalld
```

## 关闭SELINUX并重启系统

```shell
vim /etc/sysconfig/selinux
```

```text
...
SELINUX=disabled
...
```

```shell
reboot
```

***

# 安装 Omnibus Gitlab-ce package

## 安装Gitlab组件

```shell
yum -y install curl policycoreutils openssh-server openssh-clients postfix
```

## 配置Yum仓库

```shell
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | sudo bash
```

## 启动postfix邮件服务

```shell
systemctl start postfix && systemctl enable postfix
```

## 安装Gitlab-ce社区版本

```shell
yum install -y gitlab-ce
```

***

# Omnibus Gitlab等相关配置初始化

