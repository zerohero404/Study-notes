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
- vim /etc/sysconfig/network-scripts/ifcfg-eth1<br>
&ensp;&ensp;&ensp;&ensp;修改网卡信息如下<br>
 <img width="345" height="155" alt="Linux：集群_12" src="https://github.com/user-attachments/assets/c6aa6613-3a70-4b03-9ba5-f0070ace4b92" /><br>
- vim /etc/sysconfig/network-scripts/ifcfg-eth0<br>
&ensp;&ensp;&ensp;&ensp;修改网卡信息如下<br>
 <img width="356" height="151" alt="Linux：集群_13" src="https://github.com/user-attachments/assets/f89d5c13-dd92-472b-9794-80a754fb7963" /><br>
- service network restart
- vim etc/sysctl.conf # 开启路由转发功能<br>
&ensp;&ensp;&ensp;&ensp;在配置文件添加:net.ipv4.ip_forward=1
- sysctl -p
- iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o eth0 -j SNAT –to-source 20.20.20.11<br>
&ensp;&ensp;&ensp;&ensp;# 添加防火墙记录，当源地址是 10.10.10.0/24 （内网网段）并且出口网卡为 eth0 的时候进行 SNAT 转换，转换源地址为 20.20.20.11 （外网卡地址）
- iptables -t nat -L # 查看记录是否保存成功
- ipvsadm -A -t 20.20.20.11（外网卡地址）:80 -s rr
- ipvsadm -a -t 20.20.20.11（外网卡地址）:80 -r 10.10.10.12（真实服务器1地址）:80 -m
- ipvsadm -a -t 20.20.20.11（外网卡地址）:80 -r 10.10.10.13（真实服务器2地址）:80 -m
- ipvsadm -Ln # 查看集群
- service ipvsadm save # 保存集群信息
- service iptables save # 保存防火墙规则
- chkconfig ipvsadm on
### 5.2.2 真实服务器
- echo “GATEWAY=10.10.10.11（负载调度器内网IP地址）” >> /etc/sysconfig/network-scripts/ifcfg-eth0
- service httpd start 或者 service nginx start # 开启网页服务

# 6 负载均衡集群调度算法（策略）
## 6.1 静态调度算法
- 特点：只根据算法本身去调度，不考虑服务器本身
- RR 轮询：将每次用户的请求分配给后端的服务器，从第一台服务器开始到第N 台结束， 然后循环<br>
&ensp;&ensp;&ensp;&ensp;它均等的对待每一台真实服务器，而不管服务器实际的连接数和系统负载。权重值相同，权重值若为0则表示真实服务器不可用。<br>
 <img width="798" height="366" alt="Linux：集群_14" src="https://github.com/user-attachments/assets/12ac5f9b-a27a-4690-afcf-40972b0567d8" /><br>
- WRR 加权轮询：按照权重的比例实现在多台主机之间进行调度。<br>
 <img width="755" height="369" alt="Linux：集群_15" src="https://github.com/user-attachments/assets/5e0ea3d4-7d74-402b-a8d6-22ba08e045fe" /><br>
- SH（source hash）源地址散列：将同一个 IP 的用户请求，发送给同一个服务器<br>
&ensp;&ensp;&ensp;&ensp;当用户第一次请求时，负载调度器会根据轮询算法按顺序将请求转发到后端的真实服务器上，并会将用户请求的源IP+转发到后端的真实服务器对应关系以散列键=值的方式存放到一张散列表（哈希表）中，当用户再一次请求时，负载调度器会根据请求的源IP，匹配散列表中对应关系，将请求分发到后端的同一台真实服务器上。若后端的真实服务器超载或不可用，则会返回空。<br>
 <img width="804" height="432" alt="Linux：集群_16" src="https://github.com/user-attachments/assets/ae706979-d832-4fcb-a027-34ef8ffadcc9" /><br>
- DH（destination hash）目标地址散列：将同一个目标地址的用户请求发送给同一个真实服务器（提高缓存的命中率）<br>
&ensp;&ensp;&ensp;&ensp;这种调度算法一般用于后端真实服务器为缓存服务器的场景下，我们希望将所有请求可以一直都匹配到缓存，通过缓存服务器直接返回响应给用户，这种调度算法就基本失去了负载均衡的意义，基本上不会用。<br>
 <img width="747" height="468" alt="Linux：集群_17" src="https://github.com/user-attachments/assets/a703a5ec-d0e1-4aca-a454-29bed08e5848" /><br>
 
