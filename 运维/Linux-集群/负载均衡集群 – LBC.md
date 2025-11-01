#  1 LVS 相关原理
- 1.1 LVS 的组成
&ensp;IPVS：运行在内核空间<br>
&ensp;IPVSADM：运行在用户空间，管理集群服务的命令行工具<br>
- 1.2 LVS 的原理
&ensp;根据用户请求的套接字判断，分流至真实服务器的工作模块<br>
- 1.3 LVS 工作方式
 <img width="819" height="400" alt="Linux：集群_3" src="https://github.com/user-attachments/assets/b931e5a8-6d8e-476b-8ac4-b07e45b68e75" /><br>
- 1.4 LVS – DR 模式
 <img width="827" height="438" alt="Linux：集群_4" src="https://github.com/user-attachments/assets/40d67951-f2ca-456e-8733-9e71004d0935" /><br>
&ensp;工作原理<br>
&ensp;&ensp;用户请求通过交换机被转发到负载调度器，负载调度器把用户请求分发给真实服务器，真实服务器收到用户请求后将自己伪装成负载调度器的 IP 地址再给用户返回请求<br>
&ensp;模式特点<br>
&ensp;&ensp;集群节点必须在一个网络中<br>
&ensp;&ensp;真实服务器网关指向路由器<br>
&ensp;&ensp;RIP 既可以是私有地址也可以是公网地址<br>
&ensp;&ensp;负载调度器只负责入栈请求<br>
&ensp;&ensp;大大减轻负载调度器压力，支持更多的服务器节点<br>

# 2 LVS – NAT 模式
 <img width="617" height="275" alt="Linux：集群_5" src="https://github.com/user-attachments/assets/cc4fee78-619e-4e5b-8c87-55c53dc63d94" /><br>
## 2.1 工作原理
&ensp;用户请求通过负载调度器 ，然后负载调度器将用户请求封装后分发给真实服务器，然后真实服务器处理之后封装之后传给负载调度器，然后负载调度器将真实服务器的请求更换成负载调度器的包头发给用户。<br>
## 2.2 模式特点
&ensp;集群节点必须再一个网络中<br>
&ensp;真实服务器必须将网关指向负载调度器<br>
&ensp;RIP 通常是私有 IP，仅用于各个集群节点通信<br>
&ensp;负载调度器必须位于客户端和真实服务器之间，充当网关<br>
&ensp;支持端口映射<br>
&ensp;负载调度器操作系统必须是 Linux，真实服务器可以使用任意系统<br>

# 3 LVS – TUN 模式
 <img width="689" height="316" alt="Linux：集群_6" src="https://github.com/user-attachments/assets/3cf284c6-ee6b-474f-96b3-a141bae0c348" /><br>
## 3.1 工作原理
&ensp;用户请求发给北京负载调度器，负载调度器将用户需求二次封装发给上海或者广州的真实服务器，真实服务器将负载调度器的包解封，然后再给用户返回请求<br>
## 3.2 模式特点
&ensp;集群节点不必位于同一个物理网络但必须拥有公网 IP 或可以被路由<br>
&ensp;&ensp;真实服务器不能将网关指向负载调度器<br>
&ensp;RIP 必须是公网地址<br>
&ensp;&ensp;不支持端口映射<br>
&ensp;&ensp;发送方和接收方必须支持隧道功能<br>

# 4 LVS – DR 搭建
<img width="609" height="400" alt="Linux：集群_7" src="https://github.com/user-attachments/assets/efe4d009-7aeb-4ded-9357-08f5fd6a1ff1" /><br>
## 4.1 环境准备
<img width="1147" height="543" alt="Linux：集群_8" src="https://github.com/user-attachments/assets/4f3c5d8d-87ae-42ef-91bd-0a254228cba3" /><br>
<img width="774" height="439" alt="Linux：集群_9" src="https://github.com/user-attachments/assets/f6d7ce88-5cd0-43eb-97c3-e0f62e5c15e0" /><br>
&ensp;三台服务器<br>
&ensp;&ensp;注意：每台机器都要有两块网卡<br>
&ensp;&ensp;负载调度器 本机IP1：10.10.10.11 本机IP2：10.10.10.100<br>
&ensp;&ensp;真实服务器1 本机IP ：10.10.10.12 伪装地址：10.10.10.100<br>
&ensp;&ensp;真实服务器2 本机IP ：10.10.10.13 伪装地址：10.10.10.100<br>
## 4.2 构建步骤<br>
&ensp;负载调度器<br>
&ensp;&ensp;service NetworkManager stop # 关闭网卡守护进程<br>
&ensp;&ensp;chkconfig NetworkManager off<br>
&ensp;&ensp;yum -y install gcc gcc-c++ lrzsz<br>
&ensp;&ensp;service iptables stop<br>
&ensp;&ensp;chkconfig iptables off<br>
&ensp;&ensp;vim /etc/selinux/confi # 关闭 selinux 但是需要重启<br>
&ensp;&ensp;把SELINUX=enforcing改成SELINUX=disabled<br>
&ensp;&ensp;setenforce 0 # 临时关闭 selinux<br>
&ensp;&ensp;cd /etc/sysconfig/network-scripts/<br>
&ensp;&ensp;vim ifcfg-eth0<br>

```bash
将 ONBOOT=no 改为 ONBOOT=yes
将 BOOTPROTO=dhcp 改为 BOOTPROTO=static
添加
IPADDR=10.10.10.11
NETMASK=255.255.255.0
```
&ensp;&ensp;cp -a ifcfg-eth0 ifcfg-eth0:0 # 拷贝 eth0  网卡子接口充当集群入口接口<br>
&ensp;&ensp;vim ifcfg-eth0:0 #只留下<br>

