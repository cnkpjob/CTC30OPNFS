# 02 - 旁路由设置

## 2.1 关闭 IPv4 DHCP

LuCI 路径：**网络 → 接口 → LAN → 修改 → DHCP 服务器**

勾选「忽略此接口」，保存并应用。

## 2.2 关闭 IPv6 DHCP

同一页面切换到 **IPv6 设置** 选项卡：

- DHCPv6 服务：禁用
- RA 服务：禁用

保存并应用。

## 2.3 命令行方式（可选）

```bash
# 关闭 IPv4 DHCP
uci set dhcp.lan.ignore=1

# 关闭 IPv6 DHCP
uci set dhcp.lan.dhcpv6=disabled
uci set dhcp.lan.ra=disabled

uci commit dhcp
/etc/init.d/dnsmasq restart
```

## 2.4 注意事项

- 旁路由 LAN 口设为主路由网段内的静态 IP：`192.168.31.190`
- 旁路由的网关和 DNS 都指向主路由：`192.168.31.1`
- 关闭 DHCP 后，重启设备验证获取到的网关是否为 `192.168.31.1`

---

[返回主页](../README.md)
