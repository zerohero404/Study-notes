# 1. Docker 基础概念
- Docker 三个重要概念
  - 仓库(Repository)：镜像存放的地方
  - 镜像 (image)：未运行的容器
  - 容器 (Container)：运行的镜像
- Docker 指令的基本用法
  - docker + 命令关键字(COMMAND) + 一系列的参数
  - 举例
    - docker run --name MyWordPress --link db:mysql -p 8080:80 -d wordpress
    - docker run（下载运行容器） --name（指定容器别名） MyWordPress（容器别名） --link（将本容器和其他容器链接） db（其他容器）:mysql -p 8080（宿主机端口）:80（容器内部端口） -d wordpress（镜像名称）
- Docker 基础命令
  - docker info        # 守护进程的系统资源设置
  - docker search 镜像名     # Docker 仓库的查询
  - docker pull 镜像名        # Docker 仓库的下载
  - docker images       # Docker 镜像的查询
  - docker rmi 镜像名:版本号  # Docker镜像的删除,正在运行的镜像不允许删除
  - docker ps          # 容器的查询，这个只能查看正在运行的，要查看所有的可以加 -a
  - docker run 镜像名      # 容器的创建启动
  - docker start/stop   # 容器启动停止
  - docker rm 容器 # 删除容器
  - Docker 指令除了单条使用外，还支持赋值、解析变量、嵌套使用 # 例如删除当前所有容器
  - docker rm $( docker ps -a -q ) # docker ps -a -q 是只显示容器的 CONTAINER ID

# 2. 单一容器管理
- 每个容器被创建后，都会分配一个 CONTAINER ID 作为容器的唯一标示，后续对容器的启动、停止、修改、删除等所有操作，都是通过 CONTAINER ID 来完成，偏向于数据库概念中的主键
  - 指令
    - docker ps -a --no-trunc             #查看容器的全部信息
    - docker stop/start CONTAINER ID # 停止这个 CONTAINER ID 容器
    - docker start/stop 容器别名 # 通过容器别名启动/停止
    - docker inspect 容器别名             #查看容器所有基本信息
    - docker logs 容器别名               #查看容器日志
    - docker stats 容器别名              # 查看容器所占用的系统资源
    - docker exec 容器名 容器内执行的命令     容器执行命令 # 容器在容器内部执行命令
    - docker exec -it 容器名 /bin/bash  # 进入容器内部对容器进行操作
  - Docker run 选项可加选项
    - --restart=always    # 让跟随 docker 容器的启动而启动
    - -h xxx # 设置容器主机名
    - --dns xx.xx.xx.xx           # 设置容器使用的 DNS 服务器
    - --dns-search # DNS 搜索设置
    - --add-host hostname:IP # 在 hostname <> 文件注入 IP 解析
    - --rm                      # 服务停止时自动删除容器信息

# 3. 多容器管理
- 多容器管理概念
  - Docker 提倡理念是“一个容器一个进程”，假设一个服务器需要多个进程组成，就需要多个容器自称一个系统，相互分工和配合对外提供完整服务
    - 例如上面的博客系统就有两个组件
      - 组件1：mariadb
      - 组件2：WordPress 的 Acaphe Web
  - 启动容器时，同一台主机下如果两个容器之间需要数据交流，使用 -link 选项建立两个容器之间的互联，前提是建立 mariadb 已经开启
    - 以上面的博客系统举例启动 # 容器开启顺序应该是先数据库后网页服务
      - docker start db
      - docker start MywordPress
    - 停止 # 容器停止顺序应该是先网页服务后数据库
      - docker stop db MywordPress
      - 或者 docker stop db && docker stop MywordPress
- Docker-compose
  - 而当容器过多时，管理员就容易出错，所以需要借助 Docker-compose 对多容器进行管理
  - Docker-compose 安装
    - 去 https://github.com/docker/compose/releases 下载想要版本，linux 系统下载 docker-compose-Linux-x86_64
    - 将下载下来的文件改名为 docker-compose，上传到 Docker 服务器的 /usr/local/bin 路径下，然后赋予执行权限
      - chmod 777 docker-compose
    - docker-compose --version # 查看是否安装成功，查看版本信息
  - Docker-compose 用法
    - -f # 指定使用的 yaml 文件位置
    - ps # 显示所有容器信息
    - restart # 重新启动容器
    - logs # 查看日志信息
    - config -q # 验证 yaml 配置文件是否正确
    - stop # 停止容器
    - start # 启动容器
    - up -d # 启动容器项目
    - pause # 暂停容器
    - uppause # 恢复暂停
    - rm #删除容器


