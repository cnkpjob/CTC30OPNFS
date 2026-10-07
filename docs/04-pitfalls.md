# 04 - 避坑记录

| 问题 | 原因 | 解决 |
| --- | --- | --- |
| 共享 `/mnt/sda1` 客户端看到空文件夹 | OpenWrt 新版 NFS 界面需共享下一级目录 | 共享 `/mnt/sda1/media` 等子目录 |
| 挂载成功但无权限读写 | 目录 Linux 权限不足 | `chmod -R 777` 共享目录 |
| 旁路由和主路由 DHCP 冲突 | 旁路由未关闭 DHCP | 关闭 IPv4/IPv6 DHCP，只保留主路由分配 |
| DiskImage 写入失败 | 目标选成了分区而非物理磁盘 | 选 Physical Disk |

## 快速排障顺序

1. **能进 LuCI 吗？** 进不去 → 检查 `/etc/config/network` 的静态 IP 是否在主路由网段内。
2. **上网正常吗？** 不正常 → 关闭旁路由 DHCP，确认网关 / DNS 指向主路由 `192.168.31.1`。
3. **`showmount -e localhost` 有输出吗？** 没有 → 检查 `nfsd` 是否已 start / enable。
4. **挂载后是空的？** → 共享路径改成下一级子目录（如 `/mnt/sda1/media`）。
5. **看得见但写不了？** → `chmod -R 777` 共享目录。

---

[返回主页](../README.md)
