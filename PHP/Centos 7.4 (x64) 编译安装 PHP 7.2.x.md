# Centos 7.4 (x64) 编译安装 PHP 7.2.x

### 1.更新系统
```bash
yum update
```

### 1.安装必要的依赖库和工具
```bash
yum -y install make cmake gcc gcc-c++ gcc-g77 flex bison file libtool libtool-libs autoconf kernel-devel patch wget crontabs libjpeg libjpeg-devel libpng libpng-devel libpng10 libpng10-devel gd gd-devel libxml2 libxml2-devel zlib zlib-devel glib2 glib2-devel unzip tar bzip2 bzip2-devel libzip-devel libevent libevent-devel ncurses ncurses-devel curl curl-devel libcurl libcurl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel vim-minimal gettext gettext-devel ncurses-devel gmp-devel pspell-devel unzip libcap diffutils ca-certificates net-tools libc-client-devel psmisc libXpm-devel git-core c-ares-devel libicu-devel libxslt libxslt-devel xz expat-devel libaio-devel rpcgen libtirpc-devel
```

## 路径约定：
-  **源码包路径** ： /usr/local/src

## 需要手动编译安装的包
- **autoconf** ：Shell脚本的工具
- **freetype** ：GD库依赖
- **curl** ：PHP网络请求依赖
- **openssl** ：SSL依赖
- **zlib** ：PHP压缩依赖
- **php** ：PHP源码

## 下载安装包
```bash
cd /usr/local/src
wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.13.tar.gz
wget https://download.savannah.gnu.org/releases/freetype/freetype-2.9.1.tar.gz
wget https://curl.haxx.se/download/curl-7.60.0.tar.gz
wget https://www.openssl.org/source/openssl-1.1.0h.tar.gz
wget http://www.zlib.net/zlib-1.2.11.tar.gz
wget http://cn2.php.net/distributions/php-7.2.6.tar.gz
```
> 以上软件源码包均是撰写文档时最新版本，如无法下载请自行寻找可以下载途径。

### 安装扩展包

#### autoconf
```bash
cd /usr/local/src
tar -zxf autoconf-2.13.tar.gz
cd autoconf-2.13
./configure --prefix=/usr/local/autoconf
make
make install
export PHP_AUTOCONF=/usr/local/autoconf/bin/autoconf
export PHP_AUTOHEADER=/usr/local/autoconf/bin/autoheader
```

#### freetype
```bash
cd /usr/local/src
tar -zxf freetype-2.9.1.tar.gz
cd freetype-2.9.1
./configure --prefix=/usr/local/freetype
make
make install
```

#### curl
```bash
cd /usr/local/src
tar -zxf curl-7.60.0.tar.gz
cd curl-7.60.0
./configure --prefix=/usr/local/curl --enable-ares --without-nss --with-ssl
make
make install
```

#### openssl
```bash
cd /usr/local/src
tar -zxf openssl-1.1.0h.tar.gz
cd openssl-1.1.0h
./config -fPIC --prefix=/usr/local/openssl --openssldir=/usr/local/openssl
make depend
make
make install

# 编辑 /etc/profile 文件，加入以下内容
vi /etc/profile
export PATH=$PATH:/usr/local/openssl/bin
:wq!
```


#### zlib
```bash
cd /usr/local/src
tar -zxf zlib-1.2.11.tar.gz
cd zlib-1.2.11
./configure --prefix=/usr/local/zlib
make
make install
```

### 安装PHP

#### 创建用户
```bash
groupadd www
useradd -g www www -s /bin/false
```
> php-fpm 启动的时候会已 www 用户名 和 www用户组的权限运行。

#### 编译安装
```bash
cd /usr/local/src
tar -zxf php-7.2.6.tar.gz
cd php-7.2.6

./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --with-config-file-scan-dir=/usr/local/php/conf.d --enable-fpm --with-fpm-user=www --with-fpm-group=www --enable-mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --with-iconv-dir --with-freetype-dir=/usr/local/freetype --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --disable-rpath --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --enable-mbregex --enable-mbstring --enable-intl --enable-pcntl --enable-ftp --with-gd--with-mhash --enable-sockets --with-xmlrpc --enable-zip --enable-soap --with-gettext --enable-opcache --with-xsl --disable-fileinfo --with-openssl --with-curl=/usr/local/curl --with-libdir=lib64

make
make install
```
#### Configure 参数详解
``` bash
./configure 

# 设置php的安装路径
--prefix=/usr/local/php 
# 设置 php.ini 的路径
--with-config-file-path=/usr/local/php/etc 
# 设置 php 的配置文件目录
--with-config-file-scan-dir=/usr/local/php/conf.d 
# 选择使用 PHP-FPM 
--enable-fpm 
# 设置 FPM 的用户
--with-fpm-user=www 
# 设置 FPM 的用户组
--with-fpm-group=www 
# 打开 mysql 驱动
--enable-mysqlnd
# 打开 mysqli 扩展 
--with-mysqli=mysqlnd 
# 打开 pdo-mysql 扩展
--with-pdo-mysql=mysqlnd 
# 打开 iconv 支持，用于 PHP 编译时指定 iconv 在系统里的路径
--with-iconv-dir 
# 打开 freetype 支持 
--with-freetype-dir=/usr/local/freetype 
# 打开对 jpeg 图片的支持  
--with-jpeg-dir 
# 打开对 png 图片的支持 
--with-png-dir 
# 打开 zlib 库的支持，用于http压缩传输
--with-zlib
# 打开 XML 扩展 
--with-libxml-dir=/usr 
--enable-xml 
# 禁用在搜索路径中传递其他运行库
--disable-rpath 
# 添加 bcmath 扩展 
--enable-bcmath 
# 使得你的PHP系统可以处理相关的IPC函数了
--enable-shmop 
--enable-sysvsem 
# 优化线程
--enable-inline-optimization 
# 打开 mbregex 扩展
--enable-mbregex 
# 打开 mbstring 扩展
--enable-mbstring 
# 打开 intl 是国际化扩展
--enable-intl 
#freeTDS需要用到的，可能是链接mssql 才用到
--enable-pcntl 
# 打开ftp的支持
--enable-ftp 
# 打开 GD 库
--with-gd 
# 打开 mhash 算法扩展
--with-mhash 
# 打开 sockets 支持
--enable-sockets 
# 打开xml-rpc的c语言 
--with-xmlrpc 
# 打开对zip的支持 
--enable-zip 
# 打开 soap 支持
--enable-soap 
# 打开gnu 的gettext 支持，编码库用到 
--with-gettext 
# 打开 opcache
--enable-opcache 
# 打开XSLT 文件支持，扩展了libXML2库 ，需要libxslt软件 
--with-xsl 
# 打开 fileinfo 扩展
--disable-fileinfo
# openssl的支持，加密传输https时用到的
--with-openssl
# 打开curl浏览工具的支持 
--with-curl=/usr/local/curl
--with-libdir=lib64
```
> 更多配置选项请查看手册:  http://php.net/manual/zh/configure.about.php

