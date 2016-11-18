---
layout: post
title: centos下配置Apache+Nginx+PHP+MariaDB环境--编译篇
category: 技术
author: 沈伯伟
tags: Apache Nginx PHP MariaDB
---

* 目录
{:toc}

因为之前我们已经安装、配置了Apache+Nginx+PHP+MariaDB环境，
所以编译安装之前，需要将之前安装的相关软件卸载掉……

> 那么为什么还要费劲编译? 直接安装打包好的二进制软件包不就好了。
>
> 原因是二进制软件包是软件作者开发完成某阶段发布出来的，无论是对于使用者还是平台都没有很强的定制性和适用性。
> 而通过编译源代码的方式进行软件安装，则可以在代码层面对软件进行个人定制，并且针对不同平台，可以实现性能优化等。

## 卸载Apache+Nginx+PHP+MariaDB

### 卸载 Apache

```
sudo yum remove httpd
```

运行结果如下：

```
[shebnowei@bogon ~]$ sudo yum remove httpd
[sudo] password for shebnowei: 
已加载插件：fastestmirror, langpacks
正在解决依赖关系
--> 正在检查事务
---> 软件包 httpd.x86_64.0.2.4.6-40.el7.centos.4 将被 删除
--> 正在处理依赖关系 httpd-mmn = 20120211x8664，它被软件包 php-5.4.16-36.3.el7_2.x86_64 需要
--> 正在检查事务
---> 软件包 php.x86_64.0.5.4.16-36.3.el7_2 将被 删除
--> 解决依赖关系完成
base/7/x86_64                                            | 3.6 kB     00:00     
extras/7/x86_64                                          | 3.4 kB     00:00     
nginx/x86_64                                             | 2.9 kB     00:00     
updates/7/x86_64                                         | 3.4 kB     00:00     

依赖关系解决

================================================================================
 Package      架构          版本                          源               大小
================================================================================
正在删除:
 httpd        x86_64        2.4.6-40.el7.centos.4         @updates        9.4 M
为依赖而移除:
 php          x86_64        5.4.16-36.3.el7_2             @updates        4.4 M

事务概要
================================================================================
移除  1 软件包 (+1 依赖软件包)

安装大小：14 M
是否继续？[y/N]：y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在删除    : php-5.4.16-36.3.el7_2.x86_64                                1/2 
  正在删除    : httpd-2.4.6-40.el7.centos.4.x86_64                          2/2 
  验证中      : php-5.4.16-36.3.el7_2.x86_64                                1/2 
  验证中      : httpd-2.4.6-40.el7.centos.4.x86_64                          2/2 

删除:
  httpd.x86_64 0:2.4.6-40.el7.centos.4                                          

作为依赖被删除:
  php.x86_64 0:5.4.16-36.3.el7_2                                                

完毕！
```

从上述信息可以看出`httpd`被`php`所依赖，因此为了避免卸载`httpd`后`php`无法使用，同时提示卸载`php`（但是并不是完全移除了`php`,`php`依然存在）。
接下来我们检查的`httpd`服务是否存在：

```
[shebnowei@bogon conf.d]$ systemctl start httpd.service
Failed to start httpd.service: Unit httpd.service failed to load: No such file or directory.
[shebnowei@bogon conf.d]$ which httpd
/usr/bin/which: no httpd in (/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/home/shebnowei/.local/bin:/home/shebnowei/bin)
[shebnowei@bogon conf.d]$ httpd -v
bash: httpd: 未找到命令...
```

上述命令结果显示系统中的`httpd`服务和命令都已经被卸载了。
但是当我们通过`rpm -qa | grep httpd`查询和`httpd`相关的软件包残留时，却依然发现存在如下软件包：

```
[shebnowei@bogon conf.d]$ rpm -qa | grep httpd
httpd-tools-2.4.6-40.el7.centos.4.x86_64
```

其中`httpd-tools`这个包是在安装`httpd`时安装的依赖包(**注意，是httpd依赖于httpd-tools**)。
如果想进一步卸载它，可以：

```
[shebnowei@bogon conf.d]$ sudo yum remove httpd-tools
[sudo] password for shebnowei: 
已加载插件：fastestmirror, langpacks
正在解决依赖关系
--> 正在检查事务
---> 软件包 httpd-tools.x86_64.0.2.4.6-40.el7.centos.4 将被 删除
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package          架构        版本                          源             大小
================================================================================
正在删除:
 httpd-tools      x86_64      2.4.6-40.el7.centos.4         @updates      168 k

事务概要
================================================================================
移除  1 软件包

安装大小：168 k
是否继续？[y/N]：y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在删除    : httpd-tools-2.4.6-40.el7.centos.4.x86_64                    1/1 
  验证中      : httpd-tools-2.4.6-40.el7.centos.4.x86_64                    1/1 

删除:
  httpd-tools.x86_64 0:2.4.6-40.el7.centos.4                                    

完毕！
```

此时再查询相关的`httpd`软件包，则查不到结果：

```
[shebnowei@bogon conf.d]$ rpm -qa | grep httpd
[shebnowei@bogon conf.d]$ 
```

我们查看`Apache`的配置文件，发现残留`phpMyAdmin`的配置文件，而其本身的配置文件已经被移除。

```
[shebnowei@bogon conf.d]$ cd /etc/httpd/
[shebnowei@bogon httpd]$ ls -l
总用量 0
drwxr-xr-x. 2 root root 28 11月 14 19:33 conf.d
[shebnowei@bogon httpd]$ cd conf.d/
[shebnowei@bogon conf.d]$ ls -l
总用量 4
-rw-r--r--. 1 root root 1779 8月  29 01:29 phpMyAdmin.conf
```

### 卸载 nginx

```
[shebnowei@bogon ~]$ sudo yum remove nginx
[sudo] password for shebnowei: 
已加载插件：fastestmirror, langpacks
正在解决依赖关系
--> 正在检查事务
---> 软件包 nginx.x86_64.1.1.10.2-1.el7.ngx 将被 删除
--> 正在处理依赖关系 webserver，它被软件包 phpMyAdmin-4.4.15.8-2.el7.noarch 需要
--> 正在检查事务
---> 软件包 phpMyAdmin.noarch.0.4.4.15.8-2.el7 将被 删除
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package           架构          版本                       源             大小
================================================================================
正在删除:
 nginx             x86_64        1:1.10.2-1.el7.ngx         @nginx        2.2 M
为依赖而移除:
 phpMyAdmin        noarch        4.4.15.8-2.el7             @epel          24 M

事务概要
================================================================================
移除  1 软件包 (+1 依赖软件包)

安装大小：26 M
是否继续？[y/N]：y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在删除    : phpMyAdmin-4.4.15.8-2.el7.noarch                            1/2 
警告：/etc/phpMyAdmin/config.inc.php 已另存为 /etc/phpMyAdmin/config.inc.php.rpmsave
  正在删除    : 1:nginx-1.10.2-1.el7.ngx.x86_64                             2/2 
警告：/etc/nginx/conf.d/default.conf 已另存为 /etc/nginx/conf.d/default.conf.rpmsave
  验证中      : 1:nginx-1.10.2-1.el7.ngx.x86_64                             1/2 
  验证中      : phpMyAdmin-4.4.15.8-2.el7.noarch                            2/2 

删除:
  nginx.x86_64 1:1.10.2-1.el7.ngx                                               

作为依赖被删除:
  phpMyAdmin.noarch 0:4.4.15.8-2.el7                                            

完毕！
```

