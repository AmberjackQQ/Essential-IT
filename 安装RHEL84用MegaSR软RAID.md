# 安装流程
1. 挂载 84.iso
2. 用iso启动，选择安装
3. 在安装目的地，确认没有MegSR软RAID提供的硬盘
 ![alt "example"](/assets/img/rhel5.png) 
4. 配置网络IP等,可在network & hostname GUI界面
 ![alt "example"](/assets/img/rhel3.png) 
 ![alt "example"](/assets/img/rhel4.png) 
5. 用ctrl+alt+F2切换到shell
6. 用scp 复制ESRT2_Linux_DRV_v1.61-18.02.2020.0827中RHEL8-4_RPM目录下的所有内容
7. 安装megasr 驱动
    a. rpm -i --hash --nodeps --noscripts --root=/tmp/DD kmod-megasr-RHEL8-4.el8.x86_64.rpm 
    b. cp -avRf /tmp/DD/lib/modules/* /lib/modules/$(uname -r)/updates/DD
    c. 检查/etc/modprobe.d/megasr.conf 中是否有blacklist isci, intel C600 attached serial scsi driver
    d. depmod -av
    e. 加载驱动 modprobe -av megasr
    f. 验证 lsmod | grep megasr 和 modinfo megasr
8. 用ctrl+alt+F6切换到图形安装界面
9. 进入安装目的地界面，选择扫描硬盘
 ![alt "example"](/assets/img/rhel2.png) 
10. 选择安装的硬盘，以及其他选项开始安装
 ![alt "example"](/assets/img/rhel1.png) 
11. 当安装结束，准备重启之前，用ctrl+alt+F2切换到shell
12. cp RHEL8-4_RPM目录下的所有内容到/mnt/sysroot/root
13. chroot /mnt/sysroot/
14. rpm -ivh kmod-megasr-RHEL8-4.el8.x86_64.rpm
15. 安装完成重启





