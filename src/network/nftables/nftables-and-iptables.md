# nftables 与 iptables 的差异

## 表含义与链的优先级

iptables 有固定的几张表: mangle, nat, filter, raw；而 nftables 的表本身只是链的容器，没有语义和优先级，也没有内置任何表。

iptables 每个基本链可以存在多个表中，不同表中同名基本链的优先级不一样:

![](https://image-host-1251893006.cos.ap-chengdu.myqcloud.com/20221004101604.png)

而 nftables 中链的优先级是由数字定义，数字越小优先级越高(可以为负数)。

为了兼容 iptables 规则，往往表名和链的优先级都写成 iptables 中对应的表名，优先级也提供了预定义的对应 iptables 中表名的优先级(参考 [Standard chain priority values and textual names](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/getting-started-with-nftables_configuring-and-managing-networking#standard-chain-priority-values-and-textual-names_assembly_creating-and-managing-nftables-tables-chains-and-rules))，如:

```
table ip filter {
	chain INPUT {
		type filter hook input priority filter; policy accept;
...
```

## 参考资料

- [nftables简介](https://lework.github.io/2020/02/05/nftables/)