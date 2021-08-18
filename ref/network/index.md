---
title: "网络相关"
type: book
date: "2021-08-18"
---


## conntrack 统计

统计 insert_failed 总数:

```bash
conntrack -S | awk -F "=" '{print $7}' | awk '{sum+=$1} END {print sum}'
```