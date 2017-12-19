---
title: Post-installation script at OVH
date: 2017-12-18 13:24:00 -08:00
categories:
- Linux
tags:
- CentOS7
- RedHat7
Key: 
Field name: 
---

My client asked me if it possible to automate a few actions when provisioning dedicated servers at OVH.

Unfortunately, OVH's control panel does not offer many options for customization, but they allow to execute a post-installation script.
 
![index.png](/uploads/index.png)

Here is an example of script we use to configure bridge and change default ssh port 
```sh
#!/bin/bash

## hope, it is obvious: we are getting network settings here
IP=`ifconfig | grep -A 1 eth0|tail -1|awk '{ print $2}'`
NETMASK=`ifconfig | grep -A 1 eth0|tail -1 |awk '{ print $4}'`
echo $NETMASK
GW=`ip route list| head -1|awk '{ print $3}'`
## Creating network config files
cat > /root/ifcfg-viifbr0 << EOF
DEVICE=viifbr0
TYPE=Bridge
BOOTPROTO="static"
ONBOOT="yes"
TYPE="Ethernet"
IPADDR=$IP
NETMASK=$NETMASK
GATEWAY=$GW
NM_CONTROLLED="no"
EOF
cat >  /root/ifcfg-eth0 <<EOF
DEVICE=eth0
BRIDGE=viifbr0
ONBOOT=yes
NM_CONTROLLED="no"
EOF
##0 Writing /etc/rc.local
cat > /etc/rc.local <<EOF
#!/bin/bash
mv /root/ifcfg-viifbr0 /etc/sysconfig/network-scripts/ifcfg-viifbr0
mv /root/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth0 

yum install bridge-utils net-tools wget -y
systemctl restart network

:> /etc/rc.local
touch /var/lock/subsys/local
EOF
## making /etc/rc.local executable
chmod +x /etc/rc.local
#sed -i 's/enforcing/disabled/' /etc/sysconfig/selinux
## changing a default ssh port
sed -i 's/^#Port 22/Port 8822/' /etc/ssh/sshd_config  
```