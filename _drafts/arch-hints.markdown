---
title: Arch hints
date: 2017-12-04 16:10:00 Z
---

#Arch - a few quick hits

### Recovery recently removed packages

```
for i in `cat  /var/log/pacman.log | awk '/removed/ && /07:13/ { print $5}'; do pacman -S --noconfirm $i; done

```
