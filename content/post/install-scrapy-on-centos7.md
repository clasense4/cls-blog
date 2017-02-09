+++
title = "Install Scrapy on Centos 7"
author = "Fajri Abdillah"
share = true
tags = ["snippet","centos"]
draft = false
menu = ""
image = "images/default.jpg"
slug = "install-scrapy-on-centos7"
date = "2017-02-09T09:41:00+07:00"
comments = true
+++

Tested in $5 Digital Ocean droplet.

<!--more-->


### 1. Set Swap

```
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo "/swapfile swap swap sw 0 0" >> /etc/fstab
echo "vm.swappiness = 10" >>/etc/sysctl.conf
echo "vm.vfs_cache_pressure = 50" >> /etc/sysctl.conf
```

### 2. Install Dependencies & Scrapy

```
sudo rpm -Uvh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-9.noarch.rpm
yum update -y
yum install python-pip -y
yum install python-devel -y
yum install gcc gcc-devel -y
yum install libxml2 libxml2-devel -y
yum install libxslt libxslt-devel -y
yum install openssl openssl-devel -y
yum install libffi libffi-devel -y
CFLAGS="-O0" pip install lxml
pip install scrapy
```

### 3. Check Scrapy

```
[root@bc04fe1416ed /]# scrapy -v
Scrapy 1.3.1 - no active project

Usage:
  scrapy <command> [options] [args]

Available commands:
  bench         Run quick benchmark test
  commands
  fetch         Fetch a URL using the Scrapy downloader
  genspider     Generate new spider using pre-defined templates
  runspider     Run a self-contained spider (without creating a project)
  settings      Get settings values
  shell         Interactive scraping console
  startproject  Create new project
  version       Print Scrapy version
  view          Open URL in browser, as seen by Scrapy

  [ more ]      More commands available when run from project directory

Use "scrapy <command> -h" to see more info about a command
```

Server is powered by [Digital Ocean](https://m.do.co/c/6b1c3b315e1e), use my referral link to get $10 bonus.
