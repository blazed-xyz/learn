# Server Provisioning Steps & Standards
## Blazed Labs LLC (c) 2021 Tyler Ruff

1. Install CentOS 8 or CentOS 7
2. Install some pre-recs
```shell
sudo yum install -y git nano gcc-c++ make
```


3. Create user 'swell'

```shell
sudo adduser swell
sudo passwd swell
sudo usermod -aG wheel swell
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


1. Other basic setup tasks

```shell
sudo timedatectl set-timezone America/New_York
```

## Set Hostname

```shell
hostnamectl set-hostname 'blz-cx#'
```
