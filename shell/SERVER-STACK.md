# I. Server
## Setup
First, open the ports needed. If using HTTPS, open 443. If not, only open 80.

```shell
sudo firewall-cmd --zone=public --add-service=http --permanent
sudo firewall-cmd --zone=public --add-service=https --permanent
sudo firewall-cmd --reload
```

## Installing Apache

```shell
sudo yum install httpd -y
sudo systemctl start httpd && sudo systemctl enable httpd
sudo systemctl restart httpd
```

Next, modify config:

```shell
sudo nano /etc/httpd/conf/httpd.conf
```

Optionally, setup Virtual Hosts:

```shell
sudo mkdir /var/www/blazed.space/
sudo nano /etc/httpd/conf.d/blazed.space.conf
```

```
<VirtualHost *:80>
	    ServerAdmin admin@blazed.space
	    ServerName blazed.space
	    ServerAlias www.blazed.space
		DocumentRoot /var/www/blazed.space
		SetEnv FUEL_ENV production
		<Directory "/var/www/blazed.space">
           Options FollowSymlinks
           AllowOverride all
           Require all granted
        </Directory>
</VirtualHost>
<IfModule mod_ssl.c>
	<VirtualHost *:443>
	    ServerAdmin admin@blazed.space
	    ServerName blazed.space
	    ServerAlias www.blazed.space
		DocumentRoot /var/www/blazed.space
		Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains;"
		SetEnv FUEL_ENV production
	<Directory "/var/www/blazed.space">
           Options FollowSymlinks
           AllowOverride all
           Require all granted
        </Directory>
	   SSLEngine on
	   SSLCertificateFile /etc/ssl/blazed.space/chain.pem
	   SSLCertificateKeyFile /etc/ssl/blazed.space/key.pem

	   SSLProtocol -all +TLSv1.3 +TLSv1.2
	   SSLOpenSSLConfCmd Curves X25519:secp521r1:secp384r1:prime256v1
	   SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
	</VirtualHost>
</IfModule>
```

Included in this repo are example httpd.conf files. 

Optionally, install HTTPS:

```shell
sudo yum install mod_ssl -y
sudo nano /etc/httpd/conf.d/ssl.conf
```

And install the cert bundle to the server:

```shell
sudo mkdir /etc/ssl/blazed.space
sudo nano /etc/ssl/blazed.space/cert.pem
sudo nano /etc/ssl/blazed.space/key.pem
```

## Configure Mod_wsgi and Python
```shell
sudo yum install python3-mod_wsgi -y
sudo nano /etc/httpd/conf.d/python3_wsgi.conf
```

Instantiate Mod_wsgi via alias directiories mapped to scripts:

```
WSGIScriptAlias /test_wsgi /var/www/html/test_wsgi.py
```

Now, create the python file:

```python
def application(environ, start_response):
	status = '200 OK'
	html   = '<!DOCTYPE HTML><html lang="en" dir="ltr"><body><p>Hello world!</p></body></html>'.encode("utf-8")
	response_header = [('Content-type', 'text/html')]
	start_response(status, response_header)
	return [html]
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
sudo yum install -y unzip php-cli wget
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
HASH="$(wget -q -O - https://composer.github.io/installer.sig)"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

## Installing Node.js

```shell
sudo curl -sL https://rpm.nodesource.com/setup_16.x | sudo bash -
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
