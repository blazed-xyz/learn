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

