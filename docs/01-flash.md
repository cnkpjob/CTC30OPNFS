# 01 - 刷机

## 1.1 写盘

1. 进 PE，打开 DiskGenius 确认目标磁盘状态，必要时清理旧分区。
2. 打开 DiskImage，选择镜像 `kwrt-10.07.2026-x86-64-generic-squashfs-combined-efi.img`。
3. 选择目标 **Physical Disk**（务必确认不是分区，选错会导致写入失败或数据写错位置）。
4. 写入完成后安全弹出，设备上电启动。

> ⚠️ **踩坑提醒**：DiskImage 选盘时一定要看清是 Physical Disk，不要选成具体分区。

| 工具 | 用途 |
| --- | --- |
| DiskImage | PE 环境下写 IMG 镜像到物理磁盘 |
| DiskGenius | 查看 / 清理目标磁盘分区 |

## 1.2 首次进入系统后修改 IP

刷完后默认可能是 DHCP 模式，需要通过终端先改 IP 才能访问 LuCI。

SSH 登录（默认无密码，直接回车）：

```bash
ssh root@192.168.1.1
```

编辑网络配置：

```bash
vi /etc/config/network
```

将 LAN 接口改为静态 IP：

```text
config interface 'lan'
        option type 'bridge'
        option proto 'static'
        option ipaddr '192.168.31.190'
        option netmask '255.255.255.0'
        option gateway '192.168.31.1'
        option dns '192.168.31.1'
        option ifname 'eth0'
```

保存退出后重启网络：

```bash
/etc/init.d/network restart
```

电脑重连网线，用新地址访问 LuCI：

```text
http://192.168.31.190
```

> 完整配置片段见 [`../config/network-lan.conf`](../config/network-lan.conf)。

---

[返回主页](../README.md)
