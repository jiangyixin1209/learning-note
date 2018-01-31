# Git介绍

Git是一个开源的分布式版本控制系统，可以有效、高速的处理从很小到非常大的项目版本管理。Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。

# Git客户端安装

``` shell
yum install –y git
```

# Git常用命令

## 初始化git项目

``` git
git init
```

## 查看当前项目状态

``` git
git status
```

## 新建文件并再次查看状态

``` git
echo “# My Project” > README.md
git status
```

## 记录当前操作，记录新加入的文件并再次查看状态

``` git
git add README.md
git status
```

## 记录当前更改并加以信息描述

``` git
git commit 文件名 -m’add my first project’
```

## 查看提交历史

``` git
git log
```

## 新建远程仓库

``` git
git remote add origin https://guthub.com/注册用户名/trygit.git
```

## 同步到远程仓库

``` git
git push -u origin master
```

## 从远程代码库同步到本地

``` git
git pull origin master
```

## 与同步前对比变更

``` git
git diff HEAD
```

## 查看当前更改变更

``` git
git diff --staged
```

## 恢复到为更改状态

``` git
git reset README.md
```

## 覆盖本地文件

``` git
git checkout octocat.txt
```

## 新建分支

``` git
git branch feature1
```

## 切换分支

``` git
git checkout feature1
```

## 删除本地分支

``` git
git branch –d feature1
```

# Git hook

Git也具有在特定事件发生之前或之后执行特定脚本代码功能（从概念上类比，就与监听事件、触发器之类的东西类似）。Git Hooks就是那些在Git执行特定事件（如commit、push、receive等）后触发运行的脚本。  
按照Git Hooks脚本所在的位置可以分为两类：  
本地Hooks，触发事件如commit、merge等。  
服务端Hooks，触发事件如receive等。