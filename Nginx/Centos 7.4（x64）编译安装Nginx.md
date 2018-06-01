# Centos 7.4 (x64) 编译安装 Nginx 1.13.x


本教程是将PHP以PHP-FPM的方式安装，若想以其它形式安装，请查阅其它文档。

### 1.更新系统
```bash
yum update
```

### 1.安装必要的依赖库和工具
```bash
yum -y install make cmake gcc gcc-c++ gcc-g77 flex bison file libtool libtool-libs autoconf kernel-devel patch wget crontabs libjpeg libjpeg-devel libpng libpng-devel libpng10 libpng10-devel gd gd-devel libxml2 libxml2-devel zlib zlib-devel glib2 glib2-devel unzip tar bzip2 bzip2-devel libzip-devel libevent libevent-devel ncurses ncurses-devel curl curl-devel libcurl libcurl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel vim-minimal gettext gettext-devel ncurses-devel gmp-devel pspell-devel unzip libcap diffutils ca-certificates net-tools libc-client-devel psmisc libXpm-devel git-core c-ares-devel libicu-devel libxslt libxslt-devel xz expat-devel libaio-devel rpcgen libtirpc-devel
```
> 上述很多依赖的组件都是编译php扩展需要用到的，如果只是编译PHP，不附加扩展。则只需要安装很少的依赖即可。

## 路径约定：
-  **源码包路径** ： /usr/local/src

## 需要手动编译安装的包
- **pcre** ：review 规则依赖
- **openssl** ：Http 请求 SSL依赖
- **zlib** ：Http 请求依赖
- **nginx** ：Ninx 源码

## 下载安装包
```bash
cd /usr/local/src
wget https://ftp.pcre.org/pub/pcre/pcre-8.42.tar.gz
wget https://www.openssl.org/source/openssl-1.1.0h.tar.gz
wget http://www.zlib.net/zlib-1.2.11.tar.gz
wget http://nginx.org/download/nginx-1.13.12.tar.gz
```
> 以上软件源码包均是撰写文档时最新版本，如无法下载请自行寻找可以下载途径。

### 安装扩展包

#### pcre
```bash
cd /usr/local/src
tar -zxf pcre-8.42.tar.gz
cd pcre-8.42
./configure --prefix=/usr/local/pcre
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

### 安装 Nginx

#### 创建用户
```bash
groupadd www
useradd -g www www -s /bin/false
```
> Nginx 启动的时候会已 www 用户名 和 www用户组的权限运行。

#### 编译安装
```bash
cd /usr/local/src
tar -zxf nginx-1.13.12.tar.gz
cd nginx-1.13.12

./configure --prefix=/usr/local/nginx --without-http_memcached_module --user=www --group=www --with-http_stub_status_module --with-http_ssl_module --with-http_gzip_static_module --with-openssl=/usr/local/src/openssl-1.1.0h --with-zlib=/usr/local/src/zlib-1.2.11 --with-pcre=/usr/local/src/pcre-8.42

make
make install
```

#### Configure 参数详解
``` bash
./configure

# 设置 nginx 安装目录
--prefix=/usr/local/nginx
# 打开 memcached 模块
--without-http_memcached_module
# 设置用户
--user=www
# 设置用户组
--group=www
# 添加监控状态配置
--with-http_stub_status_module
# 打开 ssl 支持
--with-http_ssl_module
# 打开 gzip
--with-http_gzip_static_module
# 指定 ssl 模块依赖的 ssl 库
--with-openssl=/usr/local/src/openssl-1.1.0h
# 指定 gzip 模块依赖的 zlib 库
--with-zlib=/usr/local/src/zlib-1.2.11
# 指定 pcre 模块依赖的 pcre 库
--with-pcre=/usr/local/src/pcre-8.42
```
> 注意：--with-openssl=/usr/local/src/openssl-1.1.0h --with-zlib=/usr/local/src/zlib-1.2.11 --with-pcre=/usr/local/src/pcre-8.42 指向的是源码包解压的路径，而不是安装的路径，否则会报错

#### 设置开机启动
编辑 /etc/rc.d/init.d/nginx 启动文件添加下面内容( `vi /etc/rc.d/init.d/nginx` ):

``` bash
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig: - 85 15
# description: Nginx is an HTTP(S) server, HTTP(S) reverse \
# proxy and IMAP/POP3 proxy server
# processname: nginx
# config: /etc/nginx/nginx.conf
# config: /usr/local/nginx/conf/nginx.conf
# pidfile: /usr/local/nginx/logs/nginx.pid
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
nginx="/usr/local/nginx/sbin/nginx"
prog=$(basename $nginx)
NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
lockfile=/var/lock/subsys/nginx
make_dirs() {
# make required directories
user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
if [ -z "`grep $user /etc/passwd`" ]; then
useradd -M -s /bin/nologin $user
fi
options=`$nginx -V 2>&1 | grep 'configure arguments:'`
for opt in $options; do
if [ `echo $opt | grep '.*-temp-path'` ]; then
value=`echo $opt | cut -d "=" -f 2`
if [ ! -d "$value" ]; then
# echo "creating" $value
mkdir -p $value && chown -R $user $value
fi
fi
done
}

start() {
[ -x $nginx ] || exit 5
[ -f $NGINX_CONF_FILE ] || exit 6
make_dirs
echo -n $"Starting $prog: "
daemon $nginx -c $NGINX_CONF_FILE
retval=$?
echo
[ $retval -eq 0 ] && touch $lockfile
return $retval
}

stop() {
echo -n $"Stopping $prog: "
killproc $prog -QUIT
retval=$?
echo
[ $retval -eq 0 ] && rm -f $lockfile
return $retval
}

restart() {
#configtest || return $?
stop
sleep 1
start
}

reload() {
#configtest || return $?
echo -n $"Reloading $prog: "
killproc $nginx -HUP
RETVAL=$?
echo
}

force_reload() {
restart
}

configtest() {
$nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
status $prog
}

rh_status_q() {
rh_status >/dev/null 2>&1
}

case "$1" in
start)

rh_status_q && exit 0
$1
;;
stop)

rh_status_q || exit 0
$1
;;

restart|configtest)
$1
;;
reload)
rh_status_q || exit 7
$1
;;
force-reload)
force_reload
;;
status)
rh_status
;;
condrestart|try-restart)
rh_status_q || exit 0
;;
*)
echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
exit 2
esac
```
保存文件。( `:wq!` )

继续：
```bash
#赋予文件执行权限
chmod 775 /etc/rc.d/init.d/nginx 

#设置开机启动
chkconfig nginx on

#重启
/etc/rc.d/init.d/nginx restart 
```

### 配置 Nginx 支持 PHP

编辑 nginx.conf ( `vi /usr/local/nginx/conf/nginx.conf` )
```base

#首行user去掉注释,修改Nginx运行组为www www；必须与/usr/local/php/etc/php-fpm.conf中的user,group配置相同，否则php运行出错
user www www; 

# 添加index.php
index index.html index.htm index.php; 

# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000 or unix:/tmp/php-cgi.sock
#
location ~ \.php$ {
    root html;
    fastcgi_pass unix:/tmp/php-cgi.sock;
    fastcgi_index index.php;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}

#取消FastCGI server部分location的注释,注意fastcgi_param行的参数,改为$document_root$fastcgi_script_name,或者使用绝对路径

#重启nginx
/etc/init.d/nginx restart
or
service nginx restart 

#启动php-fpm
service php-fpm start 
```