+++
title = "Quick Setup Adminer using PHP built in webserver"
author = "Fajri Abdillah"
share = true
tags = ["snippet","php","database"]
draft = false
menu = ""
image = "images/default.jpg"
slug = "quick-setup-adminer-php"
date = "2017-02-09T06:25:00+07:00"
comments = true
+++


**Why is Adminer better than phpMyAdmin?**

Replace phpMyAdmin with Adminer and you will get a tidier user interface, better support for MySQL features, higher performance and more security.

<!--more-->

**Requirement :**

1. PHP [Installation](/post/install-php71-on-centos7)
2. [Adminer](https://www.adminer.org/#download)

Adminer version when this article posted is `4.2.5`.

Here is the snippet :

```
php -S 127.0.0.1:8082 -t adminer.php
```

And here is some output :

```
~/Projects❯ php -S 127.0.0.1:8082 adminer.php
PHP 7.0.13 Development Server started at Thu Feb  9 13:22:31 2017
Listening on http://127.0.0.1:8082
Document root is /Users/fajriabdillah/Projects
Press Ctrl-C to quit.
```

---

>>> Server is powered by [Digital Ocean](https://m.do.co/c/6b1c3b315e1e), use my referral link to get $10 bonus.
