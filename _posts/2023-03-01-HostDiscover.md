---
layout: single
title: Host Discovery
date: 2023-03-05
classes: wide
header:
  teaser: /assets/images/htb-writeup-smasher/smasher.png
categories:
  - mapping
  - scanning
  - evasion
tags:
  - mapping
  - scanning
  - evasion
  - nmap  
---
## Author  -  IRVING ST  -  (Comandre-ex)

Host discovery is a technique used to discover live hosts on the network. In this article you will find different techniques and ways to perform a host discovery and evasion of security systems and different tools as well as the development of scripts to automate it.

##  HOST DISCOVERY (ICMP)

This type of Host discovery makes use of sending icmp-type packets to a network segment, since this technique can be done with tools such as nmap or nping or the same ping command

### ICMP HOST DISCOVERY WITH PING

For this first way of performing the host discovery we will create a bash script to send the icmp packets to the entire network segment and through the response determine if the host is alive.

``` bash  
#!/usr/bin/env bash 
# Author: IRVING ST  - (comandre-ex)

trap ctrl_c  INT  

function ctrl_c(){

  echo -ne  "\n\nExiting..\n"
  tput cnorm ;  exit  1 
}

function enumeration(){
  for  i  in $(seq  70  250);do  
    ping -c1  192.168.1.${i}  > /dev/null 2>&1
    if [[ "$(echo  $?)" == "0"  ]];then  
      echo  -ne "\n\nHost:  192.168.1.${i}\n"
    else
      :
    fi
  done
}

enumeration

```

### ICMP HOST DISCOVERY  WITH NMAP  

The well-known nmap port scanner has within its parameters one that allows the enumeration of active hosts in the network, sending only icmp type packets.

``` shell
Tools - [nmap]

Examples:

1 - nmap -sn <IP range>  --min-rate  5000 |  grep  -oP  "\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}" | sort  |  uniq -u

2 - nmap -sn <IP range>   --min-rate 5000  |  grep  -oP  "\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}" | sort  |  uniq -u

3 - nmap -iL <file txt> --min-rate  5000 -oG HostActive

4 - nmap -iR 100 --min-rate  5000 -oG HostACTIVE

```


### ICMP HOST DISCOVERY  WITH NPING 

The ping tool is a package that the nmap tool has when you install it, it is a scanner that allows you to create modified packages


``` shell
Tools - [nping:nmap:masscan]

Examples:


1 - nping -c 5 <IP range>

2 - nping --tcp -c 5 -p 80  <IP range>  --icmp

3 - nping --udp -c 5 -p 53  <IP range> --icmp

4 - nmap -sP -vvv  <IP range> --min-rate 5000 -oG HostActive

5 - masscan -p0 --ping --rate=5000 <IP range> -oG HostActive
```

##  HOST DISCOVERY (ARP)

Host discovery by arp is a technique used to determine which hosts are active on a local network, it works by sending multiple arp packets to the entire network


``` shell
Tools - [netdiscover:fping:arping:arpscan:nmap:masscan]

Examples:

1 - netdiscover -i <network interface>

2 - netdiscover -i <network interface> -r <IP range>

3 - fping -a <IP range>

4 - arping -I <network  interface>  <IP address>

5 - arpscan <IP  range>

6 - nmap --min-rate  5000 -PR -vvvv <IP range>  -oG  HostActive

7 - masscan --range <IP range> -p0 --rate=5000 --localnet -oG  HostActive
```

##  HOST DISCOVERY EVASION FIREWALL:IDS:IPS

### packet fragmentation

One of the first techniques we'll look at is packet fragmentation. We are going to fragment the packets since when they arrive at the firewall it may have problems when rezancing it. For this we will use nmap and masscan

``` shell 
Tools - [nmap:masscan]

Examples:

1 - nmap -sn <IP  range>  -f  -vvv -oG HostActive

2 - nmap -sn <IP  range> -mtu 8  -vvv -oG  HostActive

4 - masscan --ping   --range <IP  range> -p0    --fragment -oG  HostActive

5 - masscan --range <IP range> -p0 --rate=5000  --fragment -oG HostActive

6 - masscan --range <IP range> -p0  --fragment  -oG HostActive

```

### source  port

The source port technique is based on using a specified port to perform the scan. This is because firewalls or ids often only allow traffic on a certain port.

``` shell
Tools - [nmap:masscan]

Examples:

1 - nmap  -sn --source-port <port> <IP range> --send-ip -oG HostActive

2 - masscan --ping  --source-port <port>  --range  <IP range>  -oG HostActive

3 - masscan --range <IP range> -p0  --source-port <port> --localnet -oG HostActive

```

### random scan 

Random scanning can help you bypass security systems like a firewall. This is because firewalls sometimes follow a traffic pattern or may be configured with a traffic pattern

``` shell  
Tools - [nmap:masscan]

Examples:

1 - nmap -sn  --randomize-hosts <IP  range>    --send-ip -oG  HostActive

2 - masscan --ping   --randomize-hosts --range <IP range> -oG  HostActive

3 - masscan  --randomize-hosts  -p0 --range <IP range> --localnet -oG HostActive

``` 

### spoof  macc address

This technique spoofs the mac address. An intrusion detection system such as a firewall may be making use of a traffic filter by mac addresses. By impersonating the mac we would be evading this security measure


``` shell
Tools - [nmap:masscan]

Examples:

1 - nmap -sn --spoof-mac  <mac addres> -oG  HostActive

2 - masscan --ping  --spoof-mac fc:fb:fb  <mac addrees>  <IP range> -oG HostActive

3 - masscan -p0  --spoof-mac fc:fb:fb  --range  <IP range> --localnet -oG HostActive
```

### basadum

This technique sends packets with a checksum error. This can even evade a possible intrusion detection system if it does not handle errors correctly.

``` shell 
Tools - [nmap]

Examples:

1 - nmap -sn --badsum <IP range>  -T5
```



