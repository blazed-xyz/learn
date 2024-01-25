# Production Node

## Install FreeSWITCH

Step 1:
```shell
sudo rpm -ivh http://repo.okay.com.mx/centos/8/x86_64/release/okay-release-1-5.el8.noarch.rpm
sudo sed -i 's/^failovermethod=.*/#failovermethod=priority/g' /etc/yum.repos.d/okay.repo
sudo yum -y install epel-release
sudo yum install freeswitch-config-vanilla freeswitch-lang-* freeswitch-sounds-*
```

Create systemd unit file:
```shell
cat <<EOF | sudo tee /etc/systemd/system/freeswitch.service
[Unit]
Description=freeswitch
Wants=network-online.target
Requires=network.target local-fs.target
After=network.target network-online.target local-fs.target

[Service]
; service
Type=forking
Environment="DAEMON_OPTS=-nonat"
EnvironmentFile=-/etc/default/freeswitch
ExecStart=/usr/bin/freeswitch -ncwait ${DAEMON_OPTS}
RestartSec=90
Restart=always
; exec
;User=root
;Group=daemon
LimitCORE=infinity
LimitNOFILE=100000
LimitNPROC=60000
LimitSTACK=250000
LimitRTPRIO=infinity
LimitRTTIME=infinity
IOSchedulingClass=realtime
IOSchedulingPriority=2
CPUSchedulingPolicy=rr
CPUSchedulingPriority=89
UMask=0007
NoNewPrivileges=false

[Install]
WantedBy=multi-user.target
EOF
```

```shell
sudo systemctl daemon-reload
sudo systemctl start freeswitch
sudo systemctl enable freeswitch
fs_cli
```


## Keycloak
```shell
docker run -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:23.0.3 start-dev
```

## Authentik
```shell
sudo wget https://goauthentik.io/docker-compose.yml
sudo nano .env
```

```
PG_PASS=password
AUTHENTIK_SECRET_KEY=password
```

```shell
docker-compose up
```

## Kong
```shell
curl -Ls https://get.konghq.com/quickstart | bash
```

or

```shell
docker run -d --name kong-gateway kong/kong-gateway:3.5.0.2
```

## Kill Bill

Docker Compose (docker-compose.yml):
```dockerfile
version: '3.2'
volumes:
  db:
services:
  killbill:
    image: killbill/killbill:0.24.3
    ports:
      - "8080:8080"
    environment:
      - KILLBILL_DAO_URL=jdbc:mysql://db:3306/killbill
      - KILLBILL_DAO_USER=root
      - KILLBILL_DAO_PASSWORD=killbill
      - KILLBILL_CATALOG_URI=SpyCarAdvanced.xml
  kaui:
    image: killbill/kaui:2.0.11
    ports:
      - "9090:8080"
    environment:
      - KAUI_CONFIG_DAO_URL=jdbc:mysql://db:3306/kaui
      - KAUI_CONFIG_DAO_USER=root
      - KAUI_CONFIG_DAO_PASSWORD=killbill
      - KAUI_KILLBILL_URL=http://killbill:8080
  db:
    image: killbill/mariadb:0.24
    volumes:
      - type: volume
        source: db
        target: /var/lib/mysql
    expose:
      - "3306"
    environment:
      - MYSQL_ROOT_PASSWORD=killbill
```

```shell
docker-compose up
```

## Coolify
```shell
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```

## Gitblit
```shell
sudo docker run -d --name gitblit -p 8443:8443 -p 8080:8080 -p 9418:9418 -p 29418:29418 gitblit/gitblit
```

## Code Server
```shell
curl -fsSL https://code-server.dev/install.sh | sh
```

## Flarum
```shell
docker run --rm -v $(pwd):/var/www/html -e PUID=$(id -u) -e PGID=$(id -g) shinsenter/flarum composer dump-autoload
```

## CloudStack
Set selinux status to permissive:
```shell
sudo nano /etc/selinux/config
```

```shell
sudo yum -y install chrony wget
sudo nano /etc/yum.repos.d/cloudstack.repo
```

```
[cloudstack]
name=cloudstack
baseurl=http://cloudstack.apt-get.eu/centos/$releasever/4.15/
enabled=1
gpgcheck=0
```

```shell
yum -y install mysql-server
sudo nano /etc/my.cnf.d/mysql-server.cnf
```

```
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=350
log-bin=mysql-bin
binlog-format = 'ROW'
```

```shell
sudo nano /etc/my.cnf.d/cloudstack.cnf
```

```
[mysqld]
```

```shell
systemctl start mysqld
mysql -u root -p
ALTER USER 'root'@'localhost' IDENTIFIED BY 'passw0rd';
\q
systemctl restart mysqld.service
yum -y install cloudstack-management
mysqld --skip-grant-tables --user=mysql
alternatives --config java
cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:passw0rd
cloudstack-setup-management
sudo firewall-cmd --add-port={8080/tcp,8250/tcp,8443/tcp,9090/tcp} --permanent
sudo firewall-cmd --reload
```

