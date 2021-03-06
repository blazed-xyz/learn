# Server Provisioning Steps & Standards
## Blazed Labs LLC (c) 2021 Tyler Ruff

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

6. Setup Git

For access machines

```shell
git config --global user.name "tyler-ruff"
git config --global user.email hello@blazed.space
git config --global user.signingkey [GPG-KEY-NAME]
```

For on-prem machines

```shell
git config --global user.name "BlazedLabs-TX#"
git config --global user.email dev@blazed.space
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
to
HandleLidSwitch=ignore

```shell
sudo systemctl restart systemd-logind.service
```

## Setup Domain & Host Mapping

```shell
sudo nano /etc/hosts
```