# 4.镜像、仓库管理
## 4.1 镜像特征
容器创建时需要指定镜像，每个镜像都由唯一的标示 Image ID ，和容器的 Container ID 一样，默认 128 位，可以使用前 16 为缩略形式，也可以使用镜像名与版本号两部分组合唯一标示，如果省略版本号，默认使用最新版本标签( latesr )
- 镜像的分层：Docker 的镜像通过联合文件系统 ( union filesystem ) 将各层文件系统叠加在一起
  - bootfs 层：用于系统引导的文件系统，包括 bootloader 和 kernel，容器启动完成后会被卸载以节省内存资源
  - roofs：位于 bootfs 之上，表现为 Docker 容器的跟文件系统
    - 传统模式中，系统启动时，内核挂载 rootfs 时会首先将其挂载为“只读”模式，完整性自检完成后将其挂载为读写模式
    - Docker 中，rootfs 由内核挂载为“只读”模式，而后通过 UFS 技术挂载一个“可写” 层
- 镜像分层规则
  - 已有的分层只能读不能修改
  - 上层镜像优先级大于底层镜像
  - <img width="420" height="350" alt="Linux：虚拟化32" src="https://github.com/user-attachments/assets/34a4024b-0eb6-4f33-ac3f-9d18b3ea6905" />

## 4.2 DockerFile
- 容器转镜像命令
  - docker commit CID 容器名
  - 注意：镜像运行为容器后容器内部必须有一个工作在前台的守护进程，例如 Mysql 数据库，不然镜像在运行成容器的那一瞬间 docker 就会认为这个容器没有运行的必要，然后关闭这个容器
- DockerFile
  - Dockfile 是一种被 Docker 程序解释的脚本，Dockerfile 由一条一条的指令组成，每条指令对应 Linux 下面的一条命令。Docker 程序将这些 Dockerfile 指令翻译真正的 Linux 命令。Dockerfile 有自己书写格式和支持的命令， Docker 程序解决这些命令间的依赖关系，类似于 Makefile。Docker 程序将读取 Dockerfile，根据指令生成定制的 image
  - DockerFile 每一排指令都是一层，最多能构建128层