可以看到不仅卸载了`nginx`，同时移除了`phpMyAdmin`。

```
[shebnowei@bogon ~]$ which nginx
/usr/bin/which: no nginx in (/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/home/shebnowei/.local/bin:/home/shebnowei/bin)
[shebnowei@bogon ~]$ nginx -v
bash: nginx: 未找到命令...
[shebnowei@bogon ~]$ systemctl start nginx.service
Failed to start nginx.service: Unit nginx.service failed to load: No such file or directory.
[shebnowei@bogon ~]$ rpm -qa | grep nginx
nginx-release-centos-7-0.el7.ngx.noarch
```

可以看到，依然存在`nginx-release-centos-7-0.el7.ngx.noarch`,而这个是之前我们安装时所添加的`nginx`源。这里暂不处理。

因为移除了`phpMyAdmin`

```
[shebnowei@bogon conf.d]$ rpm -qi phpMyAdmin
未安装软件包 phpMyAdmin 
```

所以之前提到的配置文件`phpMyAdmin.conf`也不存在了。

```
[shebnowei@bogon conf.d]$ cd /etc/httpd/
[shebnowei@bogon httpd]$ ls -l
总用量 0
drwxr-xr-x. 2 root root 6 11月 14 21:08 conf.d
[shebnowei@bogon httpd]$ cd conf.d/
[shebnowei@bogon conf.d]$ ls -l
总用量 0
[shebnowei@bogon conf.d]$ 
```

### 卸载 php

因为之前安装`php`的时候，安装了不少组件，所以卸载的时候需要把所有组件都卸载掉。
首先查看安装的`php`相关软件包：

```
[shebnowei@bogon ~]$ rpm -qa | grep php
php-xmlrpc-5.4.16-36.3.el7_2.x86_64
php-tidy-5.4.16-5.el7.x86_64
php-pdo-5.4.16-36.3.el7_2.x86_64
php-gd-5.4.16-36.3.el7_2.x86_64
php-fpm-5.4.16-36.3.el7_2.x86_64
php-common-5.4.16-36.3.el7_2.x86_64
php-process-5.4.16-36.3.el7_2.x86_64
php-xml-5.4.16-36.3.el7_2.x86_64
php-odbc-5.4.16-36.3.el7_2.x86_64
php-tcpdf-dejavu-sans-fonts-6.2.11-1.el7.noarch
php-cli-5.4.16-36.3.el7_2.x86_64
php-pear-1.9.4-21.el7.noarch
php-mysql-5.4.16-36.3.el7_2.x86_64
php-bcmath-5.4.16-36.3.el7_2.x86_64
php-php-gettext-1.0.11-12.el7.noarch
php-ldap-5.4.16-36.3.el7_2.x86_64
php-mbstring-5.4.16-36.3.el7_2.x86_64
php-tcpdf-6.2.11-1.el7.noarch
```

完全卸载过程为：

```
sudo yum remove php-common
```

因为`php-common`是最核心的包，所以只需卸载它，其它依赖于它的包就会被相应卸载：

