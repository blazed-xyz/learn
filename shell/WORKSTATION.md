# Workstation Provisioning Steps & Standards
## Blazed Labs LLC (c) 2021 Tyler Ruff

* Follow steps in [Basic Setup](BASIC-SETUP.md)

## Install Chrome

```sh
sudo yum install https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm
```

## Install GitHub Desktop

```sh
sudo rpm --import https://packagecloud.io/shiftkey/desktop/gpgkey
sudo sh -c 'echo -e "[shiftkey]\nname=GitHub Desktop\nbaseurl=https://packagecloud.io/shiftkey/desktop/el/7/\$basearch\nenabled=1\ngpgcheck=0\nrepo_gpgcheck=1\ngpgkey=https://packagecloud.io/shiftkey/desktop/gpgkey" > /etc/yum.repos.d/shiftkey-desktop.repo'
sudo yum install -y github-desktop
```

## Install VS Code

```sh
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
sudo yum install -y code
```

## Install Snap and Bitwarden

```shell
sudo yum install epel-release
sudo yum install snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
sudo systemctl restart snapd
sudo snap install bitwarden
```

## Install Node.js

```shell
sudo curl -sL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install nodejs -y
```

## Install Angular CLI
```shell
npm install -g @angular/cli
```

## Install Netlify CLI
```shell
npm install netlify-cli -g
netlify login
```

## Install Vercel CLI
```shell
npm i -g vercel
vercel login
```

## Install Firebase CLI & firebase-tools
```
npm install -g firebase-tools
firebase login
```

## Install Google Cloud CLI
```shell
(New-Object Net.WebClient).DownloadFile("https://dl.google.com/dl/cloudsdk/channels/rapid/", "$env:Temp\GoogleCloudSDKInstaller.exe")

& $env:Temp\GoogleCloudSDKInstaller.exe
```
Once installed you can authenticate:
```shell
gcloud auth login
```

## Install Sqlite
```shell
sudo yum install -y sqlite
```

## Install Ruby & Rails
```shell
sudo yum install -y ruby ruby-devel
gem install rails
```