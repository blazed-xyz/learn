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

## Apache httpd - Enable Userdir
```sh
sudo nano /etc/httpd/conf.d/userdir.conf
```

```
# line 17 : comment out
#UserDir disabled

# line 24 : uncomment
UserDir public_html

# line 32, 33
<Directory "/home/*/public_html">
    AllowOverride All     # change if you need
    Options None          # change if you need
    Require method GET POST OPTIONS
</Directory>
```

```sh
cd ~
sudo mkdir public_html
sudo chmod 755 -R ~/public_html
sudo chown $USER:$USER ~/public_html
sudo systemctl reload httpd
sudo nano ~/public_html/index.html
```

```html
<html>
	<body>
		<div style="width: 100%; font-size: 40px; font-weight: bold; text-align: center;">
			UserDir Test Page
		</div>
	</body>
</html>
```

* If SELinux is enabled, change policy like follows:
```sh
sudo setsebool -P httpd_enable_homedirs on
sudo restorecon -R /home
```

## Installing NGINX

```shell
sudo yum install nginx
sudo systemctl start nginx && sudo systemctl enable nginx
sudo systemctl restart nginx
```

### NGINX Content
* **/usr/share/nginx/html**

### NGINX Config
* **/etc/nginx/nginx.conf**
* **/etc/nginx/conf.d/**

### Setting up Server Blocks on NGINX
First, create the directory for your virtual domain:
```shell
sudo mkdir -p /var/www/your_domain/html
```

Next, assign ownership of the directory with the $USER environment variable, which should reference your current system user:
```shell
sudo chown -R $USER:$USER /var/www/your_domain/html
```

OR:

```shell
sudo chown -R nginx:nginx /var/www/your_domain/html
```

Now you can create a simple HTML page for your content:
```shell
nano /var/www/your_domain/html/index.html
```

And then add config for your virtual domain:
```shell
sudo nano /etc/nginx/conf.d/your_domain.conf
```
```conf
server {
        listen 80;
        listen [::]:80;

        root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

Test to make sure there's no syntax errors:
```shell
sudo nginx -t
```

Restart NGINX:
```shell
sudo systemctl restart nginx
```

Update SELinux:
```shell
sudo chcon -vR system_u:object_r:httpd_sys_content_t:s0 /var/www/your_domain/
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
sudo curl -sL https://rpm.nodesource.com/setup_22.x | sudo bash -
sudo yum install nodejs -y
```

### Installing PM2 for Node Process Management
```shell
sudo npm install pm2 -g
sudo pm2 startup
```


## Installing & Logging into Firebase CLI

```shell
curl -sL https://firebase.tools | bash
firebase login
```

## Installing & Logging into Netlify CLI

```shell
sudo npm install netlify-cli -g
sudo netlify login
```

## Installing Angular CLI

```shell
sudo npm install -g @angular/cli
```

## Installing & Logging into Google Cloud CLI

```shell
sudo curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-cli-420.0.0-linux-x86_64.tar.gz
sudo tar -xf google-cloud-cli-420.0.0-linux-x86_64.tar.gz
sudo ./google-cloud-sdk/install.sh 
sudo ./google-cloud-sdk/bin/gcloud auth login
```

## Installing & Logging into the DigitalOcean CLI

* First, create an access token [here](https://cloud.digitalocean.com/account/api/tokens), copy that access token.

```shell
cd ~
sudo wget https://github.com/digitalocean/doctl/releases/download/v1.92.0/doctl-1.92.0-linux-amd64.tar.gz
sudo tar xf ~/doctl-1.92.0-linux-amd64.tar.gz
sudo sudo mv ~/doctl /usr/local/bin
doctl auth init
doctl serverless install
```

## Installing & Logging into the Linode CLI

```sh
sudo yum install -y pip3
pip3 install linode-cli --upgrade
pip3 install boto
linode-cli configure
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