```
[shebnowei@bogon ~]$ sudo yum remove php-common
已加载插件：fastestmirror, langpacks
正在解决依赖关系
--> 正在检查事务
---> 软件包 php-common.x86_64.0.5.4.16-36.3.el7_2 将被 删除
--> 正在处理依赖关系 php(api) = 20100412-64，它被软件包 php-tidy-5.4.16-5.el7.x86_64 需要
--> 正在处理依赖关系 php(zend-abi) = 20100525-64，它被软件包 php-tidy-5.4.16-5.el7.x86_64 需要
--> 正在处理依赖关系 php(language) >= 5.3，它被软件包 php-tcpdf-6.2.11-1.el7.noarch 需要
--> 正在处理依赖关系 php-bz2，它被软件包 1:php-pear-1.9.4-21.el7.noarch 需要
--> 正在处理依赖关系 php-curl，它被软件包 php-tcpdf-6.2.11-1.el7.noarch 需要
--> 正在处理依赖关系 php-date，它被软件包 php-tcpdf-6.2.11-1.el7.noarch 需要
--> 正在处理依赖关系 php-ftp，它被软件包 1:php-pear-1.9.4-21.el7.noarch 需要
--> 正在处理依赖关系 php-hash，它被软件包 php-tcpdf-6.2.11-1.el7.noarch 需要
--> 正在处理依赖关系 php-json，它被软件包 php-tcpdf-6.2.11-1.el7.noarch 需要
--> 正在处理依赖关系 php-openssl，它被软件包 php-tcpdf-6.2.11-1.el7.noarch 需要
--> 正在处理依赖关系 php-pcre，它被软件包 1:php-pear-1.9.4-21.el7.noarch 需要
--> 正在处理依赖关系 php-pcre，它被软件包 php-tcpdf-6.2.11-1.el7.noarch 需要
--> 正在处理依赖关系 php-tokenizer，它被软件包 1:php-pear-1.9.4-21.el7.noarch 需要
--> 正在处理依赖关系 php-zlib，它被软件包 1:php-pear-1.9.4-21.el7.noarch 需要
--> 正在处理依赖关系 php-zlib，它被软件包 php-tcpdf-6.2.11-1.el7.noarch 需要
--> 正在处理依赖关系 php-common，它被软件包 php-php-gettext-1.0.11-12.el7.noarch 需要
--> 正在处理依赖关系 php-common(x86-64) = 5.4.16-36.3.el7_2，它被软件包 php-cli-5.4.16-36.3.el7_2.x86_64 需要
--> 正在处理依赖关系 php-common(x86-64) = 5.4.16-36.3.el7_2，它被软件包 php-ldap-5.4.16-36.3.el7_2.x86_64 需要
--> 正在处理依赖关系 php-common(x86-64) = 5.4.16-36.3.el7_2，它被软件包 php-gd-5.4.16-36.3.el7_2.x86_64 需要
--> 正在处理依赖关系 php-common(x86-64) = 5.4.16-36.3.el7_2，它被软件包 php-xmlrpc-5.4.16-36.3.el7_2.x86_64 需要
--> 正在处理依赖关系 php-common(x86-64) = 5.4.16-36.3.el7_2，它被软件包 php-fpm-5.4.16-36.3.el7_2.x86_64 需要
--> 正在处理依赖关系 php-common(x86-64) = 5.4.16-36.3.el7_2，它被软件包 php-xml-5.4.16-36.3.el7_2.x86_64 需要
--> 正在处理依赖关系 php-common(x86-64) = 5.4.16-36.3.el7_2，它被软件包 php-bcmath-5.4.16-36.3.el7_2.x86_64 需要
--> 正在处理依赖关系 php-common(x86-64) = 5.4.16-36.3.el7_2，它被软件包 php-mbstring-5.4.16-36.3.el7_2.x86_64 需要
--> 正在处理依赖关系 php-common(x86-64) = 5.4.16-36.3.el7_2，它被软件包 php-pdo-5.4.16-36.3.el7_2.x86_64 需要
--> 正在处理依赖关系 php-common(x86-64) = 5.4.16-36.3.el7_2，它被软件包 php-process-5.4.16-36.3.el7_2.x86_64 需要
--> 正在检查事务
---> 软件包 php-bcmath.x86_64.0.5.4.16-36.3.el7_2 将被 删除
---> 软件包 php-cli.x86_64.0.5.4.16-36.3.el7_2 将被 删除
---> 软件包 php-fpm.x86_64.0.5.4.16-36.3.el7_2 将被 删除
---> 软件包 php-gd.x86_64.0.5.4.16-36.3.el7_2 将被 删除
---> 软件包 php-ldap.x86_64.0.5.4.16-36.3.el7_2 将被 删除
---> 软件包 php-mbstring.x86_64.0.5.4.16-36.3.el7_2 将被 删除
---> 软件包 php-pdo.x86_64.0.5.4.16-36.3.el7_2 将被 删除
--> 正在处理依赖关系 php-pdo(x86-64) = 5.4.16-36.3.el7_2，它被软件包 php-mysql-5.4.16-36.3.el7_2.x86_64 需要
--> 正在处理依赖关系 php-pdo(x86-64) = 5.4.16-36.3.el7_2，它被软件包 php-odbc-5.4.16-36.3.el7_2.x86_64 需要
---> 软件包 php-pear.noarch.1.1.9.4-21.el7 将被 删除
---> 软件包 php-php-gettext.noarch.0.1.0.11-12.el7 将被 删除
---> 软件包 php-process.x86_64.0.5.4.16-36.3.el7_2 将被 删除
---> 软件包 php-tcpdf.noarch.0.6.2.11-1.el7 将被 删除
--> 正在处理依赖关系 php-tcpdf = 6.2.11-1.el7，它被软件包 php-tcpdf-dejavu-sans-fonts-6.2.11-1.el7.noarch 需要
---> 软件包 php-tidy.x86_64.0.5.4.16-5.el7 将被 删除
---> 软件包 php-xml.x86_64.0.5.4.16-36.3.el7_2 将被 删除
---> 软件包 php-xmlrpc.x86_64.0.5.4.16-36.3.el7_2 将被 删除
--> 正在检查事务
---> 软件包 php-mysql.x86_64.0.5.4.16-36.3.el7_2 将被 删除
---> 软件包 php-odbc.x86_64.0.5.4.16-36.3.el7_2 将被 删除
---> 软件包 php-tcpdf-dejavu-sans-fonts.noarch.0.6.2.11-1.el7 将被 删除
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package                       架构     版本                   源          大小
================================================================================
正在删除:
 php-common                    x86_64   5.4.16-36.3.el7_2      @updates   3.8 M
为依赖而移除:
 php-bcmath                    x86_64   5.4.16-36.3.el7_2      @updates    58 k
 php-cli                       x86_64   5.4.16-36.3.el7_2      @updates   8.8 M
 php-fpm                       x86_64   5.4.16-36.3.el7_2      @updates   4.5 M
 php-gd                        x86_64   5.4.16-36.3.el7_2      @updates   342 k
 php-ldap                      x86_64   5.4.16-36.3.el7_2      @updates    56 k
 php-mbstring                  x86_64   5.4.16-36.3.el7_2      @updates   1.3 M
 php-mysql                     x86_64   5.4.16-36.3.el7_2      @updates   232 k
 php-odbc                      x86_64   5.4.16-36.3.el7_2      @updates    97 k
 php-pdo                       x86_64   5.4.16-36.3.el7_2      @updates   192 k
 php-pear                      noarch   1:1.9.4-21.el7         @base      2.2 M
 php-php-gettext               noarch   1.0.11-12.el7          @epel       57 k
 php-process                   x86_64   5.4.16-36.3.el7_2      @updates    78 k
 php-tcpdf                     noarch   6.2.11-1.el7           @epel       11 M
 php-tcpdf-dejavu-sans-fonts   noarch   6.2.11-1.el7           @epel      1.5 M
 php-tidy                      x86_64   5.4.16-5.el7           @epel       53 k
 php-xml                       x86_64   5.4.16-36.3.el7_2      @updates   325 k
 php-xmlrpc                    x86_64   5.4.16-36.3.el7_2      @updates    82 k

事务概要
================================================================================
移除  1 软件包 (+17 依赖软件包)

安装大小：34 M
是否继续？[y/N]：y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在删除    : 1:php-pear-1.9.4-21.el7.noarch                             1/18 
  正在删除    : php-php-gettext-1.0.11-12.el7.noarch                       2/18 
  正在删除    : php-fpm-5.4.16-36.3.el7_2.x86_64                           3/18 
  正在删除    : php-mysql-5.4.16-36.3.el7_2.x86_64                         4/18 
  正在删除    : php-xmlrpc-5.4.16-36.3.el7_2.x86_64                        5/18 
  正在删除    : php-tcpdf-dejavu-sans-fonts-6.2.11-1.el7.noarch            6/18 
  正在删除    : php-tcpdf-6.2.11-1.el7.noarch                              7/18 
  正在删除    : php-cli-5.4.16-36.3.el7_2.x86_64                           8/18 
  正在删除    : php-bcmath-5.4.16-36.3.el7_2.x86_64                        9/18 
  正在删除    : php-gd-5.4.16-36.3.el7_2.x86_64                           10/18 
  正在删除    : php-mbstring-5.4.16-36.3.el7_2.x86_64                     11/18 
  正在删除    : php-process-5.4.16-36.3.el7_2.x86_64                      12/18 
  正在删除    : php-tidy-5.4.16-5.el7.x86_64                              13/18 
  正在删除    : php-xml-5.4.16-36.3.el7_2.x86_64                          14/18 
  正在删除    : php-odbc-5.4.16-36.3.el7_2.x86_64                         15/18 
  正在删除    : php-pdo-5.4.16-36.3.el7_2.x86_64                          16/18 
  正在删除    : php-ldap-5.4.16-36.3.el7_2.x86_64                         17/18 
  正在删除    : php-common-5.4.16-36.3.el7_2.x86_64                       18/18 
  验证中      : php-ldap-5.4.16-36.3.el7_2.x86_64                          1/18 
  验证中      : php-xml-5.4.16-36.3.el7_2.x86_64                           2/18 
  验证中      : php-pdo-5.4.16-36.3.el7_2.x86_64                           3/18 
  验证中      : php-tidy-5.4.16-5.el7.x86_64                               4/18 
  验证中      : php-common-5.4.16-36.3.el7_2.x86_64                        5/18 
  验证中      : php-mbstring-5.4.16-36.3.el7_2.x86_64                      6/18 
  验证中      : php-odbc-5.4.16-36.3.el7_2.x86_64                          7/18 
  验证中      : php-cli-5.4.16-36.3.el7_2.x86_64                           8/18 
  验证中      : php-process-5.4.16-36.3.el7_2.x86_64                       9/18 
  验证中      : php-tcpdf-dejavu-sans-fonts-6.2.11-1.el7.noarch           10/18 
  验证中      : php-gd-5.4.16-36.3.el7_2.x86_64                           11/18 
  验证中      : php-php-gettext-1.0.11-12.el7.noarch                      12/18 
  验证中      : php-bcmath-5.4.16-36.3.el7_2.x86_64                       13/18 
  验证中      : php-xmlrpc-5.4.16-36.3.el7_2.x86_64                       14/18 
  验证中      : php-mysql-5.4.16-36.3.el7_2.x86_64                        15/18 
  验证中      : php-fpm-5.4.16-36.3.el7_2.x86_64                          16/18 
  验证中      : php-tcpdf-6.2.11-1.el7.noarch                             17/18 
  验证中      : 1:php-pear-1.9.4-21.el7.noarch                            18/18 

删除:
  php-common.x86_64 0:5.4.16-36.3.el7_2                                         

作为依赖被删除:
  php-bcmath.x86_64 0:5.4.16-36.3.el7_2                                         
  php-cli.x86_64 0:5.4.16-36.3.el7_2                                            
  php-fpm.x86_64 0:5.4.16-36.3.el7_2                                            
  php-gd.x86_64 0:5.4.16-36.3.el7_2                                             
  php-ldap.x86_64 0:5.4.16-36.3.el7_2                                           
  php-mbstring.x86_64 0:5.4.16-36.3.el7_2                                       
  php-mysql.x86_64 0:5.4.16-36.3.el7_2                                          
  php-odbc.x86_64 0:5.4.16-36.3.el7_2                                           
  php-pdo.x86_64 0:5.4.16-36.3.el7_2                                            
  php-pear.noarch 1:1.9.4-21.el7                                                
  php-php-gettext.noarch 0:1.0.11-12.el7                                        
  php-process.x86_64 0:5.4.16-36.3.el7_2                                        
  php-tcpdf.noarch 0:6.2.11-1.el7                                               
  php-tcpdf-dejavu-sans-fonts.noarch 0:6.2.11-1.el7                             
  php-tidy.x86_64 0:5.4.16-5.el7                                                
  php-xml.x86_64 0:5.4.16-36.3.el7_2                                            
  php-xmlrpc.x86_64 0:5.4.16-36.3.el7_2                                         

完毕！
```

