---
id: quick-deployment
sidebar_position: 3.1
---

# 快速部署

为方便用户快速上手，安装包是已经编译好的，只需要检查自己的环境是否缺少依赖。

## 下载安装包
点击[此处](../download.md)下载最新的安装包。

## 上传tar包并解压
```
tar -zxvf stonedb-ce-5.6-v1.0.0.el7.x86_64.tar.gz
```
上传至安装目录，解压出来的文件夹名是stonedb。
## 检查依赖文件
```
cd stonedb/install/bin
ldd mysqld
ldd mysql
```
如果检查返回有关键字"not found"，说明缺少文件，需要安装对应的依赖包。

我们可以参考如下链接寻找所需依赖包：
https://pkgs.org/search/?q=libzstd.so.1 (这里以libzstd.so.1为例说明)
会看到搜索结果为我们列举了不同系统版本下对应的依赖（系统版本的查看可以通过执行"cat /proc/version"等命令查看）
；举例来说如果系统版本是"amazon linux2" ；那么安装依赖的命令是
```shell
sudo yum install libzstd
```

## 修改配置文件
```
cd stonedb/install/
cp stonedb.cnf stonedb.cnf.bak
vi stonedb.cnf
```
主要修改路径，如果安装目录就是stonedb，只需要修改其它参数。
## 创建用户和目录
```
groupadd stonedb
useradd -g stonedb stonedb
passwd stonedb
cd stonedb/install/
mkdir binlog
mkdir log
mkdir tmp
mkdir redolog
mkdir undolog
chown -R stonedb:stonedb stonedb
```
## 初始化实例
```
/stonedb/install/bin/mysqld --defaults-file=/stonedb/install/stonedb.cnf --initialize-insecure --user=stonedb
```
初始化实例时，加参数--initialize-insecure是为了让管理员第一次登录是免密登录，登录后需要设置密码。
## 启动和关闭实例
```
/stonedb/install/bin/mysqld_safe --defaults-file=/stonedb/install/stonedb.cnf --user=stonedb &
mysqladmin -uroot -p -S /stonedb/install/tmp/mysql.sock shutdown
```
## 登录修改管理员密码
```
mysql -uroot -p -S /stonedb/install/tmp/mysql.sock
>set password = password('MYPASSWORD');
```
注：请将以上命令行中的MYPASSWORD设置为您想要的密码。
