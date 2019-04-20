---
title: Wordpress-Docker
date: 2019-04-20 12:10:34
tags:
---
在云平台上使用docker搭建Wordpress网站， 最主要的有两件事情：
1. 安装mysql
2. 安装Wordpress

## Mysql的安装

```shell
# 获取mysql镜像到本地， 不适用最新版是因为认证方式有问题
docker pull mysql：5.7
# 使用上一步获取的镜像 mysql 启动一个容器
#   --name 容器名称
#   --v 数据映射。 将mysql的数据存储在宿主机的 /var/lib/mysql-datadir
#   --e, 设置环境变量，这里主要配置了mysql默认的root用户密码
docker run --name wordpress-mysql -v /var/lib/mysql-datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=mwordpressmysql -d mysql:5.7
```

详细的配置情况可以参照官方文档[docker mysql](https://hub.docker.com/_/mysql)
使用`docker ps`查看container的运行情况，如下图，则一个mysql的container已经可以使用了。
![mysql-sucess](/images/wordpress-docker/docker-mysql-success.png)

## Wordpress的安装

```shell
# 获取最新的Wordpress镜像
docker pull wordpress
# 创建Wordpress容器
# 参数和上面相似的就不解释了
# --link docker 用来连接两个容器的，详细请参见官方文档
docker run --name wordpress -d -p 80:80 --link wordpress-mysql:mysql -v /root/wordpress-html wordpress
```

访问主机IP，看到Wordpress的安装界面即为成功。
![wordpress-sucess](/images/wordpress-docker/wordpressSuccess.png)