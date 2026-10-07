# 升腾 C30 刷 OpenWrt 及 NFS 共享配置记录

> 把一台**升腾 C30** 瘦客户机刷成 OpenWrt（Kwrt），配置为「旁路网关 + NFS 家庭影音共享」的完整实操记录。
>
> 整理日期：2026-10-07

## 目录

- [设备与环境](#设备与环境)
- [一、刷机](#一刷机)
  - [1.1 写盘](#11-写盘)
  - [1.2 首次进入系统后修改 IP](#12-首次进入系统后修改-ip)
- [二、旁路由设置](#二旁路由设置)
  - [2.1 关闭 IPv4 DHCP](#21-关闭-ipv4-dhcp)
  - [2.2 关闭 IPv6 DHCP](#22-关闭-ipv6-dhcp)
  - [2.3 命令行方式（可选）](#23-命令行方式可选)
  - [2.4 注意事项](#24-注意事项)
- [三、NFS 共享配置](#三nfs-共享配置)
  - [3.1 安装](#31-安装)
  - [3.2 配置共享目录](#32-配置共享目录)
  - [3.3 目录权限（关键）](#33-目录权限关键)
  - [3.4 启动服务](#34-启动服务)
  - [3.5 客户端挂载](#35-客户端挂载)
- [四、避坑记录](#四避坑记录)
- [五、总结](#五总结)

---

## 设备与环境

| 项目 | 说明 |
| --- | --- |
| 设备 | 升腾 C30 |
| 系统 | OpenWrt（Kwrt） |
| 镜像 | `kwrt-10.07.2026-x86-64-generic-squashfs-combined-efi.img` |
| 用途 | 旁路由 + NFS 共享 |
| 旁路由 IP | `192.168.31.190` |
| 主路由网关 | `192.168.31.1` |
| 写盘工具 | DiskImage（PE 环境下写 IMG 镜像） |
| 磁盘管理 | DiskGenius |

---

## 一、刷机

### 1.1 写盘

1. 进 PE，打开 DiskGenius 确认目标磁盘状态，必要时清理旧分区。
2. 打开 DiskImage，选择镜像 `kwrt-10.07.2026-x86-64-generic-squashfs-combined-efi.img`。
3. 选择目标 **Physical Disk**（务必确认不是分区，选错会导致写入失败或数据写错位置）。
4. 写入完成后安全弹出，设备上电启动。

> ⚠️ **踩坑提醒**：DiskImage 选盘时一定要看清是 Physical Disk，不要选成具体分区。

### 1.2 首次进入系统后修改 IP

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

---

## 二、旁路由设置

### 2.1 关闭 IPv4 DHCP

LuCI 路径：**网络 → 接口 → LAN → 修改 → DHCP 服务器**

勾选「忽略此接口」，保存并应用。

### 2.2 关闭 IPv6 DHCP

同一页面切换到 **IPv6 设置** 选项卡：

- DHCPv6 服务：禁用
- RA 服务：禁用

保存并应用。

### 2.3 命令行方式（可选）

```bash
# 关闭 IPv4 DHCP
uci set dhcp.lan.ignore=1

# 关闭 IPv6 DHCP
uci set dhcp.lan.dhcpv6=disabled
uci set dhcp.lan.ra=disabled

uci commit dhcp
/etc/init.d/dnsmasq restart
```

### 2.4 注意事项

- 旁路由 LAN 口设为主路由网段内的静态 IP：`192.168.31.190`
- 旁路由的网关和 DNS 都指向主路由：`192.168.31.1`
- 关闭 DHCP 后，重启设备验证获取到的网关是否为 `192.168.31.1`

---

## 三、NFS 共享配置

### 3.1 安装

通过 OpenWrt 软件商店（opkg）安装：

- `luci-app-nfs`
- 中文语言包（可选）

装完后 LuCI 菜单会出现 **NFS 服务器** 入口。

### 3.2 配置共享目录

在 LuCI 的 NFS 管理界面添加共享：

| 字段 | 填写内容 |
| --- | --- |
| 路径 | 共享目录，如 `/mnt/sda1/media` |
| 允许的客户端 | 如 `192.168.31.0/24` |
| 选项 | `rw,sync,no_root_squash,insecure,no_subtree_check` |

对应 `/etc/exports`：

```text
/mnt/sda1/media 192.168.31.0/24(rw,sync,no_root_squash,insecure,no_subtree_check)
```

选项含义：

| 选项 | 含义 |
| --- | --- |
| `rw` | 读写权限 |
| `sync` | 同步写入 |
| `no_root_squash` | root 客户端保持 root 权限 |
| `insecure` | 允许客户端从大于 1024 的端口连接（部分播放器 / Kodi 需要） |
| `no_subtree_check` | 不检查父目录权限，减少兼容性问题 |

### 3.3 目录权限（关键）

NFS 只负责发布目录，客户端能否读写取决于服务端目录的 Linux 权限。必须给共享目录 777：

```bash
chmod -R 777 /mnt/sda1/media
```

否则会出现：

- 能挂载但打开是空的
- 能看见文件夹但点进去无权限
- 能读不能写

> **说明**：`-R` 递归改掉目录内所有文件和子目录，否则新放进去的文件可能还是没权限。以后往目录里拷了新文件，必要时再补一次 `chmod`。

### 3.4 启动服务

```bash
/etc/init.d/nfsd start
/etc/init.d/nfsd enable
```

验证共享是否发布成功：

```bash
showmount -e localhost
```

### 3.5 客户端挂载

Windows：先启用 NFS 客户端功能（**程序与功能 → 启用或关闭 Windows 功能 → NFS 服务**），然后：

```cmd
mount 192.168.31.190:/mnt/sda1/media Z:
```

---

## 四、避坑记录

| 问题 | 原因 | 解决 |
| --- | --- | --- |
| 共享 `/mnt/sda1` 客户端看到空文件夹 | OpenWrt 新版 NFS 界面需共享下一级目录 | 共享 `/mnt/sda1/media` 等子目录 |
| 挂载成功但无权限读写 | 目录 Linux 权限不足 | `chmod -R 777` 共享目录 |
| 旁路由和主路由 DHCP 冲突 | 旁路由未关闭 DHCP | 关闭 IPv4/IPv6 DHCP，只保留主路由分配 |
| DiskImage 写入失败 | 目标选成了分区而非物理磁盘 | 选 Physical Disk |

---

## 五、总结

升腾 C30 刷 Kwrt 做旁路由 + NFS 共享，整体流程不复杂，关键卡点有三个：

1. **写盘时选对物理磁盘**
2. **旁路由关闭 DHCP**，网关 / DNS 指向主路由 `192.168.31.1`
3. **NFS 共享目录给 777 权限**，路径建议指向子目录

这三点处理好，家用场景基本就能稳定运行。

---

## 附：分章节版本

- [01 - 刷机](docs/01-flash.md)
- [02 - 旁路由设置](docs/02-bypass-gateway.md)
- [03 - NFS 共享配置](docs/03-nfs-share.md)
- [04 - 避坑记录](docs/04-pitfalls.md)

配置文件片段见 [`config/`](config/) 目录。
