# 03 - NFS 共享配置

## 3.1 安装

通过 OpenWrt 软件商店（opkg）安装：

- `luci-app-nfs`
- 中文语言包（可选）

装完后 LuCI 菜单会出现 **NFS 服务器** 入口。

## 3.2 配置共享目录

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

## 3.3 目录权限（关键）

NFS 只负责发布目录，客户端能否读写取决于服务端目录的 Linux 权限。必须给共享目录 777：

```bash
chmod -R 777 /mnt/sda1/media
```

否则会出现：

- 能挂载但打开是空的
- 能看见文件夹但点进去无权限
- 能读不能写

> **说明**：`-R` 递归改掉目录内所有文件和子目录，否则新放进去的文件可能还是没权限。以后往目录里拷了新文件，必要时再补一次 `chmod`。

## 3.4 启动服务

```bash
/etc/init.d/nfsd start
/etc/init.d/nfsd enable
```

验证共享是否发布成功：

```bash
showmount -e localhost
```

## 3.5 客户端挂载

Windows：先启用 NFS 客户端功能（**程序与功能 → 启用或关闭 Windows 功能 → NFS 服务**），然后：

```cmd
mount 192.168.31.190:/mnt/sda1/media Z:
```

Linux 客户端：

```bash
mkdir -p /mnt/media
mount -t nfs 192.168.31.190:/mnt/sda1/media /mnt/media
```

> 配置文件片段见 [`../config/exports`](../config/exports)。

---

[返回主页](../README.md)
