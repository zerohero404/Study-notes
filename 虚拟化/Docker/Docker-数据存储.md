# 1. 数据卷特性
- Docker 镜像由多个只读层叠加而成，启动容器时，Docker 会加载只读镜像层并在镜像栈顶部添加一个读写层
- 如果运行中的容器修改了现有的一个已经存在的文件，那么该文件将会从读写层下面的的只读层复制到读写层，该文件的只读版本仍然存在，只是已经被读写层中该文件的副本所隐藏，次即“写时复制”机制
<img width="456" height="257" alt="Linux：虚拟化37" src="https://github.com/user-attachments/assets/0ad07407-38c3-491a-b017-b9bc7225ad40" />

# 2. 数据卷意义
- 关闭并重启容器，其数据不受影响；但删除 Docker 容器，则其改变将会全部丢失
- 不使用数据卷容易产生的问题
  - 存在于联合文件系统中，不易于宿主机访问
  - 容器间数据共享不便
  - 删除容器其数据会丢失
- 使用“卷”
  - “卷”是容器上的一个或多个“目录”，此类目录可绕过联合文件系统，与宿主机上的某目录“绑定”
    - <img width="417" height="139" alt="Linux：虚拟化38" src="https://github.com/user-attachments/assets/1892fc04-dceb-483a-bc10-47425dfd67fb" />
  - Volume 可以在运行容器时即完成创建与绑定操作。当然，前提需要拥有对应的申明
  - Volume 的初衷就是数据持久化
    - <img width="646" height="271" alt="Linux：虚拟化39" src="https://github.com/user-attachments/assets/b5396679-4db6-429e-9264-513e1b33f876" />
- 数据卷类型
  - 容器启动加选项制定数据卷挂载点（Bind mount volume）
  - Dockerfile 文件使用 VOLUME 选项启动容器自动挂载，其挂载点在 /var/lib/docker/vfs/dir/ 路径下的随机目录（Docker-managed volume）
  - <img width="601" height="212" alt="Linux：虚拟化40" src="https://github.com/user-attachments/assets/eecff338-ae30-4cd5-91b7-eaf28e2f1a5c" />

# 3. 数据卷挂载方法
- Docker-managed volume
  - 在 Dockerfile 文件里写入
    - VOLUME /容器中要挂载的路径
  - 然后用这个 Dockerfile 创建镜像，然后吧这个镜像运行为容器后容器会将 Dockerfile 写的路径自动挂载到 /var/lib/docker/vfs/dir/ 路径下的随机目录
- Bind mount volume
  - docker run -d --name 容器名 -v 宿主机挂载路径:容器挂载路径 roc/镜像名:版本
- Docker-managed volume 创建镜像之后也可以使用 -v 宿主机挂载路径:容器挂载路径 来指定路径，-v 的优先级比 dockerfile 配置要搞高
- 如果两个容器想要共享数据，可以把两个容器挂在路径挂载在宿主机同一个目录下
- 如果不想两个容器启动时输入 -v 指定相同的宿主机目录 ，可以将容器一设置为 Docker-managed volume 然后启动，容器二启动时 docker run -d --name 容器名 --volumes-from 容器一容器名 镜像名:镜像版本，这样启动时容器二就会使用容器一挂载在 /var/lib/docker/vfs/dir/ 下的路径
# 4 存储驱动
- Docker 存储驱动 ( storage driver ) 是 Docker 的核心组件，它是 Docker 实现分成镜像的基础，有三种主流驱动
  - device mapper ( DM )：性能和稳定性存在问题，不推荐生产环境使用
  - btrfs：社区实现了 btrfs driver，稳定性和性能存在问题
  - overlayfs：unix 系统内核 3.18 overlayfs 进入主线，性能和稳定性优异，第一选择
  - 输入 dokcer info 在 Storage Driver 查看你的驱动是哪一种
- 修改存储驱动为 overlayfs 存储驱动
  - echo “overlay” > /etc/modules-load.d/overlay.conf
  - cat /proc/modules | grep overlay
  - reboot
  - vim /etc/systemd/system/docker.service
  - --storage-driver=overlay \
- Docker 存储文件方法
  - <img width="663" height="170" alt="Linux：虚拟化41" src="https://github.com/user-attachments/assets/c339e4a8-2e95-4292-b773-de25dcbdc57b" />











































