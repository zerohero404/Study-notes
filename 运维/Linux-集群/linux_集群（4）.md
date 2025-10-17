# Linux：集群（四）

## 01. SELinux

### 1.1 SELinux 前世今生

Linux 安全性与 Windows 在不开启防御措施时一致，为 C2 级别。  
**创造者**：美国国家安全局（NSA，National Security Agency）
<img width="470" height="283" alt="Linux：集群136" src="https://github.com/user-attachments/assets/262eb3fa-f8b8-453a-a7c9-73a2fed84233" />

**Selinux 在 Linux 中的地位变化**：

| 版本 | 状态 |
|------|------|
| 2.2  | 需要手动加载的一个外部模块 |
| 2.4  | 直接写到内核的一个模块 |
| 2.6  | 成为 Linux 发行版内核的一部分 |

**特性**：

- 提高了 Linux 系统内部的安全等级
- 对进程和用户只赋予最小权限
- 防止权限升级，即使受到攻击，进程或者用户权限被夺去，也不会对整个系统造成重大影响

### 1.2 安全上下文

**定义**：所有操作系统访问控制都是以关联的客体和主体的某种类型的访问控制属性为基础。在 SELinux 中，这些访问控制属性叫做安全上下文。

- **主体**：进程
- **客体**：文件、进程间通讯通道、套接字、网络主机等
- **组成**：用户、角色和类型标识符

**格式**：
```
用户:角色:类型
```

#### 相关配置命令

```bash
chcon [-R] [-t type] [-u user] [-r role] 文件
# 参数说明：
# -R ：连同目录下的子目录也同时修改
# -t ：指定安全上下文类型
# -u ：指定用户身份，例如 system_u
# -r ：指定角色，例如 system_r
```

```bash
restorecon [-Rv] 文件或目录
# 还原成原有的 SELinux type
# -R ：连同子目录一起修改
# -v ：显示过程
```

#### SELinux 布尔值

- **定义**：布尔值相当于开关，精确控制 SELinux 对某个服务的某个选项的保护，例如 samba 服务。

**命令**：
```bash
getsebool -a       # 列出系统中可用的 SELinux 布尔值
setsebool          # 改变 SELinux 布尔值
```

**示例**：
```bash
setsebool -p samba_enable_home_dirs=1   # 开启家目录访问控制
```

---

## 02. 集群装机

### 2.1 PXE

#### 2.1.1 PXE 原理

**定义**：

PXE (Pre-boot Execution Environment) 是由 Intel 设计的协议，它可以使计算机通过网络启动。  
- **PXE client**：在网卡 ROM 中  
- **工作原理**：开机时，BIOS 调用 PXE client，将远端操作系统下载到本地运行  

**注意事项**：

- 查看网卡是否支持 PXE
- TFTP 服务器使用 UDP 协议进行文件传输

#### 2.1.2 PXE 服务构建

**注意事项**：

- 虚拟环境中关闭自带 DHCP 功能
- 安装机器与 PXE 服务器在同一网段

**安装步骤**：

```bash
yum -y install vsftpd dhcp tftp syslinux tftp-server
```

1. 拷贝系统镜像文件到 FTP 服务器目录
```bash
cd /var/ftp/pub
mkdir dvd
chown ftp:ftp dvd
cp -rf /mnt/cdrom/* dvd/
```

2. 配置 DHCP 服务
```bash
vim /etc/dhcp/dhcpd.conf
# 添加：
subnet 10.10.10.0 netmask 255.255.255.0 {
    range 10.10.10.100 10.10.10.200;
    option routers 10.10.10.11;
    next-server 10.10.10.11;
    filename "pxelinux.0";
}
```

3. 开启 TFTP 服务
```bash
vim /etc/xinetd.d/tftp
# 将 disable 从 yes 修改为 no
# server_args 修改为 -s /tftpboot
```

4. 创建相关目录并拷贝文件
```bash
mkdir -p /tftpboot/pxelinux.cfg
cp /var/ftp/pub/dvd/isolinux/isolinux.cfg /tftpboot/pxelinux.cfg/default
cp /usr/share/syslinux/pxelinux.0 /tftpboot/
chmod 644 /tftpboot/pxelinux.cfg/default
cp /var/ftp/pub/dvd/isolinux/* /tftpboot/
```

