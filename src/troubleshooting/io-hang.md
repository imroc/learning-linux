# 排查 IO 高负载

## 使用 iostat 检查设备是否 hang 住

```bash
iostat -xhd 2
```

如果有 100% 的 `%util` 的设备，说明该设备基本 hang 住了:

![](https://image-host-1251893006.cos.ap-chengdu.myqcloud.com/20221004084331.png)

## 观察高 IO 的磁盘读写情况

```bash
# 捕获 %util 超过 90 时 vdb 盘的读写指标，每秒检查一次
while true; do iostat -xhd | grep -A1 vdb | grep -v vdb | awk '{if ($NF > 90){print $0}}'; sleep 1s; done
```

## 查看哪些进程占住磁盘

```bash
fuser -v -m /dev/vdb
```

## 查找 D 状态的进程

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

## 使用 ebpf 抓高 IOPS 进程

安装 bcc-tools:
```bash
yum install -y bcc-tools
```

分析:
```bash
$ cd /usr/share/bcc/tools
$ ./biosnoop 5 > io.txt
$ cat io.txt | awk '{print $3,$2,$4,$5}' | sort | uniq -c | sort -rn | head -10
   6850 3356537 containerd vdb R
   1294 3926934 containerd vdb R
    864 1670 xfsaild/vdb vdb W
    578 3953662 kworker/u180:1 vda W
    496 3540267 logsys_cfg_cli vdb R
    459 1670 xfsaild/vdb vdb R
    354 3285936 php-fpm vdb R
    340 3285934 php-fpm vdb R
    292 2952592 sap1001 vdb R
    273 324710 python vdb R
$ pstree -apnhs 3356537
systemd,1 --switched-root --system --deserialize 22
  └─containerd,3895
      └─{containerd},3356537    
$ timeout 10 strace -fp 3895 > strace.txt 2>&1
# vdb 的 IOPS 高，vdb 挂载到了 /data 目录，这里过滤下 "/data"
$ grep "/data" strace.txt | tail -10
[pid 19562] newfstatat(AT_FDCWD, "/data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/6974/fs/data/log/monitor/snaps/20211010/ps-2338.log", {st_mode=S_IFREG|0644, st_size=6509, ...}, AT_SYMLINK_NOFOLLOW) = 0
[pid 19562] newfstatat(AT_FDCWD, "/data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/6974/fs/data/log/monitor/snaps/20211010/ps-2339.log", {st_mode=S_IFREG|0644, st_size=6402, ...}, AT_SYMLINK_NOFOLLOW) = 0
[pid 19562] newfstatat(AT_FDCWD, "/data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/6974/fs/data/log/monitor/snaps/20211010/ps-2340.log", {st_mode=S_IFREG|0644, st_size=6509, ...}, AT_SYMLINK_NOFOLLOW) = 0
[pid 19562] newfstatat(AT_FDCWD, "/data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/6974/fs/data/log/monitor/snaps/20211010/ps-2341.log", {st_mode=S_IFREG|0644, st_size=6509, ...}, AT_SYMLINK_NOFOLLOW) = 0
[pid 19562] newfstatat(AT_FDCWD, "/data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/6974/fs/data/log/monitor/snaps/20211010/ps-2342.log", {st_mode=S_IFREG|0644, st_size=6970, ...}, AT_SYMLINK_NOFOLLOW) = 0
[pid 19562] newfstatat(AT_FDCWD, "/data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/6974/fs/data/log/monitor/snaps/20211010/ps-2343.log", {st_mode=S_IFREG|0644, st_size=6509, ...}, AT_SYMLINK_NOFOLLOW) = 0
[pid 19562] newfstatat(AT_FDCWD, "/data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/6974/fs/data/log/monitor/snaps/20211010/ps-2344.log", {st_mode=S_IFREG|0644, st_size=6402, ...}, AT_SYMLINK_NOFOLLOW) = 0
[pid 19562] newfstatat(AT_FDCWD, "/data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/6974/fs/data/log/monitor/snaps/20211010/ps-2345.log",  <unfinished ...>
[pid 19562] newfstatat(AT_FDCWD, "/data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/6974/fs/data/log/monitor/snaps/20211010/ps-2346.log", {st_mode=S_IFREG|0644, st_size=7756, ...}, AT_SYMLINK_NOFOLLOW) = 0
[pid 19562] newfstatat(AT_FDCWD, "/data/containerd/io.containerd.snapshotter.v1.overlayfs/snapshots/6974/fs/data/log/monitor/snaps/20211010/ps-2347.log", Process 3895 detached
$ grep "/data" strace.txt > data.txt
# 合并且排序，自行用脚本分析下哪些文件操作多
$ cat data.txt | awk -F '"' '{print $2}' | sort | uniq -c | sort -n > data-sorted.txt
```