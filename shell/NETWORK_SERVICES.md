# Network Services

First, assure the computer's network config is set:

```shell
sudo nmcli device
```

Will return:
<IFACE_NAME>

```shell
sudo nano /etc/sysconfig/network-scripts/ifcfg-<IFACE_NAME>
```

```
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=no
NAME=<IFACE_NAME>
DEVICE=<IFACE_NAME>
ONBOOT=yes
IPADDR=10.0.0.15
PREFIX=24
GATEWAY=10.0.0.1
DNS1=8.8.8.8
DNS2=8.8.4.4
IPV6_DISABLED=yes
```

or set via the shell:

```shell
sudo nmcli connection modify <IFACE_NAME> ipv4.addresses 10.0.0.15/24
sudo nmcli connection modify <IFACE_NAME> ipv4.gateway 10.0.0.1
sudo nmcli connection modify <IFACE_NAME> ipv4.dns 10.0.0.10 10.0.0.11 10.0.0.12
sudo nmcli connection modify <IFACE_NAME> ipv4.dns-search blazed.world
```

## Setting up LDAP
Install:
```sh
sudo yum config-manager --set-enabled plus
sudo yum install -y openldap openldap-clients openldap-server
sudo yum --enablerepo=epel -y && sudo yum install -y openldap-servers openldap-clients
```

Configure the database for the OpenLDAP server. The database is stored in the directory /var/lib/ldap. You can create the directory with the following command:
```sh
sudo mkdir /var/lib/ldap
```

Set the ownership and permissions for the directory with the following command:
```sh
sudo chown -R ldap:ldap /var/lib/ldap
sudo chmod 700 /var/lib/ldap
```

Generate a new Master Password:
```sh
sudo slappasswd
```

Modify the config file:
```sh
sudo nano ldaprootpasswd.ldif
```

Add the contents:

```
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}PASSWORD_CREATED
```

Then run:
```sh
sudo systemctl enable slapd && sudo systemctl restart slapd
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f ldaprootpasswd.ldif  
```

Create the initial directory structure by using the following command:
```sh
sudo slapadd -n 0 -F /etc/openldap/slapd.d
```

Set the ownership and permissions for the directory with the following command:
```sh
sudo chown -R ldap:ldap /etc/openldap/slapd.d
sudo chmod 700 /etc/openldap/slapd.d
```

Generate a password:


Modify the file /etc/openldap/slapd.d/cn=config/olcDatabase={2}hdb.ldif to set the root password. Add the following lines after the line olcRootPW: {CLEARTEXT}password:
```
replace: olcRootPW
olcRootPW: {CLEARTEXT}your_password
```

Start the OpenLDAP server using the following command:
```sh
sudo systemctl start slapd
sudo systemctl enable slapd
```

Verify the OpenLDAP server is running correctly by using the following command:
```sh
sudo ldapsearch -x -b '' -s base '(objectclass=*)' namingContexts
```

## Setting up Samba 
Install:
```sh
sudo yum install samba samba-common samba-client
```

Next, we will create the share, run the following commands:
```sh
sudo mkdir -p /srv/blazed/data
sudo chmod -R 755 /srv/blazed/data
sudo chcon -t samba_share_t /srv/blazed/data
```

After that, open the necessary port(s) to prevent issue:
```sh
sudo firewall-cmd --add-service=samba --zone=public --permanent
sudo firewall-cmd --reload
```

Once Samba is installed, you need to configure it to share resources on your network. The main configuration file is /etc/samba/smb.conf. Open this file using your preferred text editor, such as nano:
```sh
sudo nano /etc/samba/smb.conf
```

In this file, you can specify the resources you want to share, such as folders and printers. For example, to share a folder named share in your home directory, you can add the following lines to the file:
```
[global]
  workgroup = OFFICE
  server string = Samba Server %v
  netbios name = blz-one
  security = user
  map to guest = bad user
  dns proxy = no
  ntlm auth = true

[Public]
   path = /srv/blazed/data
   available = yes
   valid users = yourusername
   read only = no
   browsable = yes
   guest ok = yes
```

To verify the configurations made, run the command:
```sh
sudo testparm
```