- dockerfile 基本指令
  - FROM（制定基础镜像（image））
    - 构建指令，必须制定且需要在 Docker 其他指令的前面。后续的指令都依赖于该指令制定的镜像。FROM 指令制定的基础镜像可以是官方远程仓库中的，也可以位于本地仓库
    - 举例
      - FROM centos:7.2
  - MAINTAINER（用来制定镜像创建者的信息）
    - 构建指令，用于将镜像的制作者相关的信息写入到镜像中，当我们对这个镜像执行 docker inspect 命令的时候，输出中油相对应的字段记录该信息
    - 举例
      - MAINTAINER XXX”XXXX.COM”
  - RUN（安装软件用）
    - 构建指令，RUN 可以运行任何被基础镜像支持的命令。如基础镜像选择了 Centos，那么 RUN 就可以支持 ls、tar、cd 等命令
    - 举例
      - RUN cd /tmp && curl -L “http://archive.apache.org/dist/tomcat/tomcat-7/v7.0.8/bin/apache-tomcat-7.0.8.tar.gz” | tar -xf
      - RUN [ “/bin/bash”,”-c”,”echo hello” ]
  - CMD（设置容器（container）启动时执行的操作）
    - 设置指令，用于容器（container）启动时执行指定的操作，该操作可以是执行自定义脚本，也可以是执行系统命令。该指令只能在文件中存在一次，如果有多条这个指令，则只执行最后一条
    - 举例
      - CMD echo “hello,world”
  - ENTRYPOINT（设置容器（container）启动时执行的操作）
    - 设置指令，指定容器启动时执行的命令，可以多次设置，但是只有最后一个有效
    - 该指令使用有两种情况，一种是单独使用，另外一种和 CMD 指令配合使用。当独自使用，你还使用了 CMD 指令且 CMD 指令是一个完整的可执行的命令，那么 CMD 指令和 ENTRYPOINT 会互相覆盖，只有最后一个 CMD 或者 ENTRYPOINT 生效 # CMD 指令将不会被执行，只有 ENTRYPOINT 指令会被执行
      - CMD echo “hello,world”
        - ENTRYPOINT ls -l
    - 另一种用法和 CMD 指令配合使用用来指定 ENTRYPOINT 的默认参数，这时候 CM,D 指令不是一个完整的可执行命令，仅仅是参数部分；ENTRYPOINT 指令只能使用 JSON 方式指定执行命令，而不能指定参数
      - FORM ubuntu CMD [ “-l” ] ENTRYPOINT [ “/usr/bin/ls” ]
  - USER（设置容器（container）的用户）
    - 设置指令，设置启动容器的用户，默认是 root 用户
    - USER = admin
    - EXPOSE（指定容器需要映射到宿主机器的端口）
      - 设置指令，该指令会将同期中的端口映射成宿主机器中的某个端口，当你需要访问容器的时候，可以不是用容器的 IP 地址而是使用宿主机器的 IP 地址和映射后的端口。要完成这个操作需要两个步骤，首先在 Dockerfile 使用 EXPOSE 设置需要映射的容器端口，然后在运行容器的时候使用 -p 选项加上 EXPOSE 设置的端口，这样 EXPOSE 设置的端口号会被随机映射成宿主机器中的一个端口号。也可以制动需要映射到宿主机器的那个端口，这时要确保宿主机器上的端口号没有被使用。EXPOSE 指令可以一次设置多个端口号，相应运行容器的时候，也可以配套的多次使用 -p 选项
    - 举例
      - 映射一个端口：EXPOSE 端口1 # 相应的运行容器使用的命令 docker -run -p 端口1 镜像名（在终端执行，不写在 dockerfile 文件里）
      - 映射多个端口：EXPOSE 端口1 端口2 端口3 # 相应的运行容器使用的命令 docker -run -p 端口1 -p 端口2 -p 端口3 镜像名（在终端执行，不写在 dockerfile 文件里）
      - 指定需要映射到宿主机上的某个端口号 docker -run -p 宿主机端口:端口1 -p 宿主机端口:端口2 -p 宿主机端口:端口3 镜像名
  - ENV（用于设置环境变量）
    - 构建指令，在镜像中设置一个环境变量
    - 使用 ENV 设置环境变量后，后续的 RUN 命令都可以使用，容器启动后，可以通过 docker insoect 查看这个环境变量，也可以通过在 docker run --envkey=value 时设置或修改环境变量，假如你安装了 JAVA 程序，需要设置 JAVA_HOME,那么可以在 dockerfile 中这样写
      - ENV JAVA_HOME /path/to/java/dirent（java 程序路径） ENV PATH $JAVA_HOME/bin:$PATH
  - ADD（从宿主机复制文件制作成镜像的容器内部）
    - 构建指令，将宿主机文件复制到要制作成镜像的容器内部，如果是压缩文件就解压缩
    - 举例
      - ADD 宿主文件路径 制作成镜像的容器内部路径 # 其中宿主文件路径也可以是一个远程文件 url
  - COPY（从宿主机复制文件制作成镜像的容器内部）
    - 构建指令，与 ADD 指令功能一致，但是不会解压缩压缩文件
    - COPY 宿主文件路径 制作成镜像的容器内部路径 # 其中宿主文件路径也可以是一个远程文件 url
  - VOLUME（指定挂载点）
    - 设置指令，使容器一个目录具有持久化存储数据的能力，该目录可以被容器本身使用，也可以共享给其他容器使用。我们知道容器使用的是 AUFS，这种文件系统不能持久化数据，当容器关闭后，所有的更改都会丢失，当容器中的应用有持久化数据的需求可以在 dockerfile 中使用该指令
    - 举例
      - FROM<br>
        VOLUME [ “/tmp/data” ]<br>
  - WORKDIR（切换目录）
    - 设置指令，可以多次切换（相当于 cd 命令），对 RUN，CMD，ENTRYPOINT 生效
    - 举例
      - WORKDIR /root WORKDIR mydata RUN vim 1.txt # 相当于 RUN cd /root/mydata && vim 1.txt
    - ONBUILD（在子镜像里执行）
      - ONBUILD 制定的命令并不会在自己构建镜像是执行，在别人拿我构建的镜像再去构建镜像时执行
      - ONBUILD RUN rm -rf *
