---
title: Arch's hints
date: 2017-12-04 16:10:00 Z
---

#Arch - a few quick hits

### Recovery recently removed packages

    for i in `cat  /var/log/pacman.log | awk '/removed/ && /07:13/ { print $5}'`; do pacman -S --noconfirm $i; done
    

### Unlock Pacman

    rm /var/lib/pacman/db.lck

### where pacman's cache files are located

    /var/cache/pacman/pkg/