# iptables-legacy 与 iptables-nft 的关系与区别

## 区别

- iptables-legacy 调用的是 iptables kernel API，iptables-nft 调用的是 nftables kernel API，所以它们的规则在不同内核模块中管理，nft 命令只读取 nftables 内核模块中的规则，所以 nft list ruleset 只能看到 iptables-nft 配置的规则。

## 共同点

- 都兼容 iptables 命令配置规则的语法。
- 匹配报文逻辑共用的同一份代码(xtables match)，nft 配置的规则才会用 nftables match。

## 与 iptables 命令的关系

- 通常在支持 nftables 的发行版中才会有 iptables-legacy 与 iptables-nft，而 iptables 命令本身是个软链，用户可以选择在两这者之间切换。
- iptables-legacy 等同于不支持 nftables 发行版的 iptables 命令，即传统的 iptables。
- 较新发行版往往将 iptables 软链到 iptables-nft。

## 最佳实践

- 当系统支持 iptables-nft 时，可以取代 iptables-legacy。
- 不要两者混用，避免混淆。

## 参考资料

- [iptables: The two variants and their relationship with nftables](https://developers.redhat.com/blog/2020/08/18/iptables-the-two-variants-and-their-relationship-with-nftables)