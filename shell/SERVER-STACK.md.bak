# I. Server
## Installing Apache

```shell
sudo yum install httpd
sudo systemctl start httpd && sudo systemctl enable httpd
sudo systemctl restart httpd
```
## Installing NGINX

```shell
sudo yum install nginx
sudo systemctl start nginx && sudo systemctl enable nginx
sudo systemctl restart nginx
```

# II. Platform
## Installing PHP 8.0

```shell
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module enable php:remi-8.0 -y
sudo yum install -y php php-fpm php-cli php-common php-bz2 php-ctype php-curl php-date php-exif php-fileinfo php-filter php-gd php-gettext php-hash php-iconv php-json php-libxml php-mbstring php-mcrypt php-mysqli php-openssl php-pcre php-phar php-pear php-readline php-session php-sockets php-standard php-tokenizer php-zlib -y

```

### Installing Composer (PHP)
```shell
sudo yum install -y unzip php-cli
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
HASH="$(wget -q -O - https://composer.github.io/installer.sig)"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

## Installing Node.js

```shell
sudo curl -sL https://rpm.nodesource.com/setup_14.x | sudo bash -
sudo yum install nodejs -y
```

# III. DB
## Installing MariaDB

```shell
sudo yum install mariadb-server
sudo systemctl start mariadb && sudo systemctl enable mariadb
sudo mysql_secure_installation
```

### Creating db & users

```shell
mysql -u root -p
> CREATE DATABASE yourDB;
> SHOW DATABASES;
> CREATE USER 'user1'@localhost IDENTIFIED BY 'password1';
> GRANT ALL PRIVILEGES ON yourDB.* TO 'user1'@localhost IDENTIFIED BY 'password1';
> FLUSH PRIVILEGES;
```

If you would like to enter a db to add tables,

```shell
USE databasename;
```


## Installing MongoDB
Please ensure nano is installed before starting this tutorial.

```shell
sudo nano /etc/yum.repos.d/mongodb-org.repo
```

```
[mongodb-org-4.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.0.asc
```

```shell
sudo yum install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
sudo nano /etc/mongod.conf
```

```conf
security:
        authorization: enabled
```

```shell
sudo systemctl restart mongod
mongo
use admin
```

```
db.createUser(
        {
        user: "mongoAdmin", 
        pwd: "changeMe", 
        roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
        }
)
```

```shell
quit()
```