## 6.2 动态调度算法
- 特点：除了考虑算法本身，还要考虑服务器状态<br>
- LC（lest-connection）最少连接：将新的连接请求，分配给连接数最少的服务器 <br>
&ensp;&ensp;&ensp;&ensp;连接数=活动连接 × 256 + 非活动连接<br>
&ensp;&ensp;&ensp;&ensp;活动连接：正在传输数据的连接<br>
&ensp;&ensp;&ensp;&ensp;非活动连接：没有传输数据的连接和传输数据完成还未关闭的连接<br>
- WLC 加权最少连接：特殊的最少连接算法，权重越大承担的请求数越多 <br>   
&ensp;&ensp;&ensp;&ensp;连接数=（活动连接 × 256 +  非活动连接 ）/ 权重<br>
- SED 最短期望延迟：特殊的 WLC 算法（活动连接 + 1）× 256 / 权重<br>
- NQ 永不排队：特殊的 SED 算法，无需等待，如果有真实服务器的连接数等于 0 那就直接分配不需要运算<br>
- LBLC 特殊的 DH 算法：提高缓存命中率，又要考虑服务器性能的方案<br>
&ensp;&ensp;&ensp;&ensp;即将 DH 算法中对某一缓存服务器中的缓存文件请求过多，导致这台服务器性能不够，这时这个动态算法就会让缓存服务器集群的另外一台缓存服务器从 web 服务器下载这个缓存文件，然后分担这个缓存文件的请求压力，如果两台服务器也承担不了那就再从缓存服务器集群中再增加一台依次类推<br>
- LBLCR LBLC+缓存：尽可能提高负载均衡和缓存命中率的折中方案<br>
&ensp;&ensp;&ensp;&ensp;即当某一缓存服务器中的缓存文件请求过多，集群中的其他缓存服务器想分担请求压力的时候，并不是向 web 服务器下载这个缓存文件，而是向有这个缓存文件的缓存服务器下载，减少web服务器的压力<br>
## 6.3 持久连接
- 相关的集群命令<br>
&emsp;&emsp;ipvsadm -D -t 集群负载调度器IP:端口 # 删除一个集群<br>
&emsp;&emsp;ipvsadm -Ln –persistent-conn # 查询集群持久化连接时长<br>
&emsp;&emsp;ipvsadm -Ln -c # 当前集群的连接信息<br>
- 持久客户端连接<br>
&emsp;&emsp;定义：每客户端持久，将来自于同一个客户端的所有请求统统定向至此前选定的真实服务器；也就是只要 IP 相同，分配的服务器始终相同<br>
&emsp;&emsp;创建代码<br>
&emsp;&emsp;&emsp;&emsp;ipvsadm -A -t 172.16.0.8（负载调度器IP地址）:0 -s wlc -p 120（时间长度，单位秒） # 添加一个 tcp 负载集群，集群地址为 172.16.0.8 ， 算法为 wlc，持久化时间为 120s<br>
- 持久端口连接<br>
&emsp;&emsp;定义：每端口持久，将来自于同一个客户端对同一个服务(端口)的请求，始终定向至此前选定的真实服务器<br>
&emsp;&emsp;创建代码<br>
&emsp;&emsp;&emsp;&emsp;ipvsadm -A -t 172.16.0.8（负载调度器IP地址）:80 -s rr -p 120 （时间长度，单位秒） # 添加一个 tcp 负载集群，集群地址为 172.16.0.8:80 ， 算法为 wlc，持久化时间为 120s<br>
- 持久防火墙标记连接<br>
&emsp;&emsp;定义：将来自于同一客户端对指定服务(端口)的请求，始终定向至此选定的 RS；不过它可以将两个毫不相干的端口定义为一个集群服务<br>
&emsp;&emsp;创建代码<br>
&emsp;&emsp;&emsp;&emsp;iptables -t mangle -A PREROUTING -d 172.16.0.8 -p tcp –dport 80 -j MARK –set-mark 10 #添加一个防火墙规则，当目标地址为 172.16.0.8 并且 目标端口为 80 时给数据包打一个标记，设置 mark 值为 10<br>
&emsp;&emsp;&emsp;&emsp;iptables -t mangle -A PREROUTING -d 172.16.0.8 -p tcp –dport 443 -j MARK –set-mark 10 # 添加一个防火墙规则，当目标地址为 172.16.0.8 并且 目标端口为 443 时给数据包打一个标记，设置 mark 值为 10<br>
&emsp;&emsp;&emsp;&emsp;service iptables save   # 保存防火墙规则持久化生效<br>
&emsp;&emsp;&emsp;&emsp;ipvsadm -A -f 10 -s wlc -p 120       # 添加一个负载调度器，当 mark 值为 10 时进行负载均衡使用 wlc 算法，持久化生效时间为 120s<br>
