# 编写kickstart

对于Redhat系列的OS来说，已经有内置的kickstart脚本，不过这些默认的ks文件，可能无法满足用户定制化的需求，我们可以自己制定自己的ks文件。kickstart文件支持自定义分区、自定义安装包和自定义脚本等。

通常，一个ks 文件由三部分组成：

1. 选项指令段，用于自动应答图形界面安装时除包选择外的所有手动操作
2. package选择段，定义需要安装的第三方包。 从’%packages’引导该功能
3. 脚本执行段，该段可有可无，分为两种：
   - %pre  预安装脚本段，在安装系统之前就执行的脚本，该段很少使用，因为可    	   用的命令太少
   - %post 后安装脚本段，在系统安装完成后执行的脚本

在虚拟机OS安装完成以后，Oz将调用 libvirt 使用 KVM 启动虚拟机，然后通过 ssh 等远程连接方式连接到虚拟机完成ks 文件的第二部分和第三部分内容。

更详细的Kickstart 文件结构说明: https://www.cnblogs.com/f-ck-need-u/archive/2017/08/10/7342022.htmll

以下是试验用到的kickstart文件，用注释对相关字段进行说明。对于一个简单的实践，只需要定义ks 文件的第一部分即可。

```
install    # 安装系统
Text        # 文本模式安装

# System language
lang en_US.UTF-8     # 系统安装阶段的必选项 （*）
keyboard us           #（*）
# Network information
network  --bootproto=dhcp
network  --hostname=localhost.localdomain
firewall --enabled --service=ssh
firstboot --disable  # 禁用首次启动后的手动配置界面
auth --enableshadow --passalgo=sha512  #启用shadow （*）
rootpw --iscrypted thereisnopasswordanditslocked  #使用加密密码 (*)
selinux --disabled
services --disabled="kdump" --enabled= "network,sshd,rsyslog,chronyd, docker"
timezone --utc Asia/Shanghai

# Disk
ignoredisk --only-use=vda
bootloader --append="console=tty0" --location=mbr --timeout=1 --boot-drive=vda     #（*）
zerombr   #清楚磁盘mbr
clearpart --all --initlabel #清除所有分区信息、创建标签
# --grow：使用所有可用空间，即为其分配所有剩余空间
# --size 单位是MB
part / --fstype ext4 --size=5000 --grow  

#repo  设置repo, 指定其他yum源
repo --name "os" --baseurl="http://10.127.2.8/centos/7/os/x86_64/" --cost=100
repo --name "updates" --baseurl="http://10.127.2.8/centos/7/updates/x86_64/" --cost=100
repo --name "extras" --baseurl="http://10.127.2.8/centos/7/extras/x86_64/" --cost=100
repo --name "epel" --baseurl="http://10.127.2.8/epel/7/x86_64/" --cost=100
repo --name "docker-ce" --baseurl="http://10.127.2.8/docker-ce/linux/centos/7/x86_64/stable/" --cost=100
reboot   # 安装结束后重启

--log=/mnt/sysimage/root/postinstall_stage1.log

%packages
chrony                    # 需要安装的包
cloud-init
cloud-utils-growpart
dracut-config-generic
dracut-norescue
firewalld
grub2
kernel
rsync
tar
yum-utils
wget
docker-ce
nginx
-NetworkManager   # 最前面使用横杠表示取反，即不选择
-aic94xx-firmware
-alsa-firmware
-alsa-lib
-alsa-tools-firmware
-biosdevname
-iprutils
-ivtv-firmware
-iwl100-firmware
-iwl1000-firmware
-iwl105-firmware
-iwl135-firmware
-iwl2000-firmware
-iwl2030-firmware
-iwl3160-firmware
-iwl3945-firmware
-iwl4965-firmware
-iwl5000-firmware
-iwl5150-firmware
-iwl6000-firmware
-iwl6000g2a-firmware
-iwl6000g2b-firmware
-iwl6050-firmware
-iwl7260-firmware
-libertas-sd8686-firmware
-libertas-sd8787-firmware
-libertas-usb8388-firmware
-plymouth
%end


%post --erroronfail --log=/root/postinstall.log

# passwd related

passwd -d root
passwd -l root

# 通过cloud-init 命令在第一次开机强制修改密码
cat > /etc/cloud/cloud.cfg.d/change_user_password.cfg << "EOF"
users:
   - name: default
chpasswd:
   list: |
      centos:changeme  #第一次开机登录时用户名/密码
   expire: True
ssh_pwauth: True
EOF

cat > /etc/sysconfig/network << "EOF"
NETWORKING=yes
NOZEROCONF=yes
EOF

cat > /etc/sysconfig/network-scripts/ifcfg-eth0 << "EOF"
DEVICE="eth0"
BOOTPROTO="dhcp"
ONBOOT="yes"
TYPE="Ethernet"
USERCTL="yes"
PEERDNS="yes"
IPV6INIT="no"
PERSISTENT_DHCLIENT="1"
EOF
%end
```