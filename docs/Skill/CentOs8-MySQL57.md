# Centos8安装Mysql5.7详细教程

# 一、安装准备

## （1）下载Mysql

如果没有wget先安装wget

> yum install wget

下载mysql安装包

> wget [dev.mysql.com/get/mysql80…](https://link.juejin.cn?target=http%3A%2F%2Fdev.mysql.com%2Fget%2Fmysql80-community-release-el7-3.noarch.rpm)

## （2）安装MySQL的安装工具

> rpm -ivh mysql80-community-release-el7-3.noarch.rpm

## （3）默认安装 的是mysql8.0，修改配置文件

> cd /etc/yum.repos.d/ vim mysql-community.repo

将mysql80的`enabled=1`改为0，`mysql57=0`的enable改为1，如图

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f823018cbf54ab09ed4833f10a278d8~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

保存退出。

# 二、安装Mysql

## （1）安装

先执行

> yum module disable mysql

再执行

> yum install mysql-community-server --nogpgcheck

## （2）查看版本

> mysql --version

## （3）启动

> systemctl start mysqld.service

## （4）查看是否启动成功

如图表示启动成功 ![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/458c6be654cb4d2c91b159c2078306f4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

# 三、修改密码

## （1）查看临时密码

> grep 'temporary password' /var/log/mysqld.log

红框内的就是临时密码

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7601cc7a6ab1405daeb0e6929dd5c0d4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## （2）用临时密码登录Mysql

> mysql -uroot -p

把查看到的临时密码输入即可登录。

## （3）修改密码

> set password=password("你的新密码");

设置成功时如下图： ![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9bc4004c1474b59af1699198ecf81b4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

# 四、开启远程连接

## （1）首先关闭防火墙

查看防火墙状态

> systemctl status firewalld

如图表示开启防火墙

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae03fc0289854e7ead241f9ee1171a82~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

关闭防火墙

> service firewalld stop

如图则是关闭状态 ![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f29016e82844964875ac9c1137782ec~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

## （2）登录到Mysql

## （3）授权通过密码远程连接

> GRANT ALL ON *.* TO root@'%' IDENTIFIED BY '你的登录密码' WITH GRANT OPTION;

## （4）刷新权限

> flush privileges;

# 五、设置开机自启动

> systemctl enable mysqld

> systemctl daemon-reload

查看是否设置成功，显示`enabled`表示设置成功

> systemctl is-enabled mysqld



