# 01. OpenStack 组件
- 选用 OpenStack Juno 版本，这个版本有以下的核心组件
  - Compute (Nova) – 计算服务
  - Image Service (Glance) – 镜像服务
  - Object Storage (Swift) – 对象存储
  - Block Storage (Cinder) – 块存储
  - Networking (Neutron) – 网络服务
  - Dashboard (Horizon) – 仪表盘
  - Identituy Service (Keystone) – 认证服务
  - Orchestration (Heat) – 编排
  - Telemetry (Ceilometer) – 监控
  - Database Service (Trove) – 数据库服务
  - Data Processing (Sahara) – 数据处理
 

# 02. OpenStack 组件搭建环境
- 操作系统
  - CentOs 7
  - 关闭防火墙和 SELinux
- 安装版本
  - Juno
- 安装模块
  - Compute (Nova) – 计算服务
  - Image Service (Glance) – 镜像服务
  - Block Storage (Cinder) – 块存储
  - Networking (Neutron) – 网络服务
  - Dashboard (Horizon) – 仪表盘
  - Identituy Service (Keystone) – 认证服务
  - Database Service (Trove) – 数据库服务
  - Data Processing (Sahara) – 数据处理
- 结构拓扑图
  - Contriller（控制节点）
  - Identituy Service (Keystone) – 认证服 务
  - Image Service (Glance) – 镜像服务
  - Database Service (Trove) – 数据库服务
  - Data Processing (Sahara) – 数据处理
  - Block（存储节点）
  - Block Storage (Cinder) – 块存储
  - Compute（实例节点）
  - Compute (Nova) – 计算服务
  - Dashboard (Horizon) – 仪表盘
  - Network（网络节点）
  - Networking (Neutron) – 网络服务<br>
  <img width="821" height="501" alt="Linux：虚拟化43" src="https://github.com/user-attachments/assets/3dca5087-3610-4085-b14a-e275a14505ae" /><br>
- 扩展
  - Contriller（控制节点）在生产环境中应该配置数据库集群
  - block（存储节点）可以在管理网络中添加多个
- 实验所需服务器最低配置
  - Contriller（控制节点）
    - 两核 CPU
    - 内存 1.5 GB 以上
    - 一块网卡
    - 100GB存储
  - Compute（实例节点）
    - MAX CPU
    - MAX 内存
    - 两块网卡
    - 100GB存储
  - Network（网络节点）
    - 两核 CPU
    - 内存 1.5 GB 以上
    - 三块网卡
    - 20GB存储
  - Block（存储节点）
    - 两核 CPU
    - 内存 1 GB 以上
    - 一块网卡
    - 一个 20 GB 存储用于存放系统文件
    - 多块存储用于块存储的底层存储

# 03. Identituy Service (Keystone) 组件

## 3.1 Identituy Service (Keystone) 说明
- Keystone 是 OpenStack Identituy Service 的项目名称，是负责身份管理和授权的组件
  - 主要功能：实现用户的身份认证，基于角色的权限管理，及 OpenStack 其他组件的访问地址和安全策略管理
  - Keystone 项目是给 OpenStack 的其他组件（nova，cinder，glance….）提供一个统一的验证方式

- 用户管理
  - Account 账户
  - Authentication 身份认证
  - Authorization 授权
- 服务目录管理
  - User（用户）：一个人、系统或服务在 OpenStack 中的数字表示。已经登陆的用户分配令牌环以访问资源。用户可以直接分配给特定的租户，就像隶属于每个组
  - Credentials（凭证）：用于确认用户身份的数据。例如：用户名和密码，用户名和 API key，或由认证服务提供的身份验证令牌
  - Authentication（验证）：确认用户身份的过程
  - Token（令牌）：一个用于访问 OpenStack API 和资源的字母数字字符串。一个临牌可以随时撤销，并且只在一段时间内生效
  - Trnant（租户）：一个组织或者故里资源的容器。租户可以组织或者隔离认证对象。根据服务运营的要求，一个租户可以映射到客户、账户、组织或项目
  - Service OpenStack （服务于服务的服务）：它提供了一个或者多个端点，供用户访问资源和执行操作。
  - Endpoint（端点）：一个用户可以通过网络对某个服务进行访问的地址，通常是一个 URL 地址
  - Role（角色）：包含特定用户权限和特权的定制化权限集合
  - Keystone Client（Keystone 命令行工具）：Keystone 的命令行工具。通过这个工具可以创建用户，服务和端点等。