移除完毕后，查看是否存在残留：

```
[shebnowei@bogon ~]$ rpm -qa | grep php
[shebnowei@bogon ~]$ php -v
bash: php: 未找到命令...
```

### 卸载 MariaDB

```
sudo yum remove mariadb-server mariadb
```

执行结果：

```
[shebnowei@bogon ~]$ sudo yum remove mariadb-server mariadb
[sudo] password for shebnowei: 
已加载插件：fastestmirror, langpacks
正在解决依赖关系
--> 正在检查事务
---> 软件包 mariadb.x86_64.1.5.5.50-1.el7_2 将被 删除
---> 软件包 mariadb-server.x86_64.1.5.5.50-1.el7_2 将被 删除
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package              架构         版本                    源              大小
================================================================================
正在删除:
 mariadb              x86_64       1:5.5.50-1.el7_2        @updates        49 M
 mariadb-server       x86_64       1:5.5.50-1.el7_2        @updates        56 M

事务概要
================================================================================
移除  2 软件包

安装大小：104 M
是否继续？[y/N]：y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在删除    : 1:mariadb-server-5.5.50-1.el7_2.x86_64                      1/2 
警告：/var/log/mariadb/mariadb.log 已另存为 /var/log/mariadb/mariadb.log.rpmsave
  正在删除    : 1:mariadb-5.5.50-1.el7_2.x86_64                             2/2 
  验证中      : 1:mariadb-5.5.50-1.el7_2.x86_64                             1/2 
  验证中      : 1:mariadb-server-5.5.50-1.el7_2.x86_64                      2/2 

删除:
  mariadb.x86_64 1:5.5.50-1.el7_2     mariadb-server.x86_64 1:5.5.50-1.el7_2    

完毕！
```

检查相关服务和指令：

```
[shebnowei@bogon my.cnf.d]$ which mysql
/usr/bin/which: no mysql in (/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin:/home/shebnowei/.local/bin:/home/shebnowei/bin)
[shebnowei@bogon my.cnf.d]$ systemctl start mariadb.service
Failed to start mariadb.service: Unit mariadb.service failed to load: No such file or directory.
```

## 编译环境搭建

### 安装gcc gcc-c++

```
[shebnowei@bogon ~]$ rpm -q make
make-3.82-21.el7.x86_64
[shebnowei@bogon ~]$ rpm -q gcc
未安装软件包 gcc 
[shebnowei@bogon ~]$ rpm -q gcc-c++
未安装软件包 gcc-c++ 
```

编译时需要c、c++环境，所以在之前需要安装c、c++的编译器：gcc和gcc-c++。

