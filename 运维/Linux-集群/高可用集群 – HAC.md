# 1. LVS+DR+Keepalived
## 1.1 Keepalived 相关说明
- 软件相关介绍<br>
&emsp;&emsp;案例环境专为LVS 和 HA 设计的一款健康检查工具，支持故障自动切换（Failover），支持节点健康状态检查（Health Checking）<br>
&emsp;&emsp;官方网站：http://www.keepalived.org/<br>
- 软件实现原理<br>
&emsp;&emsp;VRRP（Virtual Router Redundancy Protocol，虚拟路由冗余协议）<br>
&emsp;&emsp;一主 + 多备，共用同一个 IP 地址，但优先级不同<br>
 <img width="537" height="252" alt="Linux：集群_18" src="https://github.com/user-attachments/assets/04619cef-d17f-4898-a0dc-8fea1687272d" /><br>
## 1.2 Keepalived + LVS 高可用集群构建
 <img width="574" height="358" alt="Linux：集群_19" src="https://github.com/user-attachments/assets/a61f45d9-1358-4fe1-903a-7ad3a6197f75" /><br>
### 1.2.1 实验环境准备
- LVS-M<br>
&emsp;&emsp;eth0 10.10.10.11<br>
&emsp;&emsp;eth0:0 10.10.10.100<br>
- LVS-S<br>
&emsp;&emsp;eth0 10.10.10.12<br>
&emsp;&emsp;eth0:0 10.10.10.100<br>
- RS-1<br>
&emsp;&emsp;eth0 10.10.10.13<br>
&emsp;&emsp;lo:0 10.10.10.100<br>
- RS-2<br>
&emsp;&emsp;eth0 10.10.10.14<br>
&emsp;&emsp;lo:0 10.10.10.100<br>

### 1.2.2构建步骤<br>
- 首先把 LVS-M、RS-1、RS-2 搭建成 LVS+DR 集群<br>
- LVS-M 操作<br>
&emsp;&emsp;yum -y install gcc gcc-c++ lrzsz<br>
&emsp;&emsp;service NetworkManager stop<br>
&emsp;&emsp;chkconfig NetworkManager off<br>
&emsp;&emsp;service iptables stop<br>
&emsp;&emsp;chkconfig iptables off<br>
&emsp;&emsp;vim /etc/selinux/config # 关闭 selinux 但是需要重启<br>
&emsp;&emsp;把SELINUX=enforcing改成SELINUX=disabled<br>
&emsp;&emsp;setenforce 0 # 临时关闭 selinux<br>
&emsp;&emsp;cd /etc/sysconfig/network-scripts/<br>
&emsp;&emsp;vim ifcfg-eth0

```bash
将 ONBOOT=no 改为 ONBOOT=yes
将 BOOTPROTO=dhcp 改为 BOOTPROTO=static
添加
IPADDR=10.10.10.11
NETMASK=255.255.255.0
```

&emsp;&emsp;cp -a ifcfg-eth0 ifcfg-eth0:0 # 拷贝 eth0  网卡子接口充当集群入口接口<br>
&emsp;&emsp;vim ifcfg-eth0:0<br>
&emsp;&emsp;只留下:<br>

```bash
DEVICE=eth0:0
ONBOOT=yes
BOOTPROTO=static
IPADDR=10.10.10.100（虚拟IP）
NETMASK=255.255.255.0
```

&emsp;&emsp;ifup eth0:0<br>
&emsp;&emsp;service network restart<br>
&emsp;&emsp;vim /etc/sysctl.conf # 关闭网卡重定向功能（修改 ARP 响应级别和通告行为）<br>
&emsp;&emsp;添加<br>

```bash
# LVS – ARP （注释）
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.eth0.send_redirects = 0
```
&emsp;&emsp;sysctl -p<br>
&emsp;&emsp;yum -y install ipvsadm # 安装 ipvsadm 命令行工具<br>
&emsp;&emsp;modprobe ip_vs # 重载 ipvs 模块<br>
&emsp;&emsp;service ipvsadm start<br>
&emsp;&emsp;chkconfig ipvsadm on<br>
&emsp;&emsp;ipvsadm -Ln （查看集群节点）<br>
&emsp;&emsp;下面的操作需要配置好（真实服务器1） 和（真实服务器2）<br>

