# intel_n5105_bugfix

# 问题现象

* VM 系统僵死（Freeze）
* 重启（reboot）
* 崩溃（Crash）

# 历史方案

## 推荐方案

1. 更新 BIOS
1.1 U盘格式化成 FAT32格式  
1.2 找到本仓库的 EFI 文件夹，将解压的 EFI 文件夹整个放进去，切记刷之前拔掉硬盘  
1.3 通电按 F11 ，选择UEFI SHELL ....回车后，输入FS0: 回车   再输入 ``cd EFI`` 回车后 按 1 回车进入刷机  
开始刷机后要让他刷完才能拔电，时间有点长，需要耐心等待。  
1.4 配置 BIOS：关闭 BIOS 的 c-stat，再将 pve 的 cpu 调度器设置成 powersave  
  
   N5105 driver: intel_pstate

   安装：cpufrequtils
   
   将调度器设置成 powersave

```shell
cat /proc/cmdline
initrd=\EFI\proxmox\6.1.10-1-pve\initrd.img-6.1.10-1-pve root=ZFS=rpool/ROOT/pve-1 boot=zfs quiet intel_idle.max_cstate=1 intel_iommu=on iommu=pt mitigations=off i915.enable_guc=2 initcall_blacklist=sysfb_init nvme_core.default_ps_max_latency_us=14900
```
2. 安装并将内核锁死在 ``6.1.10-1-pve`` 版本内核：  
建议后续不要随意升级 Linux 内核，避免 microcode 不生效。  
2.1 安装 6.1 版本内核，具体版本号是 ``6.1.10-1-pve``  
apt install pve-kernel-6.1   
2.2 告知 proxmox 锁定内核版本   
proxmox-boot-tool kernel pin 6.1.10-1-pve  
2.3 告知 apt 锁定内核软件包，不要更新它  
apt-mark hold pve-kernel-6.1.10-1-pve linux-image-6.1.10-1-pve linux-headers-6.1.10-1-pve  
2.4 卸载不用内核版本  
dpkg --purge --force-remove-essential pve-kernel-5.15.104-1-pve  
把不用的内核删除完成后，更新引导菜单  
update-grub  
查看启动菜单  
cat /boot/grub/grub.cfg |grep menuentry  
2.5 安装 intel 微代码 ``0x24000024`` 版本：  
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
至此大功告成。
## 参考资料

[外部 pve 论坛](https://forum.proxmox.com/threads/vm-freezes-irregularly.111494/page-31)

[How to Install the Latest Microcode on Proxmox VE](https://cyrusyip.org/en/post/2023/01/31/install-microcode-on-proxmox/)

[chiphell 跟踪帖](https://www.chiphell.com/forum.php?mod=viewthread&tid=2446440&extra=&ordertype=1&page=1&mobile=no)

[intel microcode repo "Pentium N6000/N6005, Celeron N4500/N4505/N5100/N5105"](https://github.com/intel/Intel-Linux-Processor-Microcode-Data-Files/releases)

[PVE 系统调整](https://gitee.com/callmer/pve_toss_notes/blob/master/03.PVE%E7%B3%BB%E7%BB%9F%E8%B0%83%E6%95%B4.md)

