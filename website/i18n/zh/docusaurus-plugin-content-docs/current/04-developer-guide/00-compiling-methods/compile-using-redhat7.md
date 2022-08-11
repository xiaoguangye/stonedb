---
id: compile-using-redhat7
sidebar_position: 5.13
---

# RedHat 7 下编译StoneDB

编译工具以及第三方库的版本要求如下。

| 编译工具及第三方库 | 版本要求 |
| --- | --- |
| gcc | 9.3.0 |
| make | 3.82 |
| cmake | 3.7.2 |
| marisa | 0.77 |
| rocksdb | 6.12.6 |
| boost | 1.66 |

## 第一步：安装依赖包
```shell
yum install -y tree
yum install -y gcc
yum install -y gcc-c++
yum install -y libzstd-devel
yum install -y make
yum install -y ncurses
yum install -y ncurses-devel
yum install -y bison
yum install -y libaio
yum install -y perl
yum install -y perl-DBI
yum install -y perl-DBD-MySQL
yum install -y perl-Time-HiRes
yum install -y readline-devel
yum install -y numactl
yum install -y zlib
yum install -y zlib-devel
yum install -y openssl
yum install -y openssl-devel
yum install -y redhat-lsb-core
yum install -y git
yum install -y autoconf
yum install -y automake
yum install -y libtool
yum install -y lrzsz
yum install -y lz4
yum install -y lz4-devel
yum install -y snappy
yum install -y snappy-devel
yum install -y bzip2
yum install -y bzip2-devel
yum install -y zstd
yum install -y libedit
yum install -y libedit-devel
yum install -y libaio-devel
yum install -y libicu
yum install -y libicu-devel
```
## 第二步：安装 gcc 9.3.0
通过执行以下语句，检查当前 gcc 版本是否符合安装要求。
```shell
gcc --version
```
如果版本不符合要求，按照以下步骤将 gcc 切换为正确版本。
### 1. 安装 scl 源
```shell
yum install centos-release-scl scl-utils-build -y
```
### 2. 安装 9.3.0 版本的 gcc、gcc-c++、gdb 
```shell
yum install devtoolset-9-gcc.x86_64 devtoolset-9-gcc-c++.x86_64 devtoolset-9-gcc-gdb-plugin.x86_64 -y
```
### 3. 切换至 9.3.0 版本
```shell
scl enable devtoolset-9 bash
```
### 4. 版本检查
```shell
gcc --version
```
## 第三步：安装第三方库
安装第三库前需要确认 cmake 版本是3.7.2以上，make 版本是3.82以上，如果低于这两个版本，需要进行安装。StoneDB 依赖 marisa、rocksdb、boost，在编译 marisa、rocksdb、boost 时，可以不指定安装路径，默认安装路径在 /usr/local 下。如果不指定 marisa、rocksdb、boost 的安装路径，在编译安装 StoneDB 时也无需指定路径。示例中我们指定了 marisa、rocksdb、boost 的安装路径。
### 1. 安装 cmake
```shell
wget https://cmake.org/files/v3.7/cmake-3.7.2.tar.gz
tar -zxvf cmake-3.7.2.tar.gz
cd cmake-3.7.2
./bootstrap && make && make install
/usr/local/bin/cmake --version
rm -rf /usr/bin/cmake
ln -s /usr/local/bin/cmake /usr/bin/
```
### 2. 安装 make
```shell
http://mirrors.ustc.edu.cn/gnu/make/
tar -zxvf make-3.82.tar.gz
./configure  --prefix=/usr/local/make
make && make install
rm -rf /usr/local/bin/make
ln -s /usr/local/make/bin/make /usr/local/bin/make
```
### 3. 安装 marisa 
```shell
git clone https://github.com/s-yata/marisa-trie.git
cd marisa-trie
autoreconf -i
./configure --enable-native-code --prefix=/usr/local/stonedb-marisa
make && make install 
```
marisa 的安装路径可以根据实际情况指定，示例中的安装路径是 /usr/local/stonedb-marisa。
### 4. 安装 rocksdb 
```shell
wget https://github.com/facebook/rocksdb/archive/refs/tags/v6.12.6.tar.gz
tar -zxvf v6.12.6.tar.gz
cd rocksdb-6.12.6

cmake ./ \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_INSTALL_PREFIX=/usr/local/stonedb-gcc-rocksdb \
  -DCMAKE_INSTALL_LIBDIR=/usr/local/stonedb-gcc-rocksdb \
  -DWITH_JEMALLOC=ON \
  -DWITH_SNAPPY=ON \
  -DWITH_LZ4=ON \
  -DWITH_ZLIB=ON \
  -DWITH_ZSTD=ON \
  -DUSE_RTTI=ON \
  -DROCKSDB_BUILD_SHARED=ON \
  -DWITH_GFLAGS=OFF \
  -DWITH_TOOLS=OFF \
  -DWITH_BENCHMARK_TOOLS=OFF \
  -DWITH_CORE_TOOLS=OFF 

make -j`nproc`
make install -j`nproc`
```
rocksdb 的安装路径可以根据实际情况指定，示例中的安装路径是 /usr/local/stonedb-gcc-rocksdb。
### 5. 安装 boost
```shell
wget https://sourceforge.net/projects/boost/files/boost/1.66.0/boost_1_66_0.tar.gz
tar -zxvf boost_1_66_0.tar.gz
cd boost_1_66_0
./bootstrap.sh --prefix=/usr/local/stonedb-boost
./b2 install --with=all
```
boost 的安装路径可以根据实际情况指定，示例中的安装路径是 /usr/local/stonedb-boost。<br />注：在编译过程中，除非有关键字 "error" 报错自动退出，否则出现关键字 "warning"、"failed"是正常的。
## 第四步：执行编译
StoneDB 现有 5.6 和 5.7 两个分支，下载的源码包默认是 5.7 分支。下载的源码包存放路径可根据实际情况指定，示例中的源码包存放路径是在根目录下，并且是切换为 5.6 分支的编译安装。<br />注：gcc 9.3.0以上版本已经支持 5.6 的编译，并且支持自定义指定 rocksdb 和 marisa 的安装路径。5.7 的编译还是 gcc 7.3.0版本，后续会得到支持。
```shell
cd /
git clone https://github.com/stoneatom/stonedb.git
cd stonedb
git checkout remotes/origin/stonedb-5.6
```
在执行编译脚本前，需要修改编译脚本的两处内容：<br />1）StoneDB 安装目录，可根据实际情况修改，示例中的安装目录是 /stonedb56/install；<br />2）marisa、rocksdb、boost 的实际安装路径，必须与上文安装 marisa、rocksdb、boost 的路径保持一致。
```shell
###修改编译脚本
cd /stonedb/scripts
vim stonedb_build.sh
...
install_target=/stonedb56/install
...
-DDOWNLOAD_BOOST=0 \
-DWITH_BOOST=/usr/local/stonedb-boost/ \
-DWITH_MARISA=/usr/local/stonedb-marisa \
-DWITH_ROCKSDB=/usr/local/stonedb-gcc-rocksdb \
2>&1 | tee -a ${build_log}

###执行编译脚本
sh stonedb_build.sh
```
## 第五步：启动实例
按照以下步骤启动 StoneDB。
### 1. 创建用户
```shell
groupadd mysql
useradd -g mysql mysql
passwd mysql
```
### 2. 手动安装
编译完成后，如果 StoneDB 安装目录不是 /stonedb56，不会自动生成 reinstall.sh、install.sh 和 my.cnf 文件，需要手动创建目录、初始化和启动实例。还需要配置 my.cnf 文件，如安装目录，端口等参数。
```shell
###创建目录
mkdir -p /data/stonedb56/install/data/innodb
mkdir -p /data/stonedb56/install/binlog
mkdir -p /data/stonedb56/install/log
mkdir -p /data/stonedb56/install/tmp
chown -R mysql:mysql /data

###配置my.cnf
vim /data/stonedb56/install/my.cnf
[mysqld]
port      = 3306
socket    = /data/stonedb56/install/tmp/mysql.sock
datadir   = /data/stonedb56/install/data
pid-file  = /data/stonedb56/install/data/mysqld.pid
log-error = /data/stonedb56/install/log/mysqld.log

chown -R mysql:mysql /data/stonedb56/install/my.cnf

###初始化实例
/data/stonedb56/install/scripts/mysql_install_db --datadir=/data/stonedb56/install/data --basedir=/data/stonedb56/install --user=mysql

###启动实例
/data/stonedb56/install/bin/mysqld_safe --defaults-file=/data/stonedb56/install/my.cnf --user=mysql &
```
### 3. 自动安装
编译完成后，如果 StoneDB 安装目录是 /stonedb56，会自动生成 reinstall.sh、install.sh 和 my.cnf 文件，执行 reinstall.sh 就是创建目录、初始化实例和启动实例的过程。
```shell
cd /stonedb56/install
./reinstall.sh
```
注：reinstall.sh 与 install.sh 的区别？<br />reinstall.sh 是自动化安装脚本，执行脚本的过程是创建目录、初始化实例和启动实例的过程，只在第一次使用，其他任何时候使用都会删除整个目录，重新初始化数据库。install.sh 是手动安装提供的示例脚本，用户可根据自定义的安装目录修改路径，然后执行脚本，执行脚本的过程也是创建目录、初始化实例和启动实例。以上两个脚本都只能在第一次使用。
### 4. 执行登录
```shell
/stonedb56/install/bin/mysql -uroot -p -S /stonedb56/install/tmp/mysql.sock
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.24-StoneDB-debug build-

Copyright (c) 2000, 2022 StoneAtom Group Holding Limited
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| cache              |
| innodb             |
| mysql              |
| performance_schema |
| sys_stonedb        |
| test               |
+--------------------+
7 rows in set (0.00 sec)
```