```bash
ipvsadm -A -t 10.10.10.100（虚拟IP）:80 -s rr
ipvsadm -a -t 10.10.10.100（虚拟IP）:80 -r 10.10.10.12（真实服务器1IP）:80 -g
ipvsadm -a -t 10.10.10.100（虚拟IP）:80 -r 10.10.10.13（真实服务器2IP）:80 -g
service ipvsadm save     
# 保存 ipvs 集群内容至文件，进行持久化存储
chkconfig ipvsadm on 
# 设置为开机自启
ipvsadm -Ln –stats
# 查看集群节点数据分发情况
```

- 真实服务器<br>
- 真实服务器 1 和真实服务器 2 都按照这个顺序搭建，不同的地方会提示<br>
- service iptables stop<br>
- chkconfig iptables off<br>
- vim /etc/selinux/config # 关闭 selinux 但是需要重启<br>

```bash
把SELINUX=enforcing改成SELINUX=disabled
```
- setenforce 0 # 临时关闭 selinux<br>
- service NetworkManager stop # 关闭网卡守护进程<br>
- chkconfig NetworkManager off<br>
- cd /etc/sysconfig/network-scripts/<br>
- cp -a ifcfg-lo ifcfg-lo:0 # 拷贝回环网卡子接口<br>
- vim ifcfg-lo:0<br>

```bash
修改 DEVICE= 为 DEVICE=lo:0
修改 IPADDR= 为 IPADDR=10.10.10.100（虚拟IP）
修改 NETMASK= 为NETMASK=255.255.255.255
```

- vim /etc/sysctl.conf # 关闭对应 ARP 响应及公告功能<br>
&emsp;&emsp;添加<br>

```bash
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2
net.ipv4.conf.default.arp_ignore = 1
net.ipv4.conf.default.arp_announce = 2
net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
```

- sysctl -p<br>
- ifup lo:0<br>
- route add -host 10.10.10.100（虚拟IP） dev lo:0 # 添加路由记录，当访问虚拟 IP 交给 lo:0 网卡接受<br>
- echo “route add -host 10.10.10.100 dev lo:0” >> /etc/rc.local # 设置开机执行 route add -host 10.10.10.100 dev lo:0 这条命令<br>
- service httpd start 或者 service nginx start # 开启网页服务<br>

- 搭建 LVS-M 和 LVS-S 主从<br>
- LVS-M<br>
- yum -y install keepalived<br>
- chkconfig keepalived on<br>
- vim /etc/keepalived/keepalived.conf # 修改 Keepalived 软件配置<br>

```bash
# 将 global_defs 标签内的选项只留下
global_defs {
router_id LVS-1（LVS-M主机名）
}
```

```bash
# 修改 vrrp_instance VI_1 标签
vrrp_instance VI_1 {
state MASTER       # 设置服务类型主/从（MASTER/SLAVE）
interface eth0       # 指定那块网卡用来监听
virtual_router_id 66       # 设置组号， 如果是一组就是相同的 ID 号， 一个主里面只能有一个主服务器和多个从服务器
priority 100   # 服务器优先级， 主服务器优先级高
advert_int 1   #  心跳时间， 检测对方存活
authenticetion {     # 存活验证密码
auth_type PASS
auth_pass 1111
｝
virtual_ipaddress {
10.10.10.100       #设置集群地址
}
}
```

```bash
修改 virtual_server 10.10.10.100（集群地址） 80（端口） {
delay_loop 6        # 健康检查间隔
lb_algorr       # 使用轮询调度算法
lb_kind DR       # DR 模式的群集
protocol TCP # 使用的协议
real_server 10.10.10.13（管理的网站节点） 80（端口） {
weight 1 # 权重， 优先级 在原文件基础上删除修改
TCP_CHECK {    # 状态检查方式
connect_port 80    # 检查的目标端口
connect_timeout 3       # 连接超时（秒）
nb_get_retry 3      # 重试次数
delay_before_retry 4           # 重试间隔（秒）
}
}
```


&emsp;&emsp;有多少个网站服务器就写多少个 real_server 标签，只需要把网站节点修改<br>
&emsp;&emsp;文件多余不要的全部删除<br>
<img width="389" height="775" alt="Linux：集群_20" src="https://github.com/user-attachments/assets/d18dcf3f-368c-4950-8d45-15792f8623f0" /><br>
- service keepalived start<br>
