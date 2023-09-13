# intel_n5105_bugfix

# 问题现象

* VM 系统僵死（Freeze）
* 重启（reboot）
* 崩溃（Crash）

# 历史方案

## 方案 1

1. 升级 linux 5.19 内核

升级内核略过。

2. 确保 BIOS 配置：关闭 BIOS 的 c-stat，再将 pve 的 cpu 调度器设置成 powersave

```shell
cat /proc/cmdline
initrd=\EFI\proxmox\6.1.10-1-pve\initrd.img-6.1.10-1-pve root=ZFS=rpool/ROOT/pve-1 boot=zfs quiet intel_idle.max_cstate=1 intel_iommu=on iommu=pt mitigations=off i915.enable_guc=2 initcall_blacklist=sysfb_init nvme_core.default_ps_max_latency_us=14900
```

3. 重点讲下安装 intel 微代码 ``0x24000024`` 版本：

```shell
apt update
apt install intel-microcode
reboot
```

重启后用之前的命令确认intel-microcode版本是 ``0x24000024``

```shell
[Thu Mar 30 14:22:27 2023] microcode: microcode updated early to revision 0x24000024, date = 2022-09-02
[Thu Mar 30 14:22:28 2023] microcode: sig=0x906c0, pf=0x1, revision=0x24000024
```

## 方案 2

刷写 BIOS

## 方案 3

## 参考资料

[外部 pve 论坛](https://forum.proxmox.com/threads/vm-freezes-irregularly.111494/page-31)

[How to Install the Latest Microcode on Proxmox VE](https://cyrusyip.org/en/post/2023/01/31/install-microcode-on-proxmox/)

[chiphell 跟踪帖](https://www.chiphell.com/forum.php?mod=viewthread&tid=2446440&extra=&ordertype=1&page=1&mobile=no)

[intel microcode repo "Pentium N6000/N6005, Celeron N4500/N4505/N5100/N5105"](https://github.com/intel/Intel-Linux-Processor-Microcode-Data-Files/releases)

[PVE 系统调整](https://gitee.com/callmer/pve_toss_notes/blob/master/03.PVE%E7%B3%BB%E7%BB%9F%E8%B0%83%E6%95%B4.md)

