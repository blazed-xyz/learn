# Activate Windows

## Windows Server
Run the following commands in a new power shell (administrator) console:
```shell
DISM /Online /Get-TargetEditions

DISM /online /Set-Edition:ServerStandard /ProductKey:[ENTER PRODUCT KEY HERE] /AcceptEula

cscript C:\Windows\System32\slmgr.vbs /ipk 4HY2F-JN3MR-TRC9V-4CW3K-QV8VQ 

cscript C:\Windows\System32\slmgr.vbs /skms kms_server

cscript C:\Windows\System32\slmgr.vbs /skms kms9.msguides.com

cscript C:\Windows\System32\slmgr.vbs /ato
```