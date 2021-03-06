+++
title = "Install php 7.1 on Centos 7"
author = "Fajri Abdillah"
share = true
tags = ["snippet","centos","php"]
draft = false
menu = ""
image = "images/default.jpg"
slug = "install-php71-on-centos7"
date = "2017-02-04T04:55:00+07:00"
comments = true
+++

Recently, I want to move my old portfolio site, into a new server. Basically I want to update from centos 6 to centos 7. So, I'm start digging some information about installing php 7.1 in my centos 7 server.

<!--more-->

Here is the snippet.

```
cd ~
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
wget http://rpms.remirepo.net/enterprise/remi-release-7.rpm
rpm -Uvh epel-release-latest-7.noarch.rpm
rpm -Uvh remi-release-7.rpm

yum install yum-utils
yum-config-manager --enable remi-php71

yum install php php-devel php-fpm php-gd php-mbstring php-pdo php-pecl-swoole php-pgsql php-mcrypt
```

So, after installation is finished, we can try this :

```
[fajri@centos-512mb-sgp1 ~]# php -v
PHP 7.1.1 (cli) (built: Jan 18 2017 11:37:34) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.1.0, Copyright (c) 1998-2017 Zend Technologies
```

To make sure our php-fpm running on port *9000*:

```
[root@centos-512mb-sgp1 ~]# service php-fpm status
Redirecting to /bin/systemctl status  php-fpm.service
● php-fpm.service - The PHP FastCGI Process Manager
   Loaded: loaded (/usr/lib/systemd/system/php-fpm.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2017-02-08 01:23:18 UTC; 6 days ago
 Main PID: 2614 (php-fpm)
   Status: "Processes active: 0, idle: 21, Requests: 923, slow: 0, Traffic: 0req/sec"
   Memory: 280.8M
   CGroup: /system.slice/php-fpm.service
           ├─ 2614 php-fpm: master process (/etc/php-fpm.conf)
           ├─ 3943 php-fpm: pool www
           ├─ 4651 php-fpm: pool www
           ├─ 4894 php-fpm: pool www
           ├─......
           └─26856 php-fpm: pool www

Feb 08 01:23:18 centos-512mb-sgp1 systemd[1]: Starting The PHP FastCGI Process Manager...
Feb 08 01:23:18 centos-512mb-sgp1 systemd[1]: Started The PHP FastCGI Process Manager.
Feb 10 07:49:26 centos-512mb-sgp1 systemd[1]: Reloaded The PHP FastCGI Process Manager.
[root@centos-512mb-sgp1 ~]# netstat -ntpl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      736/nginx: master p
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      736/nginx: master p
tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      2614/php-fpm: maste
....
```

>>> Using port 9000 is the easiest way to setup php-fpm. You can use unix socket too. But you need to careful about permission.

On the next blog post, we will cover integration with php-fpm + nginx and some php frameworks.

Source :  

1. [https://blog.remirepo.net/post/2016/12/05/Install-PHP-7.1-on-CentOS-RHEL-or-Fedora](https://blog.remirepo.net/post/2016/12/05/Install-PHP-7.1-on-CentOS-RHEL-or-Fedora)

---

>>> Server is powered by [Digital Ocean](https://m.do.co/c/6b1c3b315e1e), use my referral link to get $10 bonus.
