---
title: Arch's hints
date: 2017-12-04 08:10:00 -08:00
categories:
- Linux
---

Arch - a few quick hits

 Recovery recently removed packages
```sh
    for i in `cat  /var/log/pacman.log | awk '/removed/ && /07:13/ { print $5}'`; do pacman -S --noconfirm $i; done
 ```   
Unlock Pacman

   ```sh rm /var/lib/pacman/db.lck```

Locations of pacman's cache files are located
```
    /var/cache/pacman/pkg/
```