- DockerFile 实例演示
```bash
FROM hub.c.163.com/public/centos:6.7
# 声明基础镜像
MAINTAINER wangyang@itxdl.cn
# 设置个人标签

ADD ./apache-tomcat-7.0.42.tar.gz /root
ADD ./jdk-7u25-linux-x64.tar.gz /root
# 将两个文件上传到这个要制作成镜像的容器内部

ENV JAVA_HOME /root/jdk1.7.0_25
# 将 java 程序所在目录声明成 JAVA_HOME 变量
ENV PATH $JAVA_HOME/bin:$PATH
# 把 JAVA_HOME 声明成全局变量

EXPOSE 8080
# 声明映射端口

ENTRYPOINT /root/apache-tomcat-7.0.42/bin/startup.sh && tailf /root/apache-tomcat-7.0.42/logs/catalina.out
# 容器运行要执行的操作，启动 startup.sh 脚本，并且把查看 catalina.out 文件放在前台，防止容器自动关闭
```
- vim Dockerfile 把上面内容复制进去，再把 apache-tomcat-7.0.42.tar.gz 和 jdk-7u25-linux-x64.tar.gz 两个文件和 Dockerfile 文件放在一个目录，再在当前目录执行 docker build -t 镜像名:版本号 ./（当前目录）就完成镜像制作了，然后运行就可以了
  - docker run --name tomcat -p 80:8080 -d tomcat:v1.0

## 4.3 私有仓库搭建
- Harbor
  - 项目地址：https://github.com/vmware/harbor
  - Harbor 是 VMware 公司开源的企业级 DockerRegistry 项目，其目标是帮助用户迅速搭建一个企业级的 Dockerregistry 服务。它以 Docker公 司开源的 registry 为基础，提供了管理 UI， 基于角色的访问控制(Role Based Access Control)， AD/LDAP 集成、以及审计日志 (Auditlogging) 等企业用户需求的功能，同时还原生支持中文。Harbor 的每个组件都是以 Docker 容器的形式构建的，使用 Docker-Compose 来对它进行部署。用于部署 Harbor 的Docker Compose 模板位于  /Deployer/docker-compose.yml，
  - Harbor 由5个容器组成，这几个容器通过 Docker link 的形式连接在一起，在容器之间通过容器名字互相访问。对终端用户而言，只需要暴露 proxy （ 即Nginx）的服务端口
    - Proxy：由Nginx 服务器构成的反向代理
    - Registry：由Docker官方的开源 registry 镜像构成的容器实例
    - UI：即架构中的 core services， 构成此容器的代码是 Harbor 项目的主体。
    - MySQL：由官方 MySQL 镜像构成的数据库容器
    - Log：运行着 rsyslogd 的容器，通过 log-driver 的形式收集其他容器的日志
- Harbor 特性
  - 基于角色控制：用户和仓库都是基于项目进行组织的， 而用户基于项目可以拥有不同的权限
  - 基于镜像的复制策略：镜像可以在多个Harbor实例之间进行复制
  - 支持LDAP： Harbor的用户授权可以使用已经存在LDAP用户
  - 镜像删除 & 垃圾回收： Image可以被删除并且回收Image占用的空间，绝大部分的用户操作API， 方便用户对系统进行扩展
  - 用户UI：用户可以轻松的浏览、搜索镜像仓库以及对项目进行管理
  - 轻松的部署功能： Harbor提供了online、offline安装，除此之外还提供了virtualappliance安装
  - Harbor 和 docker registry 关系： Harbor实质上是对 docker registry 做了封装，扩展了自己的业务模块
  - <img width="500" height="258" alt="Linux：虚拟化33" src="https://github.com/user-attachments/assets/e91d757e-3fd8-4813-9814-7554162ff682" />
- Harbor 拉取镜像认证过程
  - dockerdaemon从docker registry拉取镜像
  - 如果dockerregistry需要进行授权时， registry将会返回401 Unauthorized响应，同时在响应中包含了docker client如何进行认证的信息
  - dockerclient根据registry返回的信息，向auth server发送请求获取认证token
  - auth server则根据自己的业务实现去验证提交的用户信息是否存符合业务要求
  - 用户数据仓库返回用户的相关信息
  - auth server将会根据查询的用户信息，生成token令牌，以及当前用户所具有的相关权限信息.上述就是完整的授权过程.当用户完成上述过程以后便可以执行相关的pull/push操作。认证信息会每次都带在请求头中
  - Harbor整体架构
    - <img width="378" height="191" alt="Linux：虚拟化34" src="https://github.com/user-attachments/assets/ac055f3e-4e68-4a0a-90e0-594995c69353" />