```
sudo yum install gcc gcc-c++

[shebnowei@bogon ~]$ sudo yum install gcc gcc-c++
[sudo] password for shebnowei: 
已加载插件：fastestmirror, langpacks
base                                                     | 3.6 kB     00:00     
epel/x86_64/metalink                                     | 4.4 kB     00:00     
epel                                                     | 4.3 kB     00:00     
extras                                                   | 3.4 kB     00:00     
nginx                                                    | 2.9 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/2): epel/x86_64/updateinfo                              | 672 kB   00:00     
(2/2): epel/x86_64/primary_db                              | 4.3 MB   00:01     
Loading mirror speeds from cached hostfile
 * base: mirrors.btte.net
 * epel: mirror.premi.st
 * extras: mirrors.aliyun.com
 * updates: mirrors.zju.edu.cn
正在解决依赖关系
--> 正在检查事务
---> 软件包 gcc.x86_64.0.4.8.5-4.el7 将被 安装
--> 正在处理依赖关系 cpp = 4.8.5-4.el7，它被软件包 gcc-4.8.5-4.el7.x86_64 需要
--> 正在处理依赖关系 glibc-devel >= 2.2.90-12，它被软件包 gcc-4.8.5-4.el7.x86_64 需要
--> 正在处理依赖关系 libmpc.so.3()(64bit)，它被软件包 gcc-4.8.5-4.el7.x86_64 需要
---> 软件包 gcc-c++.x86_64.0.4.8.5-4.el7 将被 安装
--> 正在处理依赖关系 libstdc++-devel = 4.8.5-4.el7，它被软件包 gcc-c++-4.8.5-4.el7.x86_64 需要
--> 正在检查事务
---> 软件包 cpp.x86_64.0.4.8.5-4.el7 将被 安装
---> 软件包 glibc-devel.x86_64.0.2.17-106.el7_2.8 将被 安装
--> 正在处理依赖关系 glibc-headers = 2.17-106.el7_2.8，它被软件包 glibc-devel-2.17-106.el7_2.8.x86_64 需要
--> 正在处理依赖关系 glibc = 2.17-106.el7_2.8，它被软件包 glibc-devel-2.17-106.el7_2.8.x86_64 需要
--> 正在处理依赖关系 glibc-headers，它被软件包 glibc-devel-2.17-106.el7_2.8.x86_64 需要
---> 软件包 libmpc.x86_64.0.1.0.1-3.el7 将被 安装
---> 软件包 libstdc++-devel.x86_64.0.4.8.5-4.el7 将被 安装
--> 正在检查事务
---> 软件包 glibc.x86_64.0.2.17-105.el7 将被 升级
--> 正在处理依赖关系 glibc = 2.17-105.el7，它被软件包 glibc-common-2.17-105.el7.x86_64 需要
---> 软件包 glibc.x86_64.0.2.17-106.el7_2.8 将被 更新
---> 软件包 glibc-headers.x86_64.0.2.17-106.el7_2.8 将被 安装
--> 正在处理依赖关系 kernel-headers >= 2.2.1，它被软件包 glibc-headers-2.17-106.el7_2.8.x86_64 需要
--> 正在处理依赖关系 kernel-headers，它被软件包 glibc-headers-2.17-106.el7_2.8.x86_64 需要
--> 正在检查事务
---> 软件包 glibc-common.x86_64.0.2.17-105.el7 将被 升级
---> 软件包 glibc-common.x86_64.0.2.17-106.el7_2.8 将被 更新
---> 软件包 kernel-headers.x86_64.0.3.10.0-327.36.3.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

================================================================================
 Package              架构        版本                       源            大小
================================================================================
正在安装:
 gcc                  x86_64      4.8.5-4.el7                base          16 M
 gcc-c++              x86_64      4.8.5-4.el7                base         7.2 M
为依赖而安装:
 cpp                  x86_64      4.8.5-4.el7                base         5.9 M
 glibc-devel          x86_64      2.17-106.el7_2.8           updates      1.0 M
 glibc-headers        x86_64      2.17-106.el7_2.8           updates      663 k
 kernel-headers       x86_64      3.10.0-327.36.3.el7        updates      3.2 M
 libmpc               x86_64      1.0.1-3.el7                base          51 k
 libstdc++-devel      x86_64      4.8.5-4.el7                base         1.5 M
为依赖而更新:
 glibc                x86_64      2.17-106.el7_2.8           updates      3.6 M
 glibc-common         x86_64      2.17-106.el7_2.8           updates       11 M

事务概要
================================================================================
安装  2 软件包 (+6 依赖软件包)
升级           ( 2 依赖软件包)

总计：51 M
总下载量：36 M
Is this ok [y/d/N]: y
Downloading packages:
(1/8): cpp-4.8.5-4.el7.x86_64.rpm                          | 5.9 MB   00:22     
(2/8): glibc-devel-2.17-106.el7_2.8.x86_64.rpm             | 1.0 MB   00:01     
(3/8): glibc-headers-2.17-106.el7_2.8.x86_64.rpm           | 663 kB   00:01     
(4/8): kernel-headers-3.10.0-327.36.3.el7.x86_64.rpm       | 3.2 MB   00:03     
(5/8): gcc-c++-4.8.5-4.el7.x86_64.rpm                      | 7.2 MB   00:27     
(6/8): libmpc-1.0.1-3.el7.x86_64.rpm                       |  51 kB   00:00     
(7/8): libstdc++-devel-4.8.5-4.el7.x86_64.rpm              | 1.5 MB   00:07     
(8/8): gcc-4.8.5-4.el7.x86_64.rpm                          |  16 MB   01:08     
--------------------------------------------------------------------------------
总计                                               528 kB/s |  36 MB  01:09     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在更新    : glibc-2.17-106.el7_2.8.x86_64                              1/12 
  正在更新    : glibc-common-2.17-106.el7_2.8.x86_64                       2/12 
  正在安装    : libmpc-1.0.1-3.el7.x86_64                                  3/12 
  正在安装    : cpp-4.8.5-4.el7.x86_64                                     4/12 
  正在安装    : kernel-headers-3.10.0-327.36.3.el7.x86_64                  5/12 
  正在安装    : glibc-headers-2.17-106.el7_2.8.x86_64                      6/12 
  正在安装    : glibc-devel-2.17-106.el7_2.8.x86_64                        7/12 
  正在安装    : gcc-4.8.5-4.el7.x86_64                                     8/12 
  正在安装    : libstdc++-devel-4.8.5-4.el7.x86_64                         9/12 
  正在安装    : gcc-c++-4.8.5-4.el7.x86_64                                10/12 
  清理        : glibc-2.17-105.el7.x86_64                                 11/12 
  清理        : glibc-common-2.17-105.el7.x86_64                          12/12 
  验证中      : libstdc++-devel-4.8.5-4.el7.x86_64                         1/12 
  验证中      : gcc-4.8.5-4.el7.x86_64                                     2/12 
  验证中      : cpp-4.8.5-4.el7.x86_64                                     3/12 
  验证中      : glibc-devel-2.17-106.el7_2.8.x86_64                        4/12 
  验证中      : kernel-headers-3.10.0-327.36.3.el7.x86_64                  5/12 
  验证中      : glibc-headers-2.17-106.el7_2.8.x86_64                      6/12 
  验证中      : gcc-c++-4.8.5-4.el7.x86_64                                 7/12 
  验证中      : libmpc-1.0.1-3.el7.x86_64                                  8/12 
  验证中      : glibc-common-2.17-106.el7_2.8.x86_64                       9/12 
  验证中      : glibc-2.17-106.el7_2.8.x86_64                             10/12 
  验证中      : glibc-2.17-105.el7.x86_64                                 11/12 
  验证中      : glibc-common-2.17-105.el7.x86_64                          12/12 

已安装:
  gcc.x86_64 0:4.8.5-4.el7             gcc-c++.x86_64 0:4.8.5-4.el7            

作为依赖被安装:
  cpp.x86_64 0:4.8.5-4.el7                                                      
  glibc-devel.x86_64 0:2.17-106.el7_2.8                                         
  glibc-headers.x86_64 0:2.17-106.el7_2.8                                       
  kernel-headers.x86_64 0:3.10.0-327.36.3.el7                                   
  libmpc.x86_64 0:1.0.1-3.el7                                                   
  libstdc++-devel.x86_64 0:4.8.5-4.el7                                          

作为依赖被升级:
  glibc.x86_64 0:2.17-106.el7_2.8     glibc-common.x86_64 0:2.17-106.el7_2.8    

完毕！
```

### 关闭 SELinux

