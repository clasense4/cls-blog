+++
title = "Install Nginx Stable / Mainline Centos 7"
author = "Fajri Abdillah"
share = true
tags = ["snippet","centos","nginx"]
draft = false
menu = ""
image = "images/default.jpg"
slug = "install-nginx-stable-mainline-on-centos7"
date = "2017-02-10T08:55:00+07:00"
comments = true
+++

<!--more-->

### Nginx Stable

~~~
yum install http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm -y
yum update -y
yum install nginx -y
~~~

### Nginx Mainline

~~~
sudo echo "[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/7/\$basearch/
gpgcheck=0
enabled=1" > /etc/yum.repos.d/nginx.repo
yum update -y
yum install nginx -y
~~~

### Check nginx
~~~
[root@36d00d8fbd5d yum.repos.d]# nginx -v
nginx version: nginx/1.10.2
[root@36d00d8fbd5d yum.repos.d]# nginx -v
nginx version: nginx/1.11.9
~~~

>>> If you encounter, "Not using downloaded repomd.xml because it is older than what we have"
>>> try "yum clean all"

Source :  

1. [http://nginx.org/en/linux_packages.html#mainline](http://nginx.org/en/linux_packages.html#mainline)

---

>>> Server is powered by [Digital Ocean](https://m.do.co/c/6b1c3b315e1e), use my referral link to get $10 bonus.