```bash
DEVICE=eth0:0
ONBOOT=yes
BOOTPROTO=static
IPADDR=10.10.10.100（虚拟IP）
NETMASK=255.255.255.0
ifup eth0:0
```

&ensp;&ensp;service network restart<br>
&ensp;&ensp;vim /etc/sysctl.conf # 关闭网卡重定向功能（修改 ARP 响应级别和通告行为）<br>

```bash
添加
# LVS - ARP （注释）
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.eth0.send_redirects = 0
```

&ensp;&ensp;sysctl -p<br>
&ensp;&ensp;yum -y install ipvsadm # 安装 ipvsadm 命令行工具<br>
&ensp;&ensp;modprobe ip_vs # 重载 ipvs 模块<br>
&ensp;&ensp;service ipvsadm start<br>
&ensp;&ensp;chkconfig ipvsadm on<br>
&ensp;&ensp;ipvsadm -Ln （查看集群节点）<br>
&ensp;&ensp;下面的操作需要配置好（真实服务器1） 和（真实服务器2）<br>
&ensp;&ensp;ipvsadm -A -t 10.10.10.100（虚拟IP）:80 -s rr<br>
&ensp;&ensp;ipvsadm -a -t 10.10.10.100（虚拟IP）:80 -r 10.10.10.12（真实服务器1IP）:80 -g<br>
&ensp;&ensp;ipvsadm -a -t 10.10.10.100（虚拟IP）:80 -r 10.10.10.13（真实服务器2IP）:80 -g<br>
&ensp;&ensp;service ipvsadm save  # 保存 ipvs 集群内容至文件，进行持久化存储<br>
&ensp;&ensp;chkconfig ipvsadm on  # 设置为开机自启<br>
&ensp;&ensp;ipvsadm -Ln –stats # 查看集群节点数据分发情况<br>

&ensp;真实服务器<br>
&ensp;&ensp;# 真实服务器 1 和真实服务器 2 都按照这个顺序搭建，不同的地方会提示<br>
&ensp;&ensp;service iptables stop<br>
&ensp;&ensp;chkconfig iptables off<br>
&ensp;&ensp;vim /etc/selinux/config # 关闭 selinux 但是需要重启<br>
&ensp;&ensp;把SELINUX=enforcing改成SELINUX=disabled<br>
&ensp;&ensp;setenforce 0 # 临时关闭 selinux<br>
&ensp;&ensp;service NetworkManager stop # 关闭网卡守护进程<br>
&ensp;&ensp;chkconfig NetworkManager off<br>
&ensp;&ensp;cd /etc/sysconfig/network-scripts/<br>
&ensp;&ensp;vim ifcfg-eth0<br>

```bash
将 ONBOOT=no 改为 ONBOOT=yes
将 BOOTPROTO=dhcp 改为 BOOTPROTO=static
添加
IPADDR=10.10.10.12
#真实服务器2 改为 IPADDR=10.10.10.13
NETMASK=255.255.255.0
```

&ensp;&ensp;cp -a ifcfg-lo ifcfg-lo:0 # 拷贝回环网卡子接口<br>
&ensp;&ensp;vim ifcfg-lo:0<br>

```bash
修改 DEVICE= 为 DEVICE=lo:0
修改 IPADDR= 为 IPADDR=10.10.10.100（虚拟IP）
修改 NETMASK= 为NETMASK=255.255.255.255
```
&ensp;&ensp;vim /etc/sysctl.conf # 关闭对应 ARP 响应及公告功能<br>
&ensp;&ensp;添加<br>

```bash
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.default.arp_ignore = 1
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
```
&ensp;&ensp;sysctl -p<br>
&ensp;&ensp;ifup lo:0<br>
&ensp;&ensp;route add -host 10.10.10.100（虚拟IP） dev lo:0 # 添加路由记录，当访问虚拟 IP 交给 lo:0 网卡接受<br>
&ensp;&ensp;echo “route add -host 10.10.10.100 dev lo:0” >> /etc/rc.local # 设置开机执行 route add -host 10.10.10.100 dev lo:0 这条命令<br>
&ensp;&ensp;service httpd start 或者 service nginx start # 开启网页服务<br>


# 5 LVS – NET 搭建
 <img width="695" height="418" alt="Linux：集群_10" src="https://github.com/user-attachments/assets/144b7291-91f0-443a-b541-f3ad44fe88c6" /><br>
 <img width="872" height="452" alt="Linux：集群_11" src="https://github.com/user-attachments/assets/cf324f76-c7e6-4737-b336-b269b3cb07c5" /><br>
## 5.1 环境搭建
- 三台服务器
- 注意：负载调度器必须双网卡
- 负载调度器外网 IP ：20.20.20.11 内网 IP ：10.10.10.11
- 真实服务器1内网 IP ：10.10.10.12
- 真实服务器2内网 IP ：10.10.10.13
## 5.2 构建步骤
### 5.2.1 负载调度器
- service NetworkManager stop # 关闭网卡守护进程
- chkconfig NetworkManager off
- vim /etc/selinux/config # 关闭 selinux，但是需要重启
- 把SELINUX=enforcing改成SELINUX=disabled
- setenforce 0 # 临时关闭 selinux
- yum -y install ipvsadm # 安装 ipvsadm 命令行工具
- modprobe ip_vs # 重载 ipvs 模块