> SELinux
>
> SELinux(Security-Enhanced Linux) 是美国国家安全局（NSA）对于强制访问控制的实现，
> 是Linux历史上最杰出的新安全子系统。NSA是在Linux社区的帮助下开发了一种访问控制体系，在这种访问控制体系的限制下，进程只能访问那些在他的任务中所需要文件。
> SELinux 默认安装在 Fedora 和 Red Hat Enterprise Linux 上，也可以作为其他发行版上容易安装的包得到。
>
> 如果我们对于安全要求并不是特别高，使用防火墙就足够了，为了避免之后使用遇到问题，建议关闭SELinux。

通过更改`SELinux`的配置文件便可以实现关闭,将`SELINUX=enforcing`改为`SELINUX=disabled`。配置文件为`/etc/sysconfig/selinux`。
**之后需要重启系统声效**。

## 编译安装 Apache

首先下载需要的源码包，`Apache`官方地址为：`http://httpd.apache.org/`。

```
wget http://apache.fayea.com//httpd/httpd-2.4.23.tar.gz

依赖包 APR and APR-Util ,Perl-Compatible Regular Expressions Library (PCRE) ：
wget http://mirrors.cnnic.cn/apache//apr/apr-1.5.2.tar.gz
wget http://mirrors.cnnic.cn/apache//apr/apr-util-1.5.4.tar.gz
wget http://jaist.dl.sourceforge.net/project/pcre/pcre/8.39/pcre-8.39.tar.gz
```

上述三个依赖包中，`apr`和`apr-util`只需解压到`httpd`解压子目录下的`srclib/apr`和`srclib/apr-util`目录。而`pcre`需要提前编译安装。
PS. 这三个依赖包完全可以通过`yum`方式安装，这里按照官方的文档中的步骤进行。

### 编译安装 pcre

首先解压：

```
tar -zxvf pcre-8.39.tar.gz 
```

然后进入解压后的目录，配置安装目录，**注意这个目录在之后安装httpd时会用到，请记录**：

```
[shebnowei@bogon apache]$ cd pcre-8.39/
[shebnowei@bogon pcre-8.39]$ ./configure --prefix=/usr/local/pcre-8.39
```

最后执行编译命令`make`和安装命令（即把编译好的文件拷贝到指定目录）`make install`即可。
如果需要删除编译目录的临时编译文件(.o和可执行文件)，可以使用`make clean`命令清除。

检查安装是否成功，进入之前指定的安装目录查看：

```
[shebnowei@bogon pcre-8.39]$ cd /usr/local/pcre-8.39/
[shebnowei@bogon pcre-8.39]$ ls
bin  include  lib  share
[shebnowei@bogon pcre-8.39]$ cd bin
[shebnowei@bogon bin]$ ls -l
总用量 300
-rwxr-xr-x 1 root root   2373 11月 15 19:24 pcre-config
-rwxr-xr-x 1 root root 100285 11月 15 19:24 pcregrep
-rwxr-xr-x 1 root root 197860 11月 15 19:24 pcretest
```

### 编译 httpd

解压`httpd`:

```
tar -zxvf httpd-2.4.23.tar.gz 
```

之后将`apr`和`apr-util`解压到`httpd`下的`srclib`中的对应目录:

```
解压到指定目录：
tar -zxf apr-1.5.2.tar.gz -C httpd-2.4.23/srclib/ 
tar -zxf apr-util-1.5.4.tar.gz -C httpd-2.4.23/srclib/

改名，去掉版本号：
cd httpd-2.4.23/srclib/
mv apr-1.5.2 apr
mv apr-util-1.5.4 apr-util
```

查看目录下文件：

```
[shebnowei@bogon srclib]$ ls
apr  apr-util  Makefile.in
```

接下来进入`httpd`目录下，进行配置：

```
./configure --prefix=/usr/local/apache2/ --sysconfdir=/etc/httpd/ -with-pcre=/usr/local/pcre-8.39/ --with-included-apr --disable-userdir --enable-so --enable-deflate=shared --enable-expires=shared --enable-rewrite=shared --enable-static-support
```

**注意：如果在配置的时候出现因为zlib未找到所引发的错误，需要编译安装zlib或者直接通过yum安装zlib-devel**

然后依次执行`make`和`make install`(最后`make clean`清除)。
查看安装结果：

```
[shebnowei@bogon httpd-2.4.23]$ cd /usr/local/apache2/
[shebnowei@bogon apache2]$ ls
bin    cgi-bin  htdocs  include  logs  manual
build  error    icons   lib      man   modules
[shebnowei@bogon apache2]$ cd bin/
[shebnowei@bogon bin]$ ls -l
总用量 7544
-rwxr-xr-x 1 root      root       641999 11月 15 20:06 ab
-rwxr-xr-x 1 shebnowei shebnowei    3440 11月 15 20:01 apachectl
-rwxr-xr-x 1 root      root         7003 11月 15 20:06 apr-1-config
-rwxr-xr-x 1 root      root         6611 11月 15 20:06 apu-1-config
-rwxr-xr-x 1 shebnowei shebnowei   23516 11月 15 20:01 apxs
-rwxr-xr-x 1 root      root        13227 11月 15 20:06 checkgid
-rwxr-xr-x 1 shebnowei shebnowei    8925 11月 15 20:01 dbmmanage
-rw-rw-r-- 1 shebnowei shebnowei    1075 11月 15 20:01 envvars
-rw-rw-r-- 1 shebnowei shebnowei    1075 11月 15 20:01 envvars-std
-rwxr-xr-x 1 root      root       472833 11月 15 20:06 fcgistarter
-rwxr-xr-x 1 root      root       631313 11月 15 20:06 htcacheclean
-rwxr-xr-x 1 root      root       770305 11月 15 20:06 htdbm
-rwxr-xr-x 1 root      root       567189 11月 15 20:06 htdigest
-rwxr-xr-x 1 root      root       699770 11月 15 20:06 htpasswd
-rwxr-xr-x 1 root      root      2255006 11月 15 20:06 httpd
-rwxr-xr-x 1 root      root       599630 11月 15 20:06 httxt2dbm
-rwxr-xr-x 1 root      root       442929 11月 15 20:06 logresolve
-rwxr-xr-x 1 root      root       536863 11月 15 20:06 rotatelogs
```

配置文件所在目录：`/etc/httpd`。
相关控制命令：

```
sudo /usr/local/apache2/bin/apachectl -k start
sudo /usr/local/apache2/bin/apachectl -k stop
sudo /usr/local/apache2/bin/apachectl -k restart
```

## 编译安装 Nginx

下载并解压官网最新的源码包：

```
wget http://219.239.26.14/files/321500000944715A/nginx.org/download/nginx-1.10.2.tar.gz
tar -zxf nginx-1.10.2.tar.gz
```

进入解压后的`nginx-1.10.2`目录，可以通过命令`./configure --help`查看配置的帮助信息。
因为编译`nginx`中需要的`pcre`和`zlib`在之前编译`apache`时已经编译、安装过了，所以可以直接编译`nginx`:

