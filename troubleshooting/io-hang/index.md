---
title: "排查 IO hang 住"
type: book
date: "2021-07-27"
---

## 使用 iostat 检查设备是否 hang 住

```bash
iostat -xhd 2
```

如果有 100% 的 `%util` 的设备，说明该设备基本 hang 住了

![](1.png)

## 观察高 IO 的磁盘读写情况

```bash
# 捕获 %util 超过 90 时 vdb 盘的读写指标，每秒检查一次
while true; do iostat -xhd | grep -A1 vdb | grep -v vdb | awk '{if ($NF > 90){print $0}}'; sleep 1s; done
```

## 查看哪些进程占住磁盘

```bash
fuser -v -m /dev/vdb
```

# 查找 D 状态的进程

D 状态 (Disk Sleep) 表示进程正在等待 IO，不可中断，正常情况下不会保持太久，如果进程长时间处于 D 状态，通常是设备故障

```bash
ps -eo pid,ppid,stat,command

# 捕获 D 状态的进程
while true; do ps -eo pid,ppid,stat,command | awk '{if ($3 ~ /D/) {print $0}}'; sleep 0.5s; done
```

## 观察高 IO 进程

```bash
iotop -oP
# 展示 I/O 统计，每秒更新一次
pidstat -d 1
# 只看某个进程
pidstat -d 1 -p 3394470
```