- 服务目录管理通俗解释
  - 用户：个人姓名
  - 凭证：身份证
  - 验证：检查身份证
  - 令牌：官印
  - 租户：管理的辖区
  - 服务：辖区内要管理的部门
  - 端点：去往具体部门的路径
  - 角色：官位的高低

## 3.2 Identituy Service (Keystone) 组件之间的沟通
- 用户查询服务列表认证过程
  - 用户（User）拿着用户名（User）和 密码（Password）找 Keystone 验证
  - Keystone 验证用户名（User）和 密码（Password）过后，如果正确发 Token（令牌）给用户（User）
  - 用户（User）不知道服务列表，便将 Token（令牌）和服务列表查询请求发给 Keystone
  - Keystone 验证 Token（令牌）之后将 Tenant list（服务器列表）发送给用户（User）
  - <img width="688" height="342" alt="Linux：虚拟化44" src="https://github.com/user-attachments/assets/872b1229-b7c7-4d4b-853f-769c5b747a73" />
- 用户访问服务认证过程
  - 用户（User）拿着用户名（User）和 密码（Password）找 Keystone 验证
  - Keystone 验证用户名（User）和 密码（Password）过后，如果正确发 Token（令牌）和 Endpoint（端点） 给用户（User）
  - 用户（User）将 Token（令牌）和访问请求给 Endpoint（端点）
  - Endpoint（端点）将 用户（User）发过来的 Token（令牌）和访问请求发给 Keystone 验证
  - Keystone 验证过后将 Token（令牌）和访问请求发给 Nova（服务）
  - Nova（服务）接收请求后处理请求，然后返回给用户（User）
  - <img width="623" height="420" alt="Linux：虚拟化45" src="https://github.com/user-attachments/assets/6fb66cb5-7494-42db-ac30-9de3b18b2f00" />
- 组件之间的交互过程
  - 用户（User）发送验证信息给 Keystone，Keystone返回 Token（令牌）
  - 用户（User）发送 Token（令牌）和创建虚拟机请求给 Nova（计算服务）
  - Nova（计算服务）将用户（User）发送的 Token（令牌）返回给 Keystone 验证，Keystone 验证信息回复
  - Nova（计算服务）在 Keystone 返回信息之后，将 Token（令牌）和镜像请求发送给 Glance（镜像服务）
  - Glance（镜像服务）把 Nova（计算服务）发送的 Token（令牌）返回给 Keystone 验证，Keystone 验证信息回复
  - Glance（镜像服务）再将镜像发送给 Nova（计算服务）
  - Nova（计算服务）收到镜像后再将 Token（令牌）和网络请求信息发送给 Neutron（网络服务）
  - Neutron（网络服务）把 Nova（计算服务）发送的 Token（令牌）返回给 Keystone 验证，Keystone 验证信息回复
  - Neutron（网络服务）回复请求给 Nova（计算服务）
  - Nova（计算服务）回复 Neutron（网络服务）的回复
  - Neutron（网络服务）回复请求完成给 Nova（计算服务）
  - Nova（计算服务）回复请求完成给用户（User）
  - <img width="805" height="621" alt="Linux：虚拟化46" src="https://github.com/user-attachments/assets/f544c9ad-b1ee-48ff-8cae-b035b3d64b4c" />
- 用户-角色-服务交互
  - Trnant（租户）里面有各个不同的 User（用户）
  - 每个 User（用户）可以使用不同的 Role（角色）
  - 每个 Role（角色）有不同的权限
  - Role1 可以在 Compute（计算服务）中创建 VM 和绑定网络
  - Role2 可以在 Compute（计算服务）中绑定卷和在 Network（网络服务）中增加网络
  - Role3 可以在 Network（网络服务）增加端口和绑定
  - <img width="739" height="501" alt="Linux：虚拟化47" src="https://github.com/user-attachments/assets/5bc0da4a-d4c2-4038-a8b5-67f6dca4cb22" />

