# linux 或者 mac 安装mysql 忘记密码解决办法

## 使用设备

电脑: linux 或者 like-linux os
mysql版本: 5.7 +

## 问题描述

> 安装完成后不能够通过命令`mysql -u root ` 进行连接数据库

## 解决办法

一般通过dmg安装的mysql的服务器，安装的位置在`/usr/local/mysql-version-name-.../` 

通过进入该目录然后进入 `cd ./bin`

```bash
# 如果正在运行mysql， 则关闭mysql服务
$ sudo lsof -i:3306
$ sudo kill -9  mysql-pid
# 开启mysql 安全模式
$ sudo ./mysqld_safe --skip-grant-tables

# 进入mysql shell
$ sudo ./mysql -u root

# 修改mysql.user 表中的root 用户密码
$ update mysql.user set authentication_string=PASSWORD('you password') where User='root';

# 这里修改就完成了。

```

## 后语

修改完后，如果使用比如navicat 连接改数据库，则会提示修改密码，重新输入密码即可。因为通过安全模式进入修改的密码还是被标记的已过期，so 重新修改OK。


