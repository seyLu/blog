---
title: LAMP Setup (Ubuntu 22.04)
description: "How to setup LAMP stack on Ubuntu 22.04"
date: "2023-09-29"
categories: ["Notes"]
tags: ["ubuntu", "lamp", "linux", "apache", "mongodb", "php"]
image: Oreimo_Leon_Atkinson_Core_PHP_Programming.png
---

## Configure `/var/www/custom_domain`

### Directory ownership

```bash
sudo chown $USER:www-data /var/www/custom_domain
sudo chmod g+s /var/www/custom_domain
sudo chmod o-rwx /var/www/custom_domain
```

### Apacheconf

```bash
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/custom_domain.conf
sudo nano /etc/apache2/sites-available/custom_domain.conf
```

<br>

```apacheconf
<VirtualHost *:80>
    ServerName custom_domain.test
    ServerAlias *.custom_domain.test

    DocumentRoot /var/www/custom_domain/public
    FallbackResource /index.php

    <Directory "/var/www/custom_domain/public">
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

### Activate virtual host

```bash
sudo a2ensite custom_domain.conf
sudo service apache2 reload
```

## Switch `php` version used by `apache`

### Install different php version

```bash
sudo apt install php*.* php*.*-mysql php*.*-mbstring libapache2-mod-php*.*
```

### Enable php version

```bash
sudo a2enmod php*.*
```

### Disable php version

```bash
sudo a2dismod php*.*
```