5. 开启相关服务并设置为自动启动
```bash
service dhcpd restart
chkconfig dhcpd on
service xinetd restart
chkconfig xinetd on
service vsftpd restart
chkconfig vsftpd on
```

6. 配置 Kickstart 无人值守安装

```bash
yum -y install system-config-kickstart
system-config-kickstart
```

- 配置 root 密码、FTP 信息、分区信息、SELinux 和防火墙
- 保存生成的 `ks.cfg` 文件到 `/var/ftp/`

7. 修改 `/tftpboot/pxelinux.cfg/default`：
```bash
default linux
label linux
menu label ^Install or upgrade an existing system
menu default
kernel vmlinuz
append initrd=initrd.img ks=ftp://10.10.10.11/ks.cfg
```

此时，服务器开机即可自动安装系统。

---

### 2.2 Cobbler

#### 2.2.1 Cobbler 原理

- Cobbler 是红帽公司基于 PXE 的装机服务，可选择安装多种操作系统
- **PXE**：适合单一系统环境，无需人工干预
- **Cobbler**：适合多系统环境，开机需选择系统版本

**安装流程（CentOS 7）**：

```bash
yum install -y epel-release
yum install -y cobbler cobbler-web pykickstart debmirror
systemctl restart httpd
systemctl enable httpd
systemctl restart cobblerd
netstat -an | grep 25151
vim /etc/cobbler/settings  # 修改 server 与 next_server
cobbler get-loaders
systemctl enable rsyncd
systemctl start rsyncd
vim /etc/debmirror.conf  # 注释掉 @dists="sid"; 和 @arches="i386";
openssl passwd -1 -salt $(openssl rand -hex 4)  # 生成加密密码
vim /etc/cobbler/settings  # 设置 default_password_crypted
yum install cman fence-agents
yum -y install xinetd
systemctl start xinetd
systemctl enable xinetd
systemctl start tftp
systemctl enable tftp
systemctl restart cobblerd
cobbler sync
cobbler check
yum install -y dhcp
vim /etc/dhcp/dhcpd.conf
systemctl restart dhcpd
systemctl enable dhcpd
cobbler import --name="centos6-x86_64" --path=/media/
cp centos6-x86_64.cfg /var/lib/cobbler/kickstarts/
cobbler profile add --name=centos6-x86_64-basic --distro=centos6-x86_64 --kickstart=/var/lib/cobbler/kickstarts/centos6-x86_64.cfg
cobbler sync
```

#### 2.2.2 CentOS 6 和 7 KS 模板分享

**CentOS 6 KS 模板**：
```kickstart
#platform=x86, AMD64, 或 Intel EM64T
#version=DEVEL
firewall --disabled
install
url --url="http://10.10.10.17/cobbler/ks_mirror/centos6-x86_64/"
rootpw --iscrypted $default_password_crypted
auth --useshadow --passalgo=sha512
graphical
firstboot --disable
keyboard us
lang zh_CN
selinux --disabled
logging --level=info
reboot
timezone Africa/Abidjan
bootloader --location=mbr
zerombr
clearpart --all --initlabel
part /boot --fstype="ext4" --size=200
part swap --fstype="swap" --size=2048
part / --fstype="ext4" --grow --size=1

%post --interpreter=/bin/bash
%end

%packages
@base
@chinese-support
@core
...
%end
```

**CentOS 7 KS 模板**：
```kickstart
lang en_US
keyboard us
timezone Asia/Shanghai
rootpw --iscrypted $default_password_crypted
text
install
url --url="http://20.20.20.20/cobbler/ks_mirror/CentOS-7-openstack-x86_64/"
bootloader --location=mbr
zerombr
clearpart --all --initlabel
part /boot --fstype xfs --size 1024 --ondisk sda
part swap --size 4000 --ondisk sda
part / --fstype xfs --size 1 --grow --ondisk sda
auth --useshadow --enablemd5
$SNIPPET('network_config')
reboot
firewall --disabled
selinux --disabled
skipx

%pre
$SNIPPET('log_ks_pre')
$SNIPPET('kickstart_start')
$SNIPPET('pre_install_network_config')
$SNIPPET('pre_anamon')
%end

%packages
@base
@core
%end
```

