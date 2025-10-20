# rsync 概述
- rsync 是类 unix 系统下的数据镜像备份工具。一款支持快速完全备份和增量备份的工具，支持本地复制，远程同步，类似于 scp 命令；rsync 命令在同步文件之前要先登录目标主机进行用户身份认证，认证之后才能进行数据同步，身份认证方式取决于所使用的协议类型，rsync 一般使用两种协议进行数据同步：ssh 协议和 rsync 协议
# rsync 特性
- 1.能更新整个目录树和文件系统
- 2.有选择行的保留符号链接、硬链接、文件属性、权限、设备以及时间等
- 3.对于安装来说，无任何特殊权限要求
- 4.对于多个文件来说，文件传输效率高
- 5.能使用 ssh 或者自定义端口作为传输入口端口
# rsync 工作原理
- 既然涉及到数据同步，两个必要的概念L源地址（文件）、目标地址（文件），以及以哪一方为基准。例如，想让目标主机上的文件和本地文件保持同步，则是以本地文件以同步基准，将本地文件作为源文件图送到目标主机上。
- rsync 在数据同步之前需要先进行用户身份验证，验证方式取决于使用的连接方式：
- ssh 登录验证模式：使用 ssh 协议作为基础进行身份认证，然后进行数据同步
- rsync 登录验证模式：使用 rsync 协议进行用户身份认证（非系统用户），然后进行数据同步。
- 数据同步方式：推送（上传）、拉取（下载）
 <img width="846" height="259" alt="Linux：网络服务_45" src="https://github.com/user-attachments/assets/d42a7aae-3cda-4bb8-962e-f56175eed2d4" />

# rsync 实验演示
***我们一般使用 rsync 来进行单项数据同步，因此我们需要确定一个基准，比如：两台服务器，一台 NFS 作为网站数据服务器（基准服务器），另外一台专门做 rsync 数据备份服务器，我们以此为基础开始我们的实验。***
- ssh 协议数据同步：将 NFS 服务器数据同步备份到 rsync 服务器
- 实验环境：一台 NFC 服务器，一台 rsync 服务器
- 在两台服务器上分别创建目录：NFC 服务器：/filesrc 和 rsync 服务器：/filedst
- 下行同步（下载）
-    1.格式：rsnyc -avz NFC服务器的用户@NFC服务器IP地址:/NFC服务器目录/* /本地目录
-    2.示例：rsnyc -avz root@192.168.88.10: /filesrc/* /filedst
-    -a:归档模式，递归并保留对象属性  -v：显示同步过程 -z：在传输文件时进行压缩
     
## 上行同步（上传）
- 格式：rsnyc -avz /本地目录/* NFC服务器的用户@NFC服务器IP地址:/NFC服务器目录/
- 示例：rsnyc -avz /filedst/* root@192.168.88.10: /filesrc/
- 注意：是在实验中使用 root 用户可以，但是生产环境尽量使用 NFC 服务器单独创建的普通用户，减少权限溢出
```bash
useradd xxxxx
passwd xxxxx
setfacl -m u:xxxxx:rwx /filesrc
```
- 如果要实现免密数据同步，只需要做好 ssh 密钥对登录即可
1.NFC 服务器操作
```bash
ssh-keygen -t rsa -b 2048 （然后一直按回车）
ssh-copy-id rsnyc服务器用户@rsny服务器地址
2.rsnyc 服务器操作
```bash
ssh-keygen -t rsa -b 2048(然后一直按回车)
ssh-copy-id NFC服务器用户@NFC服务器IP地址
```
- rsync 协议数据同步：将 NFS 服务器数据同步备份到 rsync 服务器
实验环境：一台服务器，一台客户端
1.在两台服务器上分别创建目录
  1.1.NFC 服务器：/filesrc
  1.2.rsync 服务器：/filedst
2.搭建 rsync 服务（仅需要在 NFC 服务器上搭建)
  2.1.创建主配置文件（/etc/rsyncd.conf）
  ```bash
  address = NFC 服务器 IP 地址 #rsync 服务绑定IP
  port 873 # 默认服务端口 873
  log file = /var/log/rsyncd.log # 日志文件位置
  pid file = /var/run/rsyncd.pid # 进程号文件位置
  [web] # 共享名：是用来写在url上的
  comment = web directory backup # 共享描述话语
  path= /filesrc # 实际共享目录
  read only =on # 是否仅允许读取
  dont compress = *.gz *.bz2 # 哪些文件类型不进行压缩
  auth users = user1 # 登录用户名（非系统用户，需要自行创建）
  secrets file = /etc/rsyncd_users.db # 认证所需账户密码文件（需自行创建-同上）
  2.2.创建认证所需账户密码文件
  ```bash
  vim /etc/rsyncd_users.db
  # 写入主配置文件中的登录用户名和密码
  user1:123456
  chmod 600 /etc/rsyncd_users.db ( 必须修改权限，否则登录报错)
  ```
3.启动服务
```bash
rsync –daemon
netstat -antp | grep :873
```
4.设置映射用户对共享目录有权限（r）
  
