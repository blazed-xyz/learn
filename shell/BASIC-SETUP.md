# Server Provisioning Steps & Standards
## Blazed Labs LLC (c) 2021 Tyler Ruff

0. Some Useful Bash Functions
- Get external IP address
```sh
curl -4 ifconfig.co
```
- Get internal IP address
```sh
ip route get 8.8.8.8 | awk -F"src " 'NR==1{split($2,a," ");print a[1]}'
```
- Get Hostname
```sh
hostname -s
```
- Get full hostname+domain
```sh
name -n
```
- Get uptime duration
```sh
uptime | grep -ohe 'up .*' | sed 's/up //g' | awk -F "," '{print $1}'
```
- Get active users
```sh
uptime | grep -ohe '[0-9.*] user[s,]'
```
- Show system clock
```sh
timedatectl | sed -n '/Local time/ s/^[ \t]*Local time:\(.*$\)/\1/p'
```

1. Install Rocky Linux 8
2. Install some pre-recs
```shell
sudo yum install -y git nano gcc-c++ make
sudo yum update -y
```

3. Create user 'blazed'

```shell
sudo adduser blazed
sudo passwd blazed
sudo usermod -aG wheel blazed
```

4. Secure server with SSH key & disable root login

```shell
sudo nano /etc/ssh/sshd_config
```

```
 PermitRootLogin no
 AllowUsers blazed
 Protocol 2
 PasswordAuthentication no
 ChallengeResponseAuthentication no
 MaxAuthTries 5
 AuthorizedKeysFile %h/.ssh/authorized_keys
 ```

5. Assure you are logged in with the desired user

```shell
sudo mkdir ~/.ssh
sudo chmod 700 ~/.ssh
sudo nano ~/.ssh/authorized_keys
sudo chmod 600 ~/.ssh/authorized_keys
sudo chown $USER:$USER ~/.ssh -R
```

* To connect w/ key:
```sh
sudo ssh -i ~/Servers/[HOST]/[HOST] [USER]@[IP]
```

* Set SSH Banners

- Access Banner (Shown before login)
First, create a shell script for the banner, ex:
```sh
#!/bin/bash

echo -e "
You are now logging in to Blazed Servers
"
```
 clone the desired banner to '/etc/profile.d/welcomeBanner.sh', then run:
```sh
sudo chmod 644 /etc/profile.d/welcomeBanner.sh
sudo nano /etc/ssh/sshd_config
```
And uncomment "Banner", and add:
```
Banner /etc/profile.d/welcomeBanner.sh
```

- Welcome Banner (MOTD, shown only on successful login)
clone the desired welcome banner to '/etc/server-info.sh'
```

```

1. Setup Git

For access machines

First, [generate a new GPG key](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key), follow the official GitHub tutorial.

To get [GPG-KEY-NAME]:
```shell
gpg --list-secret-keys --keyid-format=long
```
You should get an output like:
```
sec   ed25519/[GPG-KEY-NAME]
```
You'll want to use the string of characters to the right of the slash, "[GPG-KEY-NAME]".

```shell
git config --global user.name "tyler-ruff"
git config --global user.email hello@blazed.space
git config --global user.signingkey [GPG-KEY-NAME]
git config --global commit.gpgsign true
```

For on-prem machines

```shell
git config --global user.name "BlazedLabs-TX#"
git config --global user.email hello@blazed.systems
```


1. Other basic setup tasks

```shell
sudo timedatectl set-timezone America/New_York
```

## Set Hostname

```shell
sudo hostnamectl set-hostname 'dc.blazed.space'
```

## Prevent sleep on lid close (if laptop)

```shell
sudo nano /etc/systemd/logind.conf
```

Change:
HandleLidSwitch=lock
HandleLidSwitchExternalPower=suspend
to
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore 

```shell
sudo systemctl restart systemd-logind.service
```

## Setup Domain & Host Mapping

```shell
sudo nano /etc/hosts
```

## Create SwapFile (for memory stack overflow prevention)
```sh
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```