```
./configure --prefix=/usr/local/nginx --conf-path=/etc/nginx/nginx.conf --with-pcre=/home/shebnowei/apache/pcre-8.39
```

**注意：--with-pcre= 应该配置的是pcre的源码目录(/home/shebnowei/apache/pcre-8.39)，而不是安装路径(/usr/local/pcre-8.39)！和apache是不同的。**

之后执行`make`和`make install`即可。
查看安装目录：

```
[shebnowei@localhost nginx-1.10.2]$ cd /usr/local/nginx/
[shebnowei@localhost nginx]$ ls
html  logs  sbin
[shebnowei@localhost nginx]$ cd sbin/
[shebnowei@localhost sbin]$ ls
nginx  nginx.old
[shebnowei@localhost sbin]$ sudo ./nginx
[shebnowei@localhost sbin]$ ls /etc/nginx
conf.d                  koi-win             scgi_params.default
fastcgi.conf            mime.types          uwsgi_params
fastcgi.conf.default    mime.types.default  uwsgi_params.default
fastcgi_params          nginx.conf          win-utf
fastcgi_params.default  nginx.conf.default
koi-utf                 scgi_params
```

启动文件为`/usr/local/nginx/sbin/nginx`，相关指令为：

```
启动nginx
sudo /usr/local/nginx/sbin/nginx
关闭nginx
sudo /usr/local/nginx/sbin/nginx -s stop
```

## 编译安装 MariaDB

### 准备工作

编译安装`MariaDB`需要`cmake`和`ncurses`，`openssl`可选：

```
sudo yum install ncurses-devel openssl-devel openssl cmake
```

为了方便管理`mariadb`,添加专门的用户组`mysql`和用户`mysql`：

```
groupadd mysql
useradd -g mysql mysql
```

如果已经存在，请忽略：

```
[shebnowei@localhost mariadb-10.1.19]$ id mysql
uid=27(mysql) gid=27(mysql) 组=27(mysql)
```

创建数据库数据存放目录：

```
mkdir -pv /data/mydata
```

### 编译安装

首先，我们下载最新的源码：

```
wget http://mirrors.tuna.tsinghua.edu.cn/mariadb//mariadb-10.1.19/source/mariadb-10.1.19.tar.gz
```

之后解压，进入解压目录，执行`cmake`命令：

```
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mydata -DSYSCONFDIR=/etc -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_READLINE=1 -DWITH_SSL=system -DWITH_ZLIB=system -DWITH_LIBWRAP=0 -DMYSQL_UNIX_ADDR=/tmp/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITHOUT_TOKUDB=1
```
**注意：对于gcc 4.8版本的编译器，是支持c++11的，但是对于mariadb的tokudb引擎貌似存在支持问题，据说需要更新到4.9版本以上（没有亲测），这里通过参数-DWITHOUT_TOKUDB=1来禁止安装tokudb引擎。**
**tokudb的相关错误信息为：**

```
CMake Error at storage/tokudb/PerconaFT/cmake_modules/TokuSetupCompiler.cmake:177(message):
  /usr/bin/c++ doesn't support -std=c++11 or -std=c++0x, you need one that does.
```

上述配置过程如果没有问题，就可以执行`make`和`make install`进行编译安装了。
编译过程比较漫长，大概需要40-50分钟（和机器配置相关）。

在源码目录的`support-files`文件夹中提供了相关的配置文件和服务文件：

```
[shebnowei@localhost support-files]$ ls
binary-configure        mariadb.pc.in              mysql-log-rotate
binary-configure.sh     mariadb.service.in         mysql-log-rotate.sh
build-tags              mariadb@.service.in        mysql.m4
ccfilter                my-huge.cnf                mysql-multi.server.sh
CMakeFiles              my-huge.cnf.sh             mysql.server
cmake_install.cmake     my-innodb-heavy-4G.cnf     mysql.server.sh
CMakeLists.txt          my-innodb-heavy-4G.cnf.sh  mysql.server-sys5.sh
compiler_warnings.supp  my-large.cnf               policy
CTestTestfile.cmake     my-large.cnf.sh            rpm
db.opt                  my-medium.cnf              use_galera_new_cluster.conf
dtrace                  my-medium.cnf.sh           wsrep.cnf
MacOSX                  my-small.cnf               wsrep.cnf.sh
magic                   my-small.cnf.sh            wsrep_notify
Makefile                mysqld_multi.server        wsrep_notify.sh
mariadb.pc              mysqld_multi.server.sh
```

然后拷贝配置文件为`/etc/my.cnf`：

```
[shebnowei@localhost support-files]$ sudo cp my-medium.cnf /etc/my.cnf
```

并修改里面的内容，添加`datadir = /data/mydata`到`[mysqld]`配置块下。

### 配置启动脚本

同样将`support-files`文件夹中的`mysql.server`拷贝为`/etc/init.d/mysqld`:

```
sudo cp mysql.server /etc/init.d/mysqld
```

最后我们需要创建初始化数据库：

```
cd /usr/local/mysql/scripts/
sudo ./mysql_install_db --user=mysql --basedir=/usr/local/mysql/ --datadir=/data/mydata/
```

初始化数据库之后，我们会发现`/data/mydata/`下会生成一些文件：

```
[shebnowei@localhost scripts]$ ls /data/mydata/
aria_log.00000001          localhost.localdomain.pid  mysql-bin.000004
aria_log_control           multi-master.info          mysql-bin.000005
ibdata1                    mysql                      mysql-bin.index
ib_logfile0                mysql-bin.000001           performance_schema
ib_logfile1                mysql-bin.000002
localhost.localdomain.err  mysql-bin.000003
```

最终我们可以通过`/etc/init.d/mysqld`脚本启动、终止和查看`mysql`数据库：

```
[shebnowei@localhost scripts]$ sudo /etc/init.d/mysqld status
 SUCCESS! MySQL running (89890)
[shebnowei@localhost scripts]$ sudo /etc/init.d/mysqld stop
Stopping mysqld (via systemctl):                           [  确定  ]
[shebnowei@localhost scripts]$ sudo /etc/init.d/mysqld start
Starting mysqld (via systemctl):                           [  确定  ]
```

## 编译安装 PHP

### 准备工作

编译`php`前，需要先编译安装下`libxml2`、`libmcrypt`和`gd2`（`gd2`需要`freetype`、`libpng`和`jpegsrc`）等软件包。

- libxml2编译安装

```
wget http://124.202.164.4/files/121200000913906C/fossies.org/linux/www/libxml2-2.9.4.tar.gz
tar -zxf libxml2-2.9.4.tar.gz
cd libxml2-2.9.4/
./configure --prefix=/usr/local/libxml2/
make 
sudo make install
```

**注意：在编译过程中如果提示找不到\<Python.h\>，需要安装一下python-devel，执行命令yum install python-devel。**
安装完成后，会在`/usr/local/libxml2/`下生成如下目录：

```
[shebnowei@localhost libxml2-2.9.4]$ ls /usr/local/libxml2/
bin  include  lib  share
```

