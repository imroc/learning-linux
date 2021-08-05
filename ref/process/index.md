---
title: "进程相关"
type: book
date: "2021-08-05"
---

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