+++
title = "Install MariaDB on Centos 7"
author = "Fajri Abdillah"
share = true
tags = ["snippet","centos","mysql","mariadb"]
draft = false
menu = ""
image = "images/default.jpg"
slug = "install-mariadb-on-centos7"
date = "2017-02-08T02:55:00+07:00"
comments = true
+++

<!--more-->

Here is the snippet :

```
echo "# MariaDB 10.1 CentOS repository list - created 2017-02-07 08:50 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1" > /etc/yum.repos.d/MariaDB.repo

sudo yum install MariaDB-server MariaDB-client -y

sudo systemctl start mariadb
/usr/bin/mysql_secure_installation
# and follow the instruction
```

Source :  

1. [https://mariadb.com/kb/en/mariadb/yum/](https://mariadb.com/kb/en/mariadb/yum/)

---

>>> Server is powered by [Digital Ocean](https://m.do.co/c/6b1c3b315e1e), use my referral link to get $10 bonus.