- libmcrypt编译安装

```
wget http://124.202.164.16/files/3174000001A3B7A7/admin.ooopic.com/soft/linux/libmcrypt-2.5.8.tar.gz
tar -zxf libmcrypt-2.5.8.tar.gz 
cd libmcrypt-2.5.8/
./configure --prefix=/usr/local/libmcrypt/
make 
sudo make install

cd libltdl/
./configure --enable-ltdl-install
make
sudo make install
```

安装完成后，会在`/usr/local/libmcrypt/`下生成如下目录：

```
[shebnowei@localhost libltdl]$ ls /usr/local/libmcrypt/
bin  include  lib  man  share
```

- freetype编译安装

```
wget http://jaist.dl.sourceforge.net/project/freetype/freetype2/2.7/freetype-2.7.tar.gz
tar -zxf freetype-2.7.tar.gz
cd freetype-2.7/
./configure --prefix=/usr/local/freetype/
make
sudo make install
```

安装完成后，会在`/usr/local/freetype/`下生成如下目录：

```
[shebnowei@localhost freetype-2.7]$ ls /usr/local/freetype/
bin  include  lib  share
```

- libpng编译安装

```
wget http://219.239.26.9/files/207200000954C14F/jaist.dl.sourceforge.net/project/libpng/libpng16/1.6.26/libpng-1.6.26.tar.gz
tar -zxf libpng-1.6.26.tar.gz
cd libpng-1.6.26/
./configure --prefix=/usr/local/libpng/
make
sudo make install
```

安装完成后，会在`/usr/local/libpng/`下生成如下目录：

```
[shebnowei@localhost libpng-1.6.26]$ ls /usr/local/libpng/
bin  include  lib  share
```

- jpegsrc编译安装

```
wget http://219.238.7.66/files/4029000007B700B5/www.ijg.org/files/jpegsrc.v9b.tar.gz
tar -zxf jpegsrc.v9b.tar.gz 
cd jpeg-9b/
./configure --prefix=/usr/local/jpeg9/ --enable-shared --enable-static
make
sudo make install
```

安装完成后，会在`/usr/local/jpeg9/`下生成如下目录：

```
[shebnowei@localhost jpeg-9b]$ ls /usr/local/jpeg9/
bin  include  lib  share
```

- gd编译安装

```
wget https://bitbucket.org/libgd/gd-libgd/downloads/libgd-2.1.1.tar.gz
tar -zxf libgd-2.1.1.tar.gz
cd libgd-2.1.1/
./configure --prefix=/usr/local/gd2/ --with-jpeg=/usr/local/jpeg9/ --with-freetype=/usr/local/freetype/ --with-png=/usr/local/libpng/
make
sudo make install
```

安装完成后，会在`/usr/local/gd2/`下生成如下目录：

```
[shebnowei@localhost libgd-2.1.1]$ ls /usr/local/gd2/
bin  include  lib
```

### 编译安装

推荐编译安装前先看看官方文档：[PHP 手册](http://php.net/manual/zh/install.php "跳转")。

从官方镜像下载`php`的源码包后解压：

```
wget http://219.239.26.11/files/22540000096535CD/ca1.php.net/distributions/php-5.6.28.tar.gz
tar -zxf php-5.6.28.tar.gz 
```

之后进入解压目录，配置`php`:

```
./configure --prefix=/usr/local/php/ --with-config-file-path=/usr/local/php/etc/ --with-apxs2=/usr/local/apache2/bin/apxs --with-mysql=/usr/local/mysql/ --with-mysqli=/usr/local/mysql/bin/mysql_config --with-libxml-dir=/usr/local/libxml2/ --with-mcrypt=/usr/local/libmcrypt/ --with-jpeg-dir=/usr/local/jpeg9/ --with-png-dir=/usr/local/libpng/ --with-freetype-dir=/usr/local/freetype/ --with-gd=/usr/local/gd2/ --enable-soap --enable-mbstring=all --enable-sockets --enable-fpm --with-xpm-dir=/usr/lib64/（这个参数查看后边的错误解决）
```

**如果编译时出现如下错误：**

**注意：/home/shebnowei/php/php-5.6.28/ext/gd/gd.c:57:22: 致命错误：X11/xpm.h：没有那个文件或目录 # include <X11/xpm.h>**

**解决：**

- **首先yum install libXpm-devel安装缺少的库。**
- **之后重新编译安装一下gd库。**
- **最后在php的配置阶段添加参数--with-xpm-dir=/usr/lib64/（libXpm安装路径）。**
**rpm -ql libXpm可以查看libXpm安装路径。**


配置成功后，即可进行编译安装，即执行`make`和`sudo make install`。安装完成后，可以查看安装目录：

```
[shebnowei@localhost php-5.6.28]$ ls /usr/local/php/
bin  etc  include  lib  php  sbin  var
[shebnowei@localhost php-5.6.28]$ ls /usr/local/php/etc
pear.conf  php-fpm.conf.default
```

需要将源目录中的配置文件拷贝到配置目录，同时将`php-fpm`的配置文件重命名下：

```
cp php.ini-development /usr/local/php/etc/php.ini
cd /usr/local/php/etc/
sudo cp php-fpm.conf.default php-fpm.conf
```

查看配置文件目录`/usr/local/php/etc/`：

```
[shebnowei@localhost etc]$ ls
pear.conf  php-fpm.conf  php-fpm.conf.default  php.ini
```

启动`php-fpm`:

```
sudo /usr/local/php/sbin/php-fpm
```

## 整合

为了让`Apache`把`.php`或`.phtml`后缀名解析为`PHP`。需要在配置文件中进行设置：

在`Apache`的配置文件`/etc/httpd/httpd.conf`中找到`AddType application/x-gzip .gz .tgz`指令，
下边添加一行指令：

```
AddType application/x-httpd-php .php .phtml
```

之后重启`Apache`服务即可。

关于整合`nginx`和`php`，可以参考[centos 7下配置Apache+Nginx+PHP+MariaDB环境--安装配置篇](https://shenbowei.github.io/2016/10/30/lanmp.html#php-with-nginx "跳转")。

> ## 参考文献
>
>[Apache HTTP Server Version 2.4](http://httpd.apache.org/docs/2.4/install.html "跳转")
>
>[CentOS7 下编译安装Apache http server](http://www.jianshu.com/p/949350cae1c8 "跳转")
>
>[Centos7 从零编译Nginx+PHP+MySql](http://www.cnblogs.com/project/p/5095146.html "跳转")
>
>[编译安装MariaDB-10.0.21](http://www.cnblogs.com/daixiang/p/5431639.html "跳转")
>
>[PHP 手册](http://php.net/manual/zh/install.php "跳转")
>
>[Centos7+Apache2.4+php5.6+mysql5.5搭建Lamp环境](http://www.cnblogs.com/AIThink/p/5540866.html "跳转")
>
>[error: X11/xpm.h: No such file or directory 解决](http://blog.csdn.net/weixiaodahuaidan/article/details/48682901 "跳转")







