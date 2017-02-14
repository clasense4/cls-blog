+++
title = "Install latest laravel using nginx, php-fpm on centos 7"
author = "Fajri Abdillah"
share = true
tags = ["php","laravel","nginx","centos"]
draft = false
menu = ""
image = "images/default.jpg"
slug = "install-latest-laravel-nginx-php-fpm-on-centos-7"
date = "2017-02-10T09:27:24+07:00"
comments = true
+++

Laravel 5.4 has been released. And here is step by step how to installing it on centos 7 using nginx and php-fpm.

<!--more-->

### Requirement :

1. [Install Nginx](/post/install-nginx-stable-mainline-on-centos7)
2. [Install PHP-FPM](/post/install-php71-on-centos7)

### 1. Composer

~~~
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '55d6ead61b29c7bdee5cccfb50076874187bd9f21f65d8991d46ec5cc90518f447387fb9f76ebae1fbbacf329e583e30') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
mv composer.phar /usr/local/bin/composer
~~~

### 2. Install Laravel and try using builtin webserver

~~~
composer global require "hirak/prestissimo:^0.3"
composer create-project --prefer-dist laravel/laravel blog
mv blog /var/www/blog
chown -R nginx:nginx /var/www/blog
cd /var/www/blog
php artisan serve --host=0.0.0.0
~~~

>>> Check your server, example http://128.199.127.234:8000/

### 3. Nginx conf

~~~
sudo echo "server {
    listen     8000;
    root /var/www/blog/public;
    index index.php;

    access_log     /var/log/nginx/nginx.vhost.access.log;
    error_log      /var/log/nginx/nginx.vhost.error.log;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param  SCRIPT_FILENAME   \$document_root\$fastcgi_script_name;
        fastcgi_param PATH_INFO \$fastcgi_script_name;
        include        fastcgi_params;
    }
}" > /etc/nginx/conf.d/laravel-sample.conf
nginx -t
service nginx restart
~~~

### 4. Make sure our laravel app running on exact port

~~~
[root@centos-512mb-sgp1-01-1483019484734-scrapy-512mb-sgp1-01 conf.d]# netstat -ntpl
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      23176/nginx: master
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      23176/nginx: master
tcp        0      0 0.0.0.0:8000            0.0.0.0:*               LISTEN      23176/nginx: master
tcp        0      0 127.0.0.1:9000          0.0.0.0:*               LISTEN      2614/php-fpm: maste
~~~

>>> Check your server again, example http://128.199.127.234:8000/

---
>>> Server is powered by [Digital Ocean](https://m.do.co/c/6b1c3b315e1e), use my referral link to get $10 bonus.