Start the service:
```sh
sudo systemctl start smb
sudo systemctl enable smb
sudo systemctl start nmb
sudo systemctl enable nmb
```

* Secure the Samba private directory:
First, create a new Samba user, and set a default password, as follows:
```sh
sudo useradd smbuser
sudo smbpasswd -a smbuser
```

Next, we will create a new group for this user:
```sh
sudo groupadd smb_group
sudo usermod -g smb_group smbuser
```

Now, we can create a private share:
```sh
sudo mkdir -p  /srv/blazed/private
sudo chmod -R 770 /srv/blazed/private
sudo chcon -t samba_share_t /srv/blazed/private
sudo chown -R root:smb_group /srv/blazed/private
```

Finally, change your config to support the private share:
```sh
sudo nano /etc/samba/smb.conf
```

Add to the bottom of your config:
```
[Private]
  path = /srv/blazed/private
  valid users = @smb_group
  guest ok = no
  writable = no
  browsable = yes
```

Now, restart services to see changes:
```sh
sudo systemctl restart smb && sudo systemctl restart nmb
```

To connect:
```sh
sudo yum install samba-client
```

Then, connect to the server as follows:
```sh
smbclient ‘\[SERVER-IP]\private’ -U smbuser
```

* Where *SERVER-IP* is the IP address, hostname, or FQDN of the server.

## Setting up DHCP Server

A DHCP server provides automatic IP address (and other provisions) to clients on a lan segment.

First, install:

```shell
sudo yum install dhcp-server -y
```

Then, configure:

```shell
sudo nano /etc/dhcp/dhcpd.conf
```

```
# Set domain name
option domain-name     		"blazed.world";

# Set DNS server's hostname or IP address
option domain-name-servers  dc.blazed.space;

default-lease-time 600;
max-lease-time    7200;
authoritative;

# Specify network address and subnet mask.
subnet 10.0.0.0 netmask 255.255.255.0 {

	# Set the IP lease range (~204 hosts)
	range dynamic-bootp 10.0.0.50 10.0.0.254;
	
	# Set broadcast address
	option broadcast-address 10.0.0.255;
	
	# Set gateway
	option routers 10.0.0.1;
}

```

Enable the DHCP service:

```shell
sudo systemctl enable --now dhcpd
sudo systemctl start dhcpd
sudo firewall-cmd --add-service=dhcp --zone=public --permanent
sudo firewall-cmd --reload
```

## Setting up DNS Server

```shell
sudo yum install bind bind-utils -y
sudo nano /etc/named.conf
```

A. Recursive (Resolving) DNS Servers

```shell
acl internal-network {
	10.0.0.0/24;
};

options {
	listen-on port 53 { any; };
	listen-on-v6 { any; };
	forwarders { 8.8.8.8; 8.8.4.4; 1.1.1.1; 1.0.0.1; };
	directory       "/var/named";
	dump-file       "/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	secroots-file   "/var/named/data/named.secroots";
	recursing-file  "/var/named/data/named.recursing";
	allow-query		{ localhost; internal-network; };
	# Add secondary DNS servers below if applicable
	allow-transfer  { localhost; };
	...
	...
	...
	# Set recursion to "YES" on recursive DNS servers.
	recursion yes;
	dnssec-enable yes;
	dnssec-validation yes;
	managed-keys-directory "/var/named/dynamic";
	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
	/* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
	include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
	channel default_debug {
		file "data/named.run";
		severity dynamic;
	};
};

zone "." IN {
	type hint;
	file "named.ca";
};

# Add forward and reverse ZONES for names:

zone "blazed.world" IN {
	type master;
	file "blazed.world.lan";
	allow-update { none; };
};

zone "0.0.10.in-addr.arpa" IN {
	type master;
	file "0.0.10.db";
	allow-update { none; };
};

# Network Address	10.0.0.0
# Write Like:		0.0.10.in-addr.arpa

# Network Address   192.168.1.0
# Write Like:		1.168.192.in-addr.arpa

```

Next, configure your zone files:

```shell
sudo nano /var/named/blazed.world.lan
sudo nano /var/named/0.0.10.db
```

Finally, enable BIND, and add firewall rule:

