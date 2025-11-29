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


























