# CloudFlare DDNS Solution

## GitHub/CronJob Method
1. Clone the repo to home folder (~):
```shell
git clone https://github.com/timothymiller/cloudflare-ddns.git
cd cloudflare-ddns
sudo nano config-example.json
```
2. Add API Key, Token, and other credentials & save file.
3. Now, rename it to "config.json":
```shell
sudo mv config-example.json config.json
```
4. And setup a CronJob:
```shell
sudo crontab -e
```
5. Add the following to the document (for every 15 minutes):
```cron
*/15 * * * * /home/blazed/cloudflare-ddns/sync
```


## Docker Method
1. Install Docker
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
2. Add Docker Image
```shell
docker run \
  --network host \
  -e CF_API_TOKEN=YOUR-CLOUDFLARE-API-TOKEN \
  -e DOMAINS=example.org,www.example.org,example.io \
  -e PROXIED=true \
  favonia/cloudflare-ddns:latest
```