```shell
sudo systemctl enable --now named
sudo systemctl restart named
sudo firewall-cmd --add-service=dns --permanent
sudo firewall-cmd --reload
```


B. Authoritative (Primary) DNS Servers

```shell
options {
	listen-on port 53 { any; };
	listen-on-v6 { any; };
	forwarders { 8.8.8.8; 8.8.4.4; 1.1.1.1; 1.0.0.1; };
	directory       "/var/named";
	dump-file       "/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	secroots-file   "/var/named/data/named.secroots";
	recursing-file  "/var/named/data/named.recursing";
	allow-query		{ any; };
	# Add secondary DNS servers below if applicable
	allow-transfer  { localhost; };
	...
	...
	...
	# Set recursion to "NO" on authoritative DNS servers.
	recursion no;
	dnssec-enable yes;
	dnssec-validation yes;
	managed-keys-directory "/var/named/dynamic";
	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
	/* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
	include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
	channel default_debug {
		file "data/named.run";
		severity dynamic;
	};
};

zone "." IN {
	type hint;
	file "named.ca";
};

# Add forward and reverse ZONES for names:

zone "blazed.world" IN {
	type master;
	file "blazed.world.wan";
	allow-update { none; };
};

# Where the public IP is 1.2.3.4 => 4.3.2.1
zone "4.3.2.1.in-addr.arpa" IN {
	type master;
	file "4.3.2.1.db";
	allow-update { none; };
};
```

Next, configure your zone files:


```shell
sudo nano /var/named/blazed.world.wan
sudo nano /var/named/4.3.2.1.db
```

Finally, enable BIND, and add firewall rule:

```shell
sudo systemctl enable --now named
sudo systemctl start named && sudo systemctl restart named
sudo firewall-cmd --add-service=dns --permanent
sudo firewall-cmd --reload
```

## Setting Up a NIS Server

Use a NIS server to declare network config settings.

```shell
sudo yum install ypserv rpcbind -y
```

Next, set NIS domain and setup NIS IP space.

```shell
sudo ypdomainname blazed.world
sudo nano /etc/sysconfig/network
```

Within "network", set your NIS domain:

```
NISDOMAIN=blazed.world
```

Next, specify network range

```shell
sudo nano /var/yp/securenets
```

```
# add hosts that are in the NIS (server & client)
255.0.0.0		127.0.0.0
255.255.255.0	10.0.0.0
```

Assure hosts maps include all network hosts:

```shell
sudo nano /etc/hosts
```

Enable NIS server:

```shell
sudo systemctl enable --now rpcbind ypserv ypxfrd yppasswdd nis-domainname
sudo systemctl start rpcbind ypserv ypxfrd yppasswdd nis-domainname
```

Update NIS database:

```
sudo /usr/lib64/yp/ypinit -m
```

If SELinux is enabled:

```shell
sudo setsebool -P nis_enabled on
sudo setsebool -P domain_can_mmap_files on
```

If Firewalld is running, run:

```shell
sudo nano /etc/sysconfig/network
```

add to the end:

```
YPSERV_ARGS="-p 944"
YPXFRD_ARGS="-p 945"
```

Then, run:

```shell
sudo nano /etc/sysconfig/yppasswdd
```

Add the following:

```
YPPASSWDD_ARGS="--port 950"
```

Then run:

```shell
sudo systemctl restart rpcbind ypserv ypxfrd yppasswdd
sudo firewall-cmd --add-service=rpc-bind --permanent
sudo firewall-cmd --add-port={944-951/tcp,944-951/udp} --permanent
```

## Kerberos

```sh
sudo yum install krb5-server.x86_64 krb5-workstation.x86_64 -y
sudo nano /etc/krb5.conf
```

* Set KDC DB Master Password:
```sh
sudo kdb5_util create -s
```

* Create admin account:
```sh
sudo kadmin.local -q "addprinc admin/admin"
```

* Enable Kerberos Service
```sh
sudo systemctl enable krb5kdc --now && sudo systemctl enable kadmin --now
```

* Add other users
```sh
sudo kadmin.local
addprinc blazed@BLAZED.DEV
ktadd -k /etc/blazed.keytab blazed@BLAZED.CC
exit
sudo chmod 644 /etc/blazed.keytab
```