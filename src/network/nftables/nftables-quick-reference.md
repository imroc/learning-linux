# nftables 速查手册

## 表管理

nft 命令语法:

```bash
nft list tables [<family>]
nft list table [<family>] <name> [-n] [-a]
nft (add | delete | flush) table [<family>] <name>
```

其中 `family` 包括: ip, arp, ip6, bridge, inet, netdev.

示例:

```bash
# 创建表 (ipv4)
$ nft add table ip MyFirewall
# 列出所有表
$ nft list tables
# 删除指定表
$ nft delete table MyFirewall
```

## 链管理

nft 命令语法:

```bash
$ nft (add | create) chain [<family>] <table> <name> [ { type <type> hook <hook> [device <device>] priority <priority> \; [policy <policy> \;] } ]
$ nft (delete | list | flush) chain [<family>] <table> <name>
$ nft rename chain [<family>] <table> <name> <newname>
```

示例:

```bash
# 创建常规链
$ nft add chain ip MyFirewall mychian
# 创建基本链
$ nft add chain ip MyFirewall INPUT \{type filter hook input priority 0\; policy accept\; \}
# 查看所有链
$ nft list chains
# 查看指定链
$ nft list chain ip MyFirewall INPUT
# 删除指定链
$ nft delete chain ip MyFirewall INPUT
# 清空指定链所有规则
$ nft flush chain ip MyFirewall INPUT
```

## 规则管理

nft 命令语法:

```bash
$ nft add rule [<family>] <table> <chain> <matches> <statements>
$ nft insert rule [<family>] <table> <chain> [position <position>] <matches> <statements>
$ nft replace rule [<family>] <table> <chain> [handle <handle>] <matches> <statements>
$ nft delete rule [<family>] <table> <chain> [handle <handle>]
```

示例:

```bash
$ nft add rule ip MyFirewall INPUT icmp type echo-request reject
```

## 参考资料

- [Quick reference-nftables in 10 minutes](https://wiki.nftables.org/wiki-nftables/index.php/Quick_reference-nftables_in_10_minutes)