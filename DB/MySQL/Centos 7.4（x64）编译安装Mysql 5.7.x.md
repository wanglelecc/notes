# Centos 7.4 (x64) 编译安装 Mysql 5.7.x


### 1.更新系统
```bash
yum update
```

### 1.安装必要的依赖库和工具
```bash
yum -y install make cmake gcc gcc-c++ gcc-g77 flex bison file libtool libtool-libs autoconf kernel-devel patch wget crontabs libjpeg libjpeg-devel libpng libpng-devel libpng10 libpng10-devel gd gd-devel libxml2 libxml2-devel zlib zlib-devel glib2 glib2-devel unzip tar bzip2 bzip2-devel libzip-devel libevent libevent-devel ncurses ncurses-devel curl curl-devel libcurl libcurl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel vim-minimal gettext gettext-devel ncurses-devel gmp-devel pspell-devel unzip libcap diffutils ca-certificates net-tools libc-client-devel psmisc libXpm-devel git-core c-ares-devel libicu-devel libxslt libxslt-devel xz expat-devel libaio-devel rpcgen libtirpc-devel python-devel
```
> 上述很多依赖的组件都是编译php扩展需要用到的，如果只是编译cmake，不附加扩展。则只需要安装很少的依赖即可。

## 路径约定：
-  **源码包路径** ： /usr/local/src

## 下载安装包
```bash
cd /usr/local/src
wget http://mirrors.ukfast.co.uk/sites/ftp.mysql.com/Downloads/MySQL-5.7/mysql-5.7.22.tar.gz
wget https://ufpr.dl.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz
```
> 以上软件源码包均是撰写文档时最新版本，如无法下载请自行寻找可以下载途径。

### 安装扩展包

#### boost
```bash
cd /usr/local/src
mkdir -p /usr/local/boost
cp boost_1_59_0.tar.gz /usr/local/boost
```

### 安装 Nginx

#### 创建用户
```bash
groupadd mysql
useradd -s /sbin/nologin -M -g mysql mysql
```
> Mysql 启动的时候会已 mysql 用户名 和 mysql 用户组的权限运行。

#### 编译安装
```bash
#创建MySQL数据库存放目录
mkdir -p /data/mysql 
#设置MySQL数据库存放目录权限
chown -R mysql:mysql /data/mysql 
#创建MySQL安装目录
mkdir -p /usr/local/mysql 

cd /usr/local/src
tar -zxf mysql-5.7.22.tar.gz
cd mysql-5.7.22

cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql -DSYSCONFDIR=/etc -DWITH_MYISAM_STORAGE_ENGINE=1 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DMYSQL_USER=mysql -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DWITH_FEDERATED_STORAGE_ENGINE=1 -DEXTRA_CHARSETS=all -DDEFAULT_CHARSET=utf8mb4 -DDEFAULT_COLLATION=utf8mb4_general_ci -DWITH_EMBEDDED_SERVER=1 -DENABLED_LOCAL_INFILE=1 -DWITH_BOOST=/usr/local/boost

make
make install
```

#### Mysql 配置
```bash
# 编译出错, 重新编译前要删除编译失败的文件，重新编译时，需要清除旧的对象文件和缓存信息。

make clean
rm -f CMakeCache.txt
# 删除系统默认的配置文件（如果默认没有就不用删除）
rm -rf /etc/my.cnf 
# 进入MySQL安装目录
cd /usr/local/mysql 

# 生成mysql系统数据库
./bin/mysqld --user=mysql --initialize --basedir=/usr/local/mysql --datadir=/data/mysql 

# --initialize表示默认生成密码, --initialize-insecure 表示不生成密码, 密码为空。看到这一行[Note] A temporary password is generated for root@localhost: i>X18*=Rav=7
cp /usr/local/mysql/support-files/my-default.cnf /usr/local/mysql/my.cnf

# 添加到/etc目录的软连接
ln -s /usr/local/mysql/my.cnf /etc/my.cnf 

# 把Mysql加入系统启动
cp /usr/local/mysql/support-files/mysql.server /etc/rc.d/init.d/mysqld

# 增加执行权限
chmod 755 /etc/init.d/mysqld 

# 加入开机启动
chkconfig mysqld on 

# 编辑
vi /etc/rc.d/init.d/mysqld 
# MySQL程序安装路径
basedir=/usr/local/mysql 
# MySQl数据库存放目录
datadir=/data/mysql 
:wq! #保存退出

# 启动
service mysqld start 

 #把mysql服务加入系统环境变量：在最后添加下面这一行
vi /etc/profile
export PATH=$PATH:/usr/local/mysql/bin
:wq! #保存退出

# 使配置立刻生效
source /etc/profile 

# 下面这两行把myslq的库文件链接到系统默认的位置，这样你在编译类似PHP等软件时可以不用指定mysql的库文件地址。
ln -s /usr/local/mysql/lib/mysql /usr/lib/mysql
ln -s /usr/local/mysql/include/mysql /usr/include/mysql
mkdir /var/lib/mysql #创建目录
ln -s /tmp/mysql.sock /var/lib/mysql/mysql.sock #添加软链接

# 修改Mysql密码，输入之前生成的密CSJlm3DyTG.d回车，根据提示操作。
mysql_secure_installation 

#是否安装密码安全插件？选择y
Press y|Y for Yes, any other key for No: y 

# 有以下几种密码强度选择
There are three levels of password validation policy:

LOW Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary file
Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 0 #选择0,只要8位数字即可，选1要有大写，小写，特殊字符等

UNINSTALL PLUGIN validate_password ; #卸载密码强度插件

use mysql;

# 登录mysql控制台修改
update mysql.user set authentication_string=password('123456') where user='root' ;  

# 修改密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password AS '123456'; 
```