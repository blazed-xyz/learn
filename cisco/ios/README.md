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