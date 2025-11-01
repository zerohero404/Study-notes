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
- yum -y install gcc gcc-c++ lrzsz<br>
- service NetworkManager stop<br>
- chkconfig NetworkManager off<br>
- service iptables stop<br>
- chkconfig iptables off<br>
- vim /etc/selinux/config # 关闭 selinux 但是需要重启<br>
- 把SELINUX=enforcing改成SELINUX=disabled<br>
- setenforce 0 # 临时关闭 selinux<br>
- cd /etc/sysconfig/network-scripts/<br>
- vim ifcfg-eth0

```bash
将 ONBOOT=no 改为 ONBOOT=yes
将 BOOTPROTO=dhcp 改为 BOOTPROTO=static
添加
IPADDR=10.10.10.11
NETMASK=255.255.255.0
```

- cp -a ifcfg-eth0 ifcfg-eth0:0 # 拷贝 eth0  网卡子接口充当集群入口接口<br>
- vim ifcfg-eth0:0<br>
- 只留下:<br>

```bash
DEVICE=eth0:0
ONBOOT=yes
BOOTPROTO=static
IPADDR=10.10.10.100（虚拟IP）
NETMASK=255.255.255.0
```

- ifup eth0:0<br>
- service network restart<br>
- vim /etc/sysctl.conf # 关闭网卡重定向功能（修改 ARP 响应级别和通告行为）<br>
- 添加<br>

```bash
# LVS – ARP （注释）
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.eth0.send_redirects = 0
```
- sysctl -p<br>
- yum -y install ipvsadm # 安装 ipvsadm 命令行工具<br>
- modprobe ip_vs # 重载 ipvs 模块<br>
- service ipvsadm start<br>
- chkconfig ipvsadm on<br>
- ipvsadm -Ln （查看集群节点）<br>
- 下面的操作需要配置好（真实服务器1） 和（真实服务器2）<br>

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
- 添加<br>

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


- 有多少个网站服务器就写多少个 real_server 标签，只需要把网站节点修改<br>
- 文件多余不要的全部删除<br>
<img width="389" height="775" alt="Linux：集群_20" src="https://github.com/user-attachments/assets/d18dcf3f-368c-4950-8d45-15792f8623f0" /><br>
- service keepalived start<br>


- LVS-S<br>
- service NetworkManager stop<br>
- chkconfig NetworkManager off<br>
- service iptables stop<br>
- chkconfig iptables off<br>
- vim /etc/selinux/config # 关闭 selinux，但是需要重启<br>

```bash
把SELINUX=enforcing改成SELINUX=disabled
```
- setenforce 0 # 临时关闭 selinux<br>
- cd /etc/sysconfig/network-scripts/<br>
- vim ifcfg-eth0<br>
- cp -a ifcfg-eth0 ifcfg-eth0:0 # 拷贝 eth0  网卡子接口充当集群入口接口<br>
- vim ifcfg-eth0:0<br>
&emsp;&emsp;只留下:<br>

```bash
DEVICE=eth0:0
ONBOOT=yes
BOOTPROTO=static
IPADDR=10.10.10.100（虚拟IP）
NETMASK=255.255.255.0
```

- vim /etc/sysconfig/network-script/ifup-eth<br>
 <img width="652" height="66" alt="Linux：集群_21" src="https://github.com/user-attachments/assets/35c3a0bc-94c1-4172-a970-53550ecaf7e8" /><br>
&emsp;&emsp;# 将这几行注释掉，大约在256 行左右<br>

- ifup eth0:0<br>
- vim /etc/sysctl.conf # 关闭网卡重定向功能（修改 ARP 响应级别和通告行为）<br>
&emsp;&emsp;添加:<br>

```bash
# LVS - ARP （注释）
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.eth0.send_redirects = 0
```

- sysctl -p<br>
- yum -y install ipvsadm # 安装 ipvsadm 命令行工具<br>
- modprobe ip_vs # 重载 ipvs 模块<br>
- service ipvsadm start<br>
- chkconfig ipvsadm on<br>
- yum -y install keepalived<br>
- chkconfig keepalived on<br>
- 切换到 LVS-M 服务器将 LVS-M 的 keepalived 配置文件复制到 LVS-S 服务器上<br>

```bash
scp /etc/keepalived/keepalived.conf root@10.10.10.12（LVS-S 服务器 IP）:/etc/keepalived/keepalived.conf
```

- vim /etc/keepalived/keepalived.conf<br>
&emsp;&emsp;修改文件中的<br>

```bash
router_id LVS-1 修改为 router_id LVS-2
state MASTER     修改至 state SLAVE
priority 100 修改至 priority 47 # 一般建议与主服务器差值为 50
```

- ipvsadm -A -t 10.10.10.100（虚拟IP）:80 -s rr<br>
- ipvsadm -a -t 10.10.10.100（虚拟IP）:80 -r 10.10.10.12（真实服务器1IP）:80 -g<br>
- ipvsadm -a -t 10.10.10.100（虚拟IP）:80 -r 10.10.10.13（真实服务器2IP）:80 -g<br>
- service ipvsadm save   # 保存 ipvs 集群内容至文件，进行持久化存储<br>
- service keepalived start<br>

# 2. 多级负载
## 2.1 服务器拓扑图
 <img width="885" height="480" alt="Linux：集群_22" src="https://github.com/user-attachments/assets/c6fb9ed7-01c9-4c96-85bb-651bbbeabe2a" /><br>
## 2.2 服务器构建
### 2.2.1 网页服务器（三台都是如此设置）
- ervice iptables stop<br>
- chkconfig iptables off<br>
- vim /etc/selinux/config # 关闭 selinux 但是需要重启<br>
- setenforce 0 # 临时关闭 selinux<br>
- service NetworkManager stop<br>
- chkconfig NetworkManager off<br>
- service httpd start<br>
- echo “www.baidu.com-1” >> /var/www/html/index.html<br>
&emsp;&emsp;# 此步骤为了区分各个网页服务器，15 服务器写 www.baidu.com-2，16 服务器写 www.baidu.cn
### 2.2.2 两台 nginx 服务器
- yum -y install gcc gcc-c++ lrzsz<br>
- yum -y install gcc*<br>
- yum -y install pcre-devel openssl openssl-devel<br>
- 下载 nginx 源码包<br>
- wget http://nginx.org/download/nginx-1.19.4.tar.gz<br>
- tar -xf nginx-1.19.4.tar.gz<br>
- cd nginx-1.19.4<br>
- useradd -s /sbin/nologin -M nginx<br>
- ./configure –prefix=/usr/local/nginx –user=nginx –group=nginx<br>
- make && make instal<br>
- cd /usr/local/nginx/conf/<br>
- vim nginx.conf<br>
&emsp;&emsp;将 http 标签修改成这样:<br>
 <img width="580" height="719" alt="Linux：集群_23" src="https://github.com/user-attachments/assets/41635a6c-2cf2-4647-934d-c261697a33c2" /><br>
- ln /usr/local/nginx/sbin/* /usr/sbin/<br>
- nginx -t # 检查配置文件<br>
- /usr/local/nginx/sbin/nginx # 启动 nginx<br>
- service NetworkManager stop<br>
- chkconfig NetworkManager off<br>
- service NetworkManager stop<br>
- chkconfig NetworkManager off<br>
- cd /etc/sysconfig/network-scripts/<br>
- cp -a ifcfg-lo ifcfg-lo:0<br>
- vim ifcfg-lo:0<br>

```bash
修改 DEVICE= 为 DEVICE=lo:0
修改 IPADDR= 为 IPADDR=10.10.10.100（虚拟IP）
修改 NETMASK= 为NETMASK=255.255.255.255
```

- vim /etc/sysctl.conf<br>
&emsp;&emsp;添加:<br>

``bash
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

### 
