---
layout:     post
title:      gPTP Test Theory
subtitle:   AVTP
date:       2018-09-08
author:     Richie
header-img: img/post-PKI-AVTP-diagram.jpg
catalog: true
tags:
    - gPTP
    - AVTP
    - TSN
---




# 1.	Check the features:
```
richie@richie-VirtualBox:~/Downloads/OpenAvnu-master/daemons/gptp/linux/build/obj$ ./daemon_cl 
INFO     : GPTP [21:00:20:754] gPTP starting
Interface name required
./daemon_cl <network interface> [-S] [-P] [-M <filename>] [-G <group>] [-R <priority 1>] [-D <gb_tx_delay,gb_rx_delay,mb_tx_delay,mb_rx_delay>] [-T] [-L] [-E] [-GM] [-INITSYNC <value>] [-OPERSYNC <value>] [-INITPDELAY <value>] [-OPERPDELAY <value>] [-F <path to gptp_cfg.ini file>] 
	-S start syntonization
	-P pulse per second
	-M <filename> save/restore state
	-G <group> group id for shared memory
	-R <priority 1> priority 1 value
	-D Phy Delay <gb_tx_delay,gb_rx_delay,mb_tx_delay,mb_rx_delay>
	-T force master (ignored when Automotive Profile set)
	-L force slave (ignored when Automotive Profile set)
	-E enable test mode (as defined in AVnu automotive profile)
	-V enable AVnu Automotive Profile
	-GM set grandmaster for Automotive Profile
	-INITSYNC <value> initial sync interval (Log base 2. 0 = 1 second)
	-OPERSYNC <value> operational sync interval (Log base 2. 0 = 1 second)
	-INITPDELAY <value> initial pdelay interval (Log base 2. 0 = 1 second)
	-OPERPDELAY <value> operational pdelay interval (Log base 2. 0 = 1 sec)
```

# 2.	Check the name of network card:
```
richie@richie-VirtualBox:~$ ifconfig
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::70f8:a129:df31:f0fc  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:d8:64:15  txqueuelen 1000  (Ethernet)
        RX packets 154235  bytes 211381511 (211.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 71109  bytes 4504965 (4.5 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 232  bytes 18664 (18.6 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 232  bytes 18664 (18.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

# 3.	 Execute the code:
```
#richie@richie-VirtualBox:~/Downloads/OpenAvnu-master/daemons/gptp/linux/build/obj$ sudo ./daemon_cl enp0s3 -V
```