# 3.3 Identituy Service (Keystone)组件安装
- 在 controller 节点上操作
  - 关闭 NetworkManager
    - systemctl stop NetworkManager
    - systemctl disable NetworkManager
  - 关闭防火墙
    - systemctl stop firewalld
    - systemctl disable firewalld
  - 设置主机名
    - hostnamectl set-hostname controller.nice.com
  - 配置 Dns 服务器或者使用 hosts 进行主机名的 IP 对应
    - vim /etc/hosts
      - 添加
        - 192.168.222.5 controller.nice.com
        - 192.168.222.6 network.nice.com
        - 192.168.222.10 compute.nice.com
        - 192.168.222.20 block.nice.com
  - 配置时间同步服务器
    - yum -y install ntp
    - vim /etc/ntp.conf
      - 注释
        - server 0.centos.pool.ntp.org iburst 到 server 3.centos.pool.ntp.org iburst
      - 添加
        - server 127.127.1.0<br>
          fudge 127.127.1.0 stratum 10<br>
        - <img width="517" height="126" alt="Linux：虚拟化48" src="https://github.com/user-attachments/assets/d8625a9f-ecee-4088-98ef-1c3fb8b8eee9" />
      - 取消注释并且修改
        -restrict 192.168.222.0（OpenStack 管理网络网段） mask 255.255.255.0 nomodify notrap

- 安装 OpenStack 预备包
  - 安装 yum-plugin-priorities 包，防止高优先级软件被低优先级软件覆盖
    - yum -y install yum-plugin-priorities
  - 安装 epel 扩展 YUM 源
    - yum -y install http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-2.noarch.rpm
  - 安装 OpenStack YUM 源
    - yum -y install http://rdo.fedorapeople.org/openstack-juno/rdo-release-juno.rpm
  - 更新操作系统
    - yum -y upgrade
  - 安装 OpenStack-selinux 自动管理 Selinux
    - yum -y install openstack-selinux
    - 也可以关闭 selinux，不安装这个软件
  - 安装 mariadb 软件包
    - yum -y install mariadb mariadb-server MySQL-python
    - 编辑 /etc/my.cnf 设置绑定 IP，默认数据库引擎及默认字符集为 UTF-8
    - vim /etc/my.cnf
      - 在 [mysql] 标签下添加
          - bind-address = 192.168.222.5（controller 节点 IP）<br>
            default-storage-engine = innodb<br>
            innodb_file_per_table<br>
            collation-server = utf8_general_ci<br>
            init-connect = ‘SET NAMES utf8’<br>
            character-set-server = utf8<br>
  - 启动数据库并设置为开机自启动
    - systemctl enable mariadb
    - systemctl start mariadb
  - 初始化数据库脚本
    - mysql_secure_installation
  - 安装 Messaing Server 服务
    - Messaing Server 服务简介
      - 功能：协调操作和状态信息服务
      - 常用的消息代理软件
        - RabbitMQ（推荐）
        - Qpid
        - ZeroMQ
    - 安装 RabbitMQ 软件包
      - yum -y install rabbitmq-server
    - 启动服务并设置开机自启动
      - systemctl enable rabbitmq-server
      - systemctl start rabbitmq-server
    - rabbitmq 默认用户名和密码是 guest，可以通过下列命令修改
      - rabbitmqctl change_password guest new_password 然后输入新密码
    - 安装和配置认证服务
      - 登录mysql数据库
        - mysql -u root -p
        - 创建keystone数据库
          - MariaDB [(none)]>CREATE DATABASE keystone（库名）; # 创建keystone数据库用户，使其可以对keystone数据库有完全控制权限
          - MariaDB [(none)]>GRANT ALL PRIVILEGES ON keystone（库名）.* TO ‘keystone（用户）’@’localhost（可连接地址）’ IDENTIFIED BY ‘KEYSTONE_DBPASS（连接密码）’;
          - MariaDB [(none)]>GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'KEYSTONE_DBPASS'；
          - MariaDB [(none)]>GRANT ALL PRIVILEGES ON keystone（库名）.* TO ‘keystone（用户）’@’%（可连接地址，%代表所有地址都可以连接）’ IDENTIFIED BY ‘KEYSTONE_DBPASS（连接密码）’;
          - MariaDB [(none)]>GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'KEYSTONE_DBPASS'；
      - 生成一个随机值作为管理令牌在初始配置
        - openssl rand -hex 10:
        - 将这个字符串记住

















