# 进程相关

## 查看指定进程 ID 的详细父子进程关系

```bash
$ pstree -apnhs 1004089
systemd,1 --system --deserialize 21
  └─dockerd,1216955 --config-file=/etc/docker/daemon.json
      └─docker-containe,1216972 --config /var/run/docker/containerd/containerd.toml
          └─docker-containe,832087 -namespace moby -workdir /data/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby/0be25e7e1583623fb6eff099dcbec8bb0eb52899cff85a8718bdfd4aa8ea0ca4 -address /var/run/docker/containerd/docker-containerd.sock -containerd-binary /usr/bin/docker-containerd -runtime-root ...
              └─systemd,832129
                  └─restorer,1004089 ../etc/config.xml
                      └─{restorer},1004093
```

## 查找异常状态的进程

查看 D 状态的进程:

```bash
ps -eo pid,ppid,stat,command | awk '{if ($3 ~ /D/) {print $0}}'
```

查看僵尸进程:

```bash
ps -eo pid,ppid,stat,command | awk '{if ($3 ~ /Z/) {print $0}}'
```

## 查找 fork 较多的进程或线程

查看使用相同启动命令的进程/线程数量排行:
```bash
ps -eLo command | sort | uniq -c | sort -rn | head -50
```

统计线程数排名

```bash
printf "NUM\tPID\tCOMMAND\n" && ps -eLf | awk '{$1=null;$3=null;$4=null;$5=null;$6=null;$7=null;$8=null;$9=null;print}' | sort |uniq -c |sort -rn | head -10
```