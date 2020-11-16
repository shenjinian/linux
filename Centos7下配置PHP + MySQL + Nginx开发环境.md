**Centos7下配置PHP + MySQL + Nginx开发环境**

**一. Nginx安装与配置**

**1.安装**

```
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm

yum -y install nginx
systemctl start nginx
systemctl enable nginx
```

 **2.配置：**

```
 vim /etc/nginx/nginx.conf

server {
listen 80;
server*name localhost;
autoindex on;
\#charset koi8-r;
\#access*log /var/log/nginx/log/host.access.log main;

location / { 
  root  /var/www/html; 
  index index.html index.htm index.php; 
} 

location ~ \.php$ { 
  root      /var/www/html; 
  fastcgi_pass  127.0.0.1:9000; 
  fastcgi_index index.php; 
  fastcgi_param SCRIPT_FILENAME /var/www/html$fastcgi_script_name; 
  include    fastcgi_params; 
}
```

 

**二. MySQL安装与配置**

**1.** **配置yum源**

```
#更新yum源
yum update    

# 下载mysql源安装包
wget http://dev.mysql.com/get/mysql57-community-release-el7-8.noarch.rpm

# 安装mysql源 
yum localinstall mysql57-community-release-el7-8.noarch.rpm

# 检查mysql源是否安装成功
yum repolist enabled | grep "mysql.*-community.*"
```

**2.** **安装MySQL**

```
yum install mysql-community-server
```

**3.** **启动MySQL**

```
# 启动MySQL服务
systemctl start mysqld

# 查看MySQL的启动状态
systemctl status mysqld

# 设置MySQL开机启动
systemctl enable mysqld
systemctl daemon-reload
```

**4.** **修改root默认密码**

```
# 找到root默认密码
grep 'temporary password' /var/log/mysqld.log

# 进入mysql控制台, 输入上述查询到的默认密码
mysql -uroot -p

# 设置root管理员的密码
set password for 'root'@'localhost'=password('PassWord123@');
```

**5.添加远程登录用户**
默认只允许root帐户在本地登录，如果要在其它机器上连接mysql，必须修改root允许远程连接，或者添加一个允许远程连接的帐户

```
# 添加远程帐户
GRANT ALL PRIVILEGES ON *.* TO 'yourname'@'%' IDENTIFIED BY 'YourPassword@123' WITH GRANT OPTION;
```

**6.** **配置默认编码为utf8**
修改配置文件 /etc/my.cnf，添加下面两行, utf8编码配置

```
character_set_server=utf8
iit_connect='SET NAMES utf8'
```


**三. PHP环境配置**

**1.** **安装 php 和 php-fpm**

```
# 首先安装php72的源
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm

#检查源是否安装成功
yum repolist enabled | grep "webtatic*"

# 安装php71和对应的扩展
yum -y install php72w php72w-fpm
yum -y install php72w-mbstring php72w-common php72w-gd php72w-mcrypt
yum -y install php72w-mysql php72w-xml php72w-cli php72w-devel
yum -y install php72w-pecl-memcached php72w-pecl-redis php72w-opcache

#验证php7.1.x和扩展是否安装成功

验证php是否安装成功
php -v 

验证对应的扩展是否安装成功
php -m

#修改php-fpm 配置
vim /etc/php-fpm.d/www.conf
user = nginx
group = nginx
```

**2.** **设置php-fpm开机自动启动**

```
systemctl enable php-fpm
```

**3.** **启动php-fpm**

```
systemctl start php-fpm
```

 **4检查开机自启动是否设置成功**

```
systemctl list-dependencies | grep php-fpm
ps -ef | grep php-fpm
```

 

**常用指令**

**mysql**

```
systemctl start mysqld # 启动
systemctl stop mysqld # 停止
systemctl restart mysqld # 重启
```

**php-fpm**

```
systemctl start php-fpm # 启动
systemctl stop php-fpm # 停止
systemctl restart php-fpm # 重启
```

**nginx**

```
sudo fuser -k 80/tcp # 杀死80端口
/usr/sbin/nginx # 开启
/usr/sbin/nginx -s stop # 停止
/usr/sbin/nginx -s reopen # 重启
/usr/sbin/nginx -s reload # 重新载入配置文件
```

**其他问题**

**1.** **关闭SELINUX(SELINUX是一个安全子系统，它能控制程序只能访问特定文件。如果不关闭，你可能访问文件受限):**

```
vi /etc/selinux/config

#SELINUX=enforcing    # 注释掉
#SELINUXTYPE=targeted   # 注释掉
SELINUX=disabled     # 增加

:wq!           # 保存退出

shutdown -r now      # 重启系统
```

**2. thinkphp** **提示错误目录 [ ./Runtime/ ] 不可写！**

```
chmod 777 -R /var/www/xxx项目目录/Application/Runtime
```

