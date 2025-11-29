# 1. Docker 相关
- Docker 由来
  - Docker 是 dotcloud 公司开源的一款产品 dotcloud 是 2010 年新成立的一家公司，主要基于 PAAS ( Platfrom as a Service ) 平台为开发者提供服务
  - 2013 年 10 月 dotcloud 公司改名为 Docker 股份有限公司
- Docker 历程
  - Docker 是基于 Linux Container 技术上开发出来的
  - Docker 是  PAAS   提供商 dotCloud 开源的一个基于 LXC 的高级容器引擎，源代码托管在 Github 上, 基于 go 语言并遵从 Apache2.0 协议开源
    - PAAS 与 POP、IAAS 的区别
    - <img width="657" height="391" alt="Linux：虚拟化30" src="https://github.com/user-attachments/assets/f4754c91-4b3a-4ca7-ba1f-eddc849fb92d" />
  - Docker 设想是交付运行环境如同海运，OS 如同一个货轮，每一个在 OS 基础上的软件都如同一个集装箱，用户可以通过标准化手段自由组装运行环境，同时集装箱的内容可  以由用户自定义，也可以由专业人员制造
- Docker 与传统虚拟化对比
- <img width="757" height="423" alt="Linux：虚拟化23" src="https://github.com/user-attachments/assets/b6009d61-1551-4503-bff4-06e5f7ea7c7e" />
- Docker 的构成
  - Docker 仓库：https://hub.docker.com
  - Docker 自身组件
  - Docker Client：Docker 的客户端
  - Docker Server：Docker daemon 的主要组成部分，接受用户通过 Docker Client 发出的请求，并按照相应的路由规则实现路由分发
  - Docker 镜像：Docker 镜像运行之后变成容器（docker run）
- Docker 组件间的协同方式
  - <img width="930" height="455" alt="Linux：虚拟化24" src="https://github.com/user-attachments/assets/32dcb501-590f-46f9-9b9b-0d677185bf5f" />
- Docker化应用存在的方式
  - 第一种
    - <img width="815" height="364" alt="Linux：虚拟化25" src="https://github.com/user-attachments/assets/dd18f59b-655e-4ca3-835a-8844d070886a" />
  - 第二种
    - <img width="804" height="366" alt="Linux：虚拟化26" src="https://github.com/user-attachments/assets/31022c9c-134c-4c32-9ba0-1773f40ef95b" />
  - 第三种
    - <img width="798" height="399" alt="Linux：虚拟化27" src="https://github.com/user-attachments/assets/a25f783c-4a6f-4f62-aa1c-30eb37fbed3a" />
  - 第四种
    - <img width="798" height="415" alt="Linux：虚拟化28" src="https://github.com/user-attachments/assets/738ff382-b1ee-4503-8c09-3105e5ec1e22" />
  - 第五种
    - <img width="850" height="380" alt="Linux：虚拟化29" src="https://github.com/user-attachments/assets/ce0bacb4-fa19-40c0-a4a9-5d72a20b9420" />

# 02. Docker 安装
- Docker 建议使用 Centos 7 以上系统，Centos 6 系统兼容性不好
- Docker 不支持 Centos 7 的 firewalld，支持 iptables
- systemctl stop firewalld
- systemctl disable firewalld
- 关闭 selinux
- yum -y install iptables-services # 安装 iptables
- systemctl start iptables
- systemctl enable iptables
- iptables -F # 清空防火墙规则
- Docker 安装有三种方法
  - Script （脚本安装）
    - yum update # 升级操作系统
    - curl -sSL https://get.docker.com/ | sh # 从 docker 官网获取脚本，并且运行脚本安装 docker。生产环境不建议使用脚本安装
    - systemctl start docker
    - systemctl enable docker
    - docker run hello-world # 运行 docker “hello-world”容器检测 docker 是否安装成功
    - systemctl status docker # 也可以用这个命令检测 docker 是否安装，成功运行
  - Yum Install
    - yum install -y yum-utils # 安装依赖包
    - yum-config-manager \ --add-repo \ https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo # 安装 docker yum 源
    - sed -i ‘s/download.docker.com/mirrors.aliyun.com\/docker-ce/g’ /etc/yum.repos.d/docker-ce.repo # 将官方源更换为阿里源
    - yum install docker-ce docker-ce-cli containerd.io
    - systemctl start docker
    - systemctl enable docker
    - docker run hello-world # 运行 docker “hello-world”容器检测 docker 是否安装成功
    - systemctl status docker # 也可以用这个命令检测 docker 是否安装，成功运行
  - Rpm install（RPM 包安装）
    - 去 https://download.docker.com/linux/centos/7/x86_64/stable/Packages/ 下载下面两个安装包
    - docker-ce-17.03.0.ce-1.el7.centos.x86_64.rpm
    - docker-ce-selinux-17.03.0.ce-1.el7.centos.noarch.rpm
    - 将上面两个安装包上传
    - yum update # 更新系统
    - reboot # 重启系统
    - 重启完成之后去两个软件包上传的文件夹 yum -y install * 安装 docker
    - systemctl start docker
    - systemctl enable docker
    - docker run hello-world # 运行 docker “hello-world”容器检测 docker 是否安装成功
    - systemctl status docker # 也可以用这个命令检测 docker 是否安装，成功运行

# 03. Docker 加速配置
- 阿里云 Docker 官网加速网址：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors?spm=5176.12901015.0.i12901015.17f7525c3JNsmE
- 在阿里云容器镜像服务-镜像中心-镜像加速器获取你的加速器地址
- cp /lib/systemd/system/docker.service /etc/systemd/system/docker.service
- chmod 777 /etc/systemd/system/docker.service
- vim /etc/systemd/system/docker.service
- 在 ExecStart=/usr/bin/dockerd 后面添加 --registry-mirror=https://05jbbkn9.mirror.aliyuncs.com（你阿里云容器镜像加速地址）
- systemctl daemon-reload
- systemctl restart docker
- ps -ef|grep docker 查看 Docker 是否成功加速，和下图一样你就加速成功了
  - <img width="788" height="47" alt="Linux：虚拟化31" src="https://github.com/user-attachments/assets/992d3d9b-3840-4643-baf0-ca33531237b5" />

# 04. Docker 简单应用
- 按照上面的步骤安装 Docker 服务
- docker pull wordpress # 下载 wordpress 镜像
- docker pull mariadb # 下载 mariadb 镜像
- docker run --name db --env MYSQL_ROOT_PASSWORD=example -d mariadb<br>   # docker run（运行镜像为容器） --name（声容器像别名） db（容器别名 ） --env（向容器内部注入环境变量） MYSQL_ROOT_PASSWORD（变量键）=example（变量值） -d（放在后台运行） mariadb（镜像名称）
- docker run --name MyWordPress --link db:mysql -p 8080:80 -d wordpress<br> # docker run（运行镜像为容器） --name（声明容器别名） MyWordPress （容器别名 ）--link（链接） db（上面 mariadb 镜像的容器别名）:mysql（容器别名） -p 8080（宿主机端口）:80（容器内部端口）-d wordpress（镜像名称）
- 然后去浏览器输入 https:\\Docker 服务器 IP:8080 即可进入容器应用





