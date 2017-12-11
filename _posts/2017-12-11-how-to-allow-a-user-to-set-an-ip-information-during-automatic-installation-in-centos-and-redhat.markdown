---
title: How to allow a user to set an IP information  during automatic installation
  in CentOS 7 and RedHat 7
date: 2017-12-11 16:22:00 Z
categories:
- Linux
tags:
- CentOS7
- RedHat7
---

One of my clients wanted me to create him unattended CentOS boot cd,  but he also wanted to set up a static IP.  Here is a solution. 
**a fragment of a kickstart file **
```sh 
post
exec < /dev/tty6 > /dev/tty6 2> /dev/tty6
chvt 6
IFS=$'\n'
echo -n "Enter IP: "
read IP
echo
echo -n "You entered:" "$IP"
echo

echo -n "Enter NETMASK: "
read NETMASK
echo
echo -n "You entered:" "$NETMASK"
echo

echo -n "Enter GW: "
read GW
echo
echo -n "You entered:" "$GW"
echo

echo -n "Enter NIC Name: "
read NIC
echo
echo -n "You entered:" "$NIC"
echo
if [ -z "${NIC}" ]; then
    NIC=eth0
fi
cat >  /etc/sysconfig/network-scripts/ifcfg-eth0 <<EOF
BOOTPROTO="static"
ONBOOT="yes"
TYPE="Ethernet"
IPADDR=$IP
NETMASK=$NETMASK
GATEWAY=$GW
NM_CONTROLLED="no"
EOF

chvt 1
exec < /dev/tty1 > /dev/tty1 2> /dev/tty1
cat  >/etc/sysconfig/network <<  EOF
DNS1=8.8.8.8
EOF

sed -i 's/quiet"/net.ifnames=0 biosdevname=0"/' /etc/default/grub && grub2-mkconfig -o /boot/grub2/grub.cfg
%end
```
