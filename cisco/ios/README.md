# Cisco IOS Tutorial

## Starting the Tutorial
First step, power on your device. A fresh install of IOS should have serial and telnet still enabled, if so it is perfectly fine to use those connections for local configuration on a secured or wind-gapped network. This is because they send data un-encrypted over a network, and snooping parties can easily intercept the packets/frames to steal the data. So, in production, or over a public internet connection, always use SSH to connect, or any other encrypted data exchange protocol. 

### Serial connection
Most Cisco devices have "console" inputs, usually RJ-45, and you can connect to these using a rollover cable, USB-to-serial, connected to serial-to-consol. Such allows you to connect a COM port on your computer (which represents a USB I/O) to the Cisco device directly.

Simply open up a serial connection tool, such as PuTTY (for Windows), under "Connection type", click the option that says "Serial".

Type the correct COM port which your connector is located, and click "open".

```
enable
```

The enable command brings you into priveleged exec mode whereupon you may change the configuration of the device.


```
config term
```

Short for "configure terminal", which brings you into "configuration" mode.

### NAT Link to Remote Linux Server
Here are the high-level steps to set up NAT resolution between your Cisco router and remote Linux VPS with a public IP:
1. Configure your Linux VPS to have a static IP address.
2. Configure the Cisco router as the default gateway for your Linux VPS.
3. On the Cisco router, configure NAT (Network Address Translation) to map the public IP of your Linux VPS to an internal IP on your LAN.
4. On the Cisco router, configure port forwarding to forward incoming traffic to the internal IP of your Linux VPS.

Here is an example of the command-line interface (CLI) configuration:
```
! On the Cisco Router
! Configure NAT
ip nat inside source static <internal IP of Linux VPS> <public IP of Linux VPS>
! Configure port forwarding
ip access-list extended <ACL name>
permit tcp any host <public IP of Linux VPS> eq <port number>
! Apply the ACL to the NAT rule
ip nat inside destination list <ACL name> interface <outside interface name> overload
```

Now, move over to the Linux device, check if IPv4 forwarding is enabled, run:
```sh
cat /proc/sys/net/ipv4/ip_forward
```
If it is 1 (enabled), go ahead. If not, you will have to put net.ipv4.ip_forward=1 on /etc/sysctl.conf and run sysctl -p.

Once you're sure IPv4 forwarding is enabled, run:
```sh
iptables -t nat -A  PREROUTING -d 8.8.8.8 -j DNAT --to-destination 192.168.0.10
```

