# Windows Workstation Setup

## Set GPG Key on Git
```shell
gpg --list-secret-keys --keyid-format LONG
git config --global user.signingkey [KEY SIGNATURE]
git config --global commit.gpgsign true
git config --global gpg.program "/c/Program Files (x86)/GnuPG/bin/gpg.exe"
```