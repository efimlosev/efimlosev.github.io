---
title: MongoDB backup
date: 2017-12-30 20:23:00 -08:00
categories:
- Linux
tags:
- bash
- mongodump
---

Recently,  I had to backup MongoDB, and as I have previous experience working Mysql, I wanted to back it up using the same approach - dump every database in separated a file.

The problem is that I don't really know how to see in an easy way a list databases to the console. I have solved this problem,  but the solution is awkward

Here is my solution.

``` sh
for i in  `mongo -u backup -p password admin --eval "db.adminCommand('listDatabases')" | awk '{if (NR > 3) {print}}' |jq ."databases"| jq .[].name`;
do mongodump -u backup -p password --db $i  --archive  --authenticationDatabase admin> /backup/mongo/$i.mongo;
done

```