- Harbor 认证流程
  - 首先，请求被代理容器监听拦截，并跳转到指定的认证服务器
  - 如果认证服务器配置了权限认证，则会返回401。通知dockerclient在特定的请求中需要带上一个合法的token。而认证的逻辑地址则指向架构图中的core services
  - 当docker client接受到错误code。client就会发送认证请求(带有用户名和密码)到coreservices进行basic auth认证
  - 当C的请求发送给ngnix以后， ngnix会根据配置的认证地址将带有用户名和密码的请求发送到core serivces
  - coreservices获取用户名和密码以后对用户信息进行认证(自己的数据库或者介入LDAP都可以)。成功以后，返回认证成功的信息
  - Harbor认证流程图
    - <img width="379" height="156" alt="Linux：虚拟化35" src="https://github.com/user-attachments/assets/a26cd2f9-1f63-479d-b35b-6d326d3e96ba" />
- Harbor 搭建步骤
  - Harbor 对系统要求
    - Python应该是2.7或更高版本
    - Docker引擎应为1.10或更高版本
    - Docker Compose需要为1.6.0或更高版本
  - 去 https://github.com/goharbor/harbor/releases/tag/v1.2.0 下载 harbor-offline-installer-v1.2.0.tgz
  - 上传到 docker 服务器，docker 开启
  - tar -xf harbor-offline-installer-v1.2.0.tgz # 解压你上传的压缩包
  - mv harbor /usr/local/
  - mkdir /key && cd /key
  - openssl genrsa -des3 -out server.key 2048 然后输入密码 # 配置 Harbor 需要的 https 证书
  - openssl req -new -key server.key -out server.csr 输入密码之后依次输入你的国家、省份、城市、公司名称、公司单位名称、域名、邮件地址、是否修改密码、可选公司名称
  - cp server.key server.key.org
  - openssl rsa -in server.key.org -out server.key # 给证书退密，因为 Harbor 登录无法输入密码
  - openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
  - mkdir -p /data/cert # 创建 Harbor 读取证书路径
  - mv * /data/cert # 把所有证书移动到 Harbor 读取证书路径
  - chmod -R 777 /data/cert
  - cd /usr/local/harbor
  - vim harbor.cfg # 修改 harbor 安装配置文件，修改一下几个选项
    - hostname = hub.zhangxu.com # 目标的主机名或者完全限定域名
    - ui_url_protocol = https # 建议修改成https。默认为http
    - db_password = root123 # 用于db_auth的MySQL数据库的根密码
    - max_job_workers = 3 # 默认值为3，作用是最大支持上传下载镜像最大线程
    - customize_crt = on #（on或off。默认为on）当此属性打开时，  prepare脚本将为注册表的令牌的生成/验证创建私钥和根证书
    - ssl_cert = /data/cert/server.crt # 你刚刚创建SSL证书的路径，仅当协议设置为https时才应用
    - ssl_cert_key = /data/cert/server.key # 你刚刚创建 SSL密钥的路径，仅当协议设置为https时才应用
    - secretkey_path = /data # 用于在复制策略中加密或解密远程注册表的密码的密钥路径
    - harbor_admin_password = harbor123 # Harbor 网页 admin 管理员的密码默认是 Harbor12345
  - ./install.sh # 运行脚本安装
  - 然后输入 https://hub.zhangxu.com（你再配置文件填写的 hostname）就可以进入管理页面
    - 需要 DNS 服务器将 hub.zhangxu.com 解析成你的 docker 服务器 IP地址
    - 如果没有你需要修改本地 Hosts 文件
      - windows 路径 ：C:\Windows\System32\drivers\etc
      - linux：/etc/hosts
- 上传、下载给私有镜像
  - vim /usr/lib/systemd/system/docker.service # 打开 docker 配置文件
  - 在 ExecStart=/usr/bin/dockerd 后面添加 --insecure-registry=hub.zhangxu.com （harbor 限定域名）
  - vim /etc/docker/daemon.json
    - 添加
```bash
      {
      "insecure-registries": ["harbor 主机名"]
      }
  - systemctl daemon-reload
  - systemctl restart docker
- 上传镜像
  - docker tag bf756（要上传镜像 ID 号） hub.zhangxu.com（私有仓库限定域名）/my-dockerhub（仓库名）/hello:v1.0（仓库显示镜像名:版本号） # 首先给你要上传的镜像打上标签
  - docker login hub.zhangxu.com（私有仓库限定域名） 输入账号密码 # 如果你要上传的是私有仓库,还需要登录
  - docker push hub.zhangxu.com/my-dockerhub/hello:v1.0（你刚刚创建的镜像标签） # 上传镜像
- 下载镜像
  - docker login hub.zhangxu.com
  - docker pull hub.zhangxu.com/my-dockerhub/hello:v1.0
























