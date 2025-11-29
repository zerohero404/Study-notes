# 1. Docker 网络通讯
- 通常情况下，Docker使用网桥（Bridge）与 NAT 的网络通信模式
  - docker 首先会在宿主机创建一个虚拟网桥 docker0,然后有多少个容器就会创建多少个vethx ，然后将 vethx 与相对应容器内部的网卡相连用于通信，各个容器之间的通信通过 docker0 相互通信，容器与外部网络通信通过 docker0 与 宿主机网卡通信<br>
<img width="730" height="339" alt="Linux：虚拟化36" src="https://github.com/user-attachments/assets/e6adf915-81a2-44e3-9daa-79649a7a2184" /><br>
- 容器访问外部网络
  - iptables -t nat -A POSTROUTING -s 172.17.0.0/16 -o docker0 -j MASQUERADE
- 外部网络访问容器
  - docker run -d -p 80:80 apache
    - 当你执行了上面的命令，容器内部自动执行了下面两条命令
      - iptables -t nat -A  PREROUTING -m addrtype –dst-type LOCAL -j DOCKER
      - iptables -t nat -A DOCKER ! -i docker0 -p tcp -m tcp –dport 80 -j DNAT –to-destination 172.17.0.2:80
# 2. Docker 网络模式修改
-  Docker 进程网络修改
  - -b, --bridge=“”
    - 指定 Docker 使用的网桥设备，默认情况下 Docker 会自动创建和使用 docker0 网桥设备，通过此参数可以使用已经存在的设备
  - --bip
    - 指定 Docker0 的 IP 和掩码，使用的标准的 CIDR 形式，如 10.10.10.10/24
  - --dns
    - 配置容器的 DNS，在启动 Docker 进程是添加，所有容器全部生效
- Docker 容器网络修改
  - --net：用于指定容器的网络通讯方式，有以下四个值
    - bridge：Docker 默认方式，网桥模式
    - none：容器没有网络栈
    - container：使用其它容器的网络栈，Docker 容器会加入其它容器的 network namespace
    - host：表示容器使用 Host（宿主机） 的网络，没有自己独立的网络栈。容器可以完全访问 Host（宿主机） 的网络，不安全
- 映射端口
  - -P/-p 选项
    - -p  :<容器端口>
      - 将制定的容器端口随机映射至宿主机中的一个端口
    - -p  <宿主机端口>:<容器端口>
      - 将制定的容器端口指定映射至宿主机中的一个端口
    - -p  <IP>: :<容器端口>
      - 将制定的容器端口随机映射至宿主机中的一个端口并且绑定 IP 地址
    - -p  <IP>:<宿主机端口>:<容器端口>
      - 将制定的容器端口指定映射至宿主机中的一个端口并且绑定 IP 地址
    - -P
      - 将所有的端口都映射宿主机上的端口
- 自定义 Docker0 网桥的网络地址（修改网桥配置文件）
  - vim /etc/docker/daemon.json
```bash
{
"bip": "192.168.1.5/24",
"fixed-cidr": "10.20.0.0/16",
"fixed-cidr-v6": "2001:db8::/64", "mtu": "1500",
"default-gateway": "10.20.1.1",
"default-gateway-v6": "2001:db8:abcd::89", "dns": ["10.20.1.2","10.20.1.3"]
}
```

# 3. Docker 网络隔离
- 隔离基础命令
  - docker network ls # 查看当前可用网络类型
  - docker network create -d 类型 网络空间名称
    - 类型分为
      - overlay network
      - bridge networ
- 隔离命令使用
  - docker network create -d bridge –subnet “172.26.0.0/16” –gateway “172.26.0.1” my-bridge-network # 创建一个叫做 my-bridge-network 网络空间
  - docker run -d –network=my-bridge-network –name test1   hub.c.163.com/public/centos:6.7-tools # 将镜像运行为容器，加入 my-bridge-network 网络空间
  - 这样不是同一网络空间的容器就无法通信，就完成了容器隔离
- Linux 桥接器进行主机间通讯
  - cd /etc/sysconfig/network-scripts
  - cp -a ifcfg-enss33 ifcfg-br0
  - vim ifcfg-eth0
    - 将 IP 地址和 NETMASK 删除
    - 添加 BRIDGE=br0
  - vim ifcfg-br0
    - 改为 IP 地址和 NETMASK
  - systemctl restart network
  - yum install -y git
  - git clone https://github.com/jpetazzo/pipework # 下载 pipework 脚本
  - cp pipework/pipework /usr/local/bin/
  - docker run -d --net=none --name=ff centos-6-x86 bash # 启动一个别名是 ff 的容器
  - pipework br0 ff 192.168.216.135/24 # 给 ff 容器赋予 IP 地址
  - 这样就可以通过访问 192.168.216.135 来访问这个容器




