#### php 配置
```bash
# 创建系统软链接，放入/usr/bing/下
ln -sf /usr/local/php/bin/php /usr/bin/php
ln -sf /usr/local/php/bin/phpize /usr/bin/phpize
ln -sf /usr/local/php/bin/pear /usr/bin/pear
ln -sf /usr/local/php/bin/pecl /usr/bin/pecl
ln -sf /usr/local/php/sbin/php-fpm /usr/bin/php-fpm

# 删除默认配置
rm -f /usr/local/php/conf.d/*

# 建立事先约定好的目录
mkdir -p /usr/local/php/etc
mkdir -p /usr/local/php/conf.d

# 拷贝 php.ini 文件
cp php.ini-production /usr/local/php/etc/php.ini

# 自动修改 php.ini 选项，这几项大家应该都能看懂，就不再细说了
sed -i 's/post_max_size =.*/post_max_size = 50M/g' /usr/local/php/etc/php.ini
sed -i 's/upload_max_filesize =.*/upload_max_filesize = 50M/g' /usr/local/php/etc/php.ini
sed -i 's/;date.timezone =.*/date.timezone = PRC/g' /usr/local/php/etc/php.ini
sed -i 's/short_open_tag =.*/short_open_tag = On/g' /usr/local/php/etc/php.ini
sed -i 's/;cgi.fix_pathinfo=.*/cgi.fix_pathinfo=0/g' /usr/local/php/etc/php.ini
sed -i 's/max_execution_time =.*/max_execution_time = 300/g' /usr/local/php/etc/php.ini
sed -i 's/disable_functions =.*/disable_functions = passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server/g' /usr/local/php/etc/php.ini

# 设置 pear , pecl 使用 php.ini 的路径
pear config-set php_ini /usr/local/php/etc/php.ini
pecl config-set php_ini /usr/local/php/etc/php.ini

# 自动创建 php-fpm.conf 配置文件
cat >/usr/local/php/etc/php-fpm.conf<<EOF
[global]
pid = /usr/local/php/var/run/php-fpm.pid
error_log = /usr/local/php/var/log/php-fpm.log
log_level = notice

[www]
listen = /tmp/php-cgi.sock
listen.backlog = -1
listen.allowed_clients = 127.0.0.1
listen.owner = www
listen.group = www
listen.mode = 0666
user = www
group = www
pm = dynamic
pm.max_children = 10
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 6
request_terminate_timeout = 100
request_slowlog_timeout = 0
slowlog = var/log/slow.log
EOF

# 以上配置均是根据当前服务器配置和使用场景来调整，已达到最优效果
# pm.max_children : pm 设置为 dynamic 时表示最大可创建的子进程的数量
# pm.start_servers : pm 设置为 dynamic 设置启动时创建的子进程数目
# pm.min_spare_servers : 设置空闲服务进程的最低数目。仅在 pm 设置为 dynamic 时使用。必须设置。
# pm.max_spare_servers : 设置空闲服务进程的最大数目。仅在 pm 设置为 dynamic 时使用。必须设置。
# request_terminate_timeout : 设置单个请求的超时中止时间
# request_slowlog_timeout : 当一个请求该设置的超时时间后，就会将对应的 PHP 调用堆栈信息完整写入到慢日志中。
# slowlog : 慢请求的记录日志.
# php-fpm.conf 更多配置选项：http://php.net/manual/zh/install.fpm.configuration.php

# 拷贝 php-fpm 到 /etc/init.d 目录下
cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
# 给 php-fpm 添加执行权限
chmod +x /etc/init.d/php-fpm

# 将 php-fpm 加入开机启动
chkconfig php-fpm on

# 启动 php-fpm
service php-fpm start

# 停止 php-fpm
service php-fpm stop
```