# java web 环境：Nginx+JDK+Tomcat+MySQL
 <img width="805" height="457" alt="Linux：网络服务_47" src="https://github.com/user-attachments/assets/304ade1e-a643-4b93-bf51-8fac12ab1b10" />
 <img width="325" height="204" alt="Linux：网络服务_48" src="https://github.com/user-attachments/assets/6398ddf2-1c10-4ffa-9f06-5195c84440ba" />

所有服务都部署在统一主机上，也可以分开部署

Nginx 默认开启的是 80 端口，用来接收用户的 web 请求

tomcat 默认开启的是 8080 端口，用来接收 Nginx转发过来的 web 请求

## 环境部署流程
- 安装 JDK （java 解析器）
1.首先安装 gcc: yum -y install gcc
2.下载、解压安装包，移动到指定位置
  ```bash
  mkdir /java
  cd /java
  wget –no-check-certificate –no-cookies –header “Cookie: oraclelicense=accept-securebackup-cookie” http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
  tar -xf jdk-8u261-linux-x64-demos.tar.gz
  cp -a jdk1.8.0_131/ /usr/local/jdk
  ```
3.配置 JDK 的环境变量

 3.1.vim /etc/profile
 3.2.在文件里添加
 ```bash
 export JAVA_HOME=/usr/local/jdk
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export PATH=$PATH:${JAVA_PATH}
```
 3.3.source /etc/profile # 加载下配置文件
 3.4.java -version # 查看 java 是否安装成功
4.安装 tomcat
 4.1.下载、解压安装包，移动到指定位置
 ```bash
 mkdir /usr/local/tomcat
cd /java
wget https://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.38/bin/apache-tomcat-9.0.38.tar.gz
cp -r apache-tomcat-9.0.38 /usr/local/tomcat
```
 4.2.配置 Tomcat 的环境变量
 
 vim /etc/profile
 ```bash
 export TOMCAT_HOME=/usr/local/tomcat
export PATH=$PATH:$TOMCAT_HOME/bin
```
 4.3.source /etc/profile # 加载下配置文件
 4.4.将 tomcat 的启动脚本赋予执行权限
 ```bash
 chmod +x /usr/local/tomcat/bin/*
 ```
 4.5./usr/local/tomcat/bin/catalina.sh start #启动 tomca

 4.6.netstat -antp # 查看 tomcat 是否已经启动

 4.7.在客户端访问 服务器IP地址：8080 进行测试

 ## 安装 MySQL 数据库
 1.安装依赖包 ncurses-devel
 ```bash
 yum -y install ncurses-devel gcc
```
2.将 mysql 文件进行传输到 192.168.20.10 上进行安装
```bash
useradd -r -s /sbin/nologin mysql
./configure –prefix=/usr/local/mysql –with-charset=utf8 –with-collation=utf8_general_ci –with-extra-charsets=gbk,gb2312
make && make install
```
3.生成配置文件
```bash
cp -a support-files/my-medium.cnf /etc/my.cnf
ln -s /usr/local/mysql/bin/* /usr/local/bin/
ln -s /usr/local/mysql/sbin/* /usr/local/sbin/
```
4.初始化数据库，生成授权表
```bash
cd /usr/local/mysql
./bin/mysql_install_db –user=mysql
```
5.生成启动管理脚本，启动 mysql 并设置开机自启
```bash
cd ~/mysql-x.x.xx/support-file
cp -a mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
chkconfig –add mysqld
chkconfig mysqld on
service mysqld start|stop|restar
```
6.为数据库的管理用户 root 设置登录密码
```bash
mysqladmin -uroot password xxxxx
```
7.登录数据库，查看是否安装正确
```bash
mysql -uroot -p
```

## 安装 Nginx
1.上传 nginx 软件包并且解压
```bash
tar -xf nginx-x.x.x.tar.gz
```
2.安装 nginx 依赖包
```bash
yum -y install pcre-devel zlib-devel gcc
```
3.添加用户
```bash
useradd -r -s /sbin/nologin nginx
```
4.编译并安装
```bash
./configure –user=nginx –group=nginx && make && make install
```
5.使用 nginx 反向代理tomcat
 
 5.1.修改 nginx 配置文件
 
 vim /usr/local/nginx/conf/nginx.conf
 ```bash
 user nginx;
 upstream tomcat { #添加负载调度（为了后期扩展更多 Tomcat 服务器方便)
 server tomcat服务器IP地址:8080;
 }
 location / { #添加反向代理
 proxy_pass http://tomcat;
 proxy_set_header Host $host;
 }
 ```
6.重启 nginx 服务
```bash
pkill -HUP nginx
```
7.输入 nginx 的地址，打开为 tomcat 就是tomcat 部署的网站
