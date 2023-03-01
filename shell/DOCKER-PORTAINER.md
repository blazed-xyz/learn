# Docker and/or Portainer
## Install Docker

```shell
sudo yum install -y yum-utils
sudo yum remove runc -y
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker $USER
newgrp docker
sudo systemctl enable docker && sudo systemctl enable containerd
sudo systemctl start docker && sudo systemctl start containerd
```
### Install Docker Compose

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## Install Portainer

Please make sure Docker and Docker Compose are installed first.

```shell
docker volume create portainer_data
```

## To run by itself (not behind rev-proxy)

```shell
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

Then, go to [IP-ADDR]:8000 and set your username/password.

## To run behind a reverse proxy (NGINX)

```shell
sudo nano ~/docker-compose.yml
```

```yml
version: "2"

services:
  nginx-proxy:
    image: jwilder/nginx-proxy
    restart: always
    networks:
      - proxy
    ports:
      - "80:80"
    volumes:
      - "/var/run/docker.sock:/tmp/docker.sock:ro"
      - "./vhost.d:/etc/nginx/vhost.d:ro"

  portainer:
    image: portainer/portainer-ce:2.5.1
    command: -H unix:///var/run/docker.sock
    restart: always
    networks:
      - proxy
    environment:
      - VIRTUAL_HOST=portainer.yourdomain.com
      - VIRTUAL_PORT=9000
    ports:
      - 8000:8000
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

networks:
  proxy:

volumes:
  portainer_data:
```

```shell
docker-compose up -d
```

Then, visit portainer.yourdomain.com and set your username/password.