---
layout: post
title: PHP编译安装openssl拓展
category: 技术
author: 沈伯伟
tags: PHP
---

之前在编译安装`PHP`的时候([centos下配置Apache+Nginx+PHP+MariaDB环境--编译篇](https://shenbowei.github.io/2016/11/10/lanmp-compiler.html "跳转"))没有添加`openssl`拓展。
在此，单独编译`openssl`拓展，并添加到`PHP`。

## 编译安装openssl拓展

首先进入`PHP`源码目录下的拓展目录中的`openssl`目录，拷贝`config0.m4`为`config.m4`：

```
[shebnowei@localhost php]$ cd php-5.6.28/ext/openssl/
[shebnowei@localhost openssl]$ ls
config0.m4  CREDITS    openssl.dsp  php_openssl.h  tests
config.w32  openssl.c  openssl.mak  README         xp_ssl.c
[shebnowei@localhost openssl]$ cp config0.m4 config.m4
[shebnowei@localhost openssl]$ ls
config0.m4  config.w32  openssl.c    openssl.mak    README  xp_ssl.c
config.m4   CREDITS     openssl.dsp  php_openssl.h  tests
```

之后执行`phpize`命令（`phpize`是用来扩展`PHP`扩展模块的，位于`PHP`安装目录的`bin`下），`phpize`执行成功后会在这个目录生成configure脚本，然后进行配置、编译安装：

```
phpize
./configure --with-openssl --with-php-config=/usr/local/php/bin/php-config
make
sudo make install
```

## 配置PHP.ini

打开`PHP`的配置文件`php.ini`:

```
sudo vim /usr/local/php/etc/php.ini 
```

在`Dynamic Extensions`下,添加一行：

```
extension = openssl.so
```

检查安装配置是否成功：

```
[shebnowei@localhost openssl]$ php -i | grep openssl
openssl
Openssl default config => /etc/pki/tls/openssl.cnf
openssl.cafile => no value => no value
openssl.capath => no value => no value
PWD => /home/shebnowei/php/php-5.6.28/ext/openssl
_SERVER["PWD"] => /home/shebnowei/php/php-5.6.28/ext/openssl
```

启动`nginx`或者`apache`，查看`phpinfo()`:

![phpinfo()](/public/img/php/php_openssl.png "phpinfo()")

## 附录：安装autoconf

```
[shebnowei@localhost openssl]$ phpize
Configuring for:
PHP Api Version:         20131106
Zend Module Api No:      20131226
Zend Extension Api No:   220131226
Cannot find autoconf. Please check your autoconf installation and the
$PHP_AUTOCONF environment variable. Then, rerun this script.
```

如果在执行`phpize`命令的时候提示找不到`autoconf`,还需要安装`autoconf`,因为`autoconf`还需要`m4`的支持，所以需要先安装编译`m4`:

- `m4`编译安装：

```
wget http://ftp.gnu.org/gnu/m4/m4-1.4.17.tar.gz
tar -zxf m4-1.4.17.tar.gz
cd m4-1.4.17/
./configure
make 
sudo make install

[shebnowei@localhost php]$ m4 --version
m4 (GNU M4) 1.4.17
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by Rene' Seindal.
```

- `autoconf`编译安装：

```
wget http://219.238.7.71/files/2049000005B39A16/ftpmirror.gnu.org/autoconf/autoconf-2.69.tar.gz
tar -zxf autoconf-2.69.tar.gz
cd autoconf-2.69/
./configure
make 
sudo make install

[shebnowei@localhost php]$ autoconf --version
autoconf (GNU Autoconf) 2.69
Copyright (C) 2012 Free Software Foundation, Inc.
License GPLv3+/Autoconf: GNU GPL version 3 or later
<http://gnu.org/licenses/gpl.html>, <http://gnu.org/licenses/exceptions.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by David J. MacKenzie and Akim Demaille.
```

> ## 参考文献
>
> [php编译openssl模块的步骤](http://blog.csdn.net/wgw335363240/article/details/41984267 "跳转")
>
> [centos下php添加openssl扩展](http://www.51ou.com/browse/linuxwt/59080.html "跳转")
>
> [Cannot find autoconf. Please check your autoconf installation](http://helpinlinux.com/cannot-find-autoconf-please-check-your-autoconf-installation/ "跳转")
>
>[安装php openssl扩展](https://my.oschina.net/u/195896/blog/332970 "跳转")








