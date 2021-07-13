# Server Provisioning Steps & Standards
## Blazed Labs LLC (c) 2021 Tyler Ruff

1. Install CentOS 8
2. Install some pre-recs
```shell
sudo yum install -y git nano gcc-c++ make
```


3. Create user 'blazed'

```shell
sudo adduser blazed
sudo passwd blazed
sudo usermod -aG wheel blazed
```

4. Secure server with SSH key & disable root login & change bind port

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
sudo chmod 700 ~/.ssh
sudo chmod 600 ~/.ssh/authorized_keys
sudo chown $USER:$USER ~/.ssh -R
```

6. Setup Git

For access machines

```shell
git config --global user.name "FirstLast-XX##"
git config --global user.email first.last@blazed.work
git config --global user.signingkey [GPG-KEY-NAME]
```

For on-prem machines

```shell
git config --global user.name "BlazedLabs-TX#"
git config --global user.email git@blazed.space
```

For cloud machines

```shell
git config --global user.name "BlazedLabs-CX#"
git config --global user.email dev@blazed.space
```

7. Other basic setup tasks

```shell
sudo timedatectl set-timezone America/New_York
```

## Create a swapfile

```shell
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
sudo sh -c 'echo "/swapfile none swap sw 0 0" >> /etc/fstab'
```

## Set Hostname

```shell
hostnamectl set-hostname 'BLZ-CX#'
```
