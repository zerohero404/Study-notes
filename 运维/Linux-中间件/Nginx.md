# 1.1 Nginx 用到的几种概念

## 同步与异步

同步与异步的重点在消息通知的方式上，也就是调用结果的通知方式不同：

- **同步**：当一个同步调用发出去之后，调用者要一直等待调用的结果通知后，才能进行后续的执行。
- **异步**：当一个异步调用发出去后，调用者不必一直等待调用结果的返回。异步调用想要获得结果一般有两种方式：
  1. 主动轮询异步调用的结果
  2. 被调用方通过 callback（回调通知）来通知调用方调用结果

### 实例解释

- **同步**：小明收到快递即将送达的短信，在楼下一直等着。
- **异步**：小明收到快递即将送达的短信，快递到楼下后，小明再下楼去取。小明知道快递到达楼下有两种方式：
  - 不停的打电话给快递小哥，即主动轮询
  - 快递小哥到楼下后，打电话通知小明，然后小明下楼取快递，即回调通知

## 阻塞与非阻塞

阻塞与非阻塞的区别在于进程/线程等待消息时候的行为，也就是在等待消息的时候，当前进程/线程是挂起状态还是非挂起状态：

- **阻塞**：调用发出去后，在消息返回之前，当前进程/线程会被挂起，直到有消息返回，当前进程/线程才会被激活。
- **非阻塞**：调用发出去后，不会阻塞当前进程/线程，而会立即返回。

### 实例解释

- **阻塞取快递**：小明收到快递即将送达消息，什么事都不做，一直专门等快递。
- **非阻塞取快递**：小明收到快递即将送达消息，等快递的时候，还一边敲代码、一边刷微信。

## 四种组合方式

同步与异步，重点在于消息通知的方式；阻塞与非阻塞，重点在于等消息时候的行为，所以有下面四种组合方式：

1. **同步阻塞**：小明收到信息后，啥都不干，等快递。
2. **同步非阻塞**：小明收到信息后，边刷微博，边等着取快递。
3. **异步阻塞**：小明收到信息后，啥都不干，一直等着快递员通知他取快递。
4. **异步非阻塞**：小明收到信息后，边刷微博，边等快递员通知他取快递。

大部分程序的 I/O 模型都是同步阻塞，单个进程每次只在一个文件描述上执行 I/O 操作，每次 I/O 系统调用都会阻塞，直到完成数据传输。传统的服务器采用的就是同步阻塞的多进程模型。一个 server 采用一个进程负责一个 request 的方式，一个进程负责一个 request，直到会话结束。进程数就是并发数，而操作系统支持的进程数是有限的，且进程数越多，调度的开销也越大，因此无法面对高并发。

Nginx 采用了异步非阻塞的方式工作。我们先来了解一下 I/O 多路复用中的 epoll 模型。

## epoll 模型

当连接有 I/O 事件产生的时候，epoll 就会去告诉进程哪个连接有 I/O 事件产生，然后进程去处理这个事件。

**实例解释**：  
小明家楼下有一个收发室，每次有快递到了，门卫就先代收并做了标记；然后再通知小明去取快递。

## 为什么 Nginx 比其他 Web 服务器并发高（Nginx工作原理）

- Nginx 配置 `use epoll` 后，以异步非阻塞方式工作，能够处理百万级的并发连接。
- 处理过程：
  1. 每个 request 进来时，会有一个 worker 进程去处理。
  2. 处理过程中可能会发生阻塞的地方（如向后端服务器转发 request 并等待返回），这个 worker 不会傻等，而是发送完请求后注册一个事件：“如果后端服务器返回了，告诉我一声，我再接着干”。
  3. 此时 worker 去休息，如果再有新的 request 进来，可以很快按这种方式处理。
  4. 一旦后端服务器返回，就触发事件，worker 接手 request 并继续处理。

通过这种快速处理、快速释放请求的方式，Nginx 可以在相同配置下处理更高的并发量。

# 1.2 Nginx 详解

 <img width="726" height="413" alt="Linux：网络服务_41" src="https://github.com/user-attachments/assets/64a6c8e5-124d-4d48-ac1e-f39899cae2c8" />
 
 <img width="913" height="405" alt="Linux：网络服务_42" src="https://github.com/user-attachments/assets/7e877149-c5ba-4f1d-9709-b1c36f277f99" />

 # 1.2 Nginx 工作模式

Nginx 有两种工作模式：**master-worker 模式**和**单进程模式**。

## Master-Worker 模式

在这种模式下，有一个 **master 进程** 和至少一个 **worker 进程**：

- **Master 进程**：
  - 处理系统信号
  - 加载配置
  - 管理 worker 进程（启动、杀死、监控、发送消息/信号等）
- **Worker 进程**：
  - 负责处理具体的业务逻辑
  - 对外提供服务，是真正的服务提供者

### 优点

1. **稳定性高**  
   worker 进程挂掉时，master 进程会立即启动新的 worker 进程，保证服务不中断。
2. **充分利用多核 CPU**  
   配合 Linux 的 CPU 亲和性配置，提升性能。
3. **支持热重启**  
   处理信号、配置重新加载或升级时，可以尽可能少或者不中断服务。

生产环境一般使用此模式。

## 单进程模式

在单进程模式下，Nginx 启动后只有一个进程，所有工作都由该进程负责：

- 优点：
  - 易于调试，可使用 gdb 等工具进行调试。
- 缺点：
  - 不支持平滑升级功能
  - 信号处理可能造成服务终端
  - 进程挂掉后，若无外部监控无法自动重启

一般只在开发阶段和调试阶段使用，生产环境不会采用。

# 1.3 Nginx 配置文件结构

## 配置文件位置

- Nginx 的配置文件位置

**<你安装nginx的路径>/nginx/conf/nginx.conf**

```bash
user XXX XXX;                            
# 程序运行用户和组
worker_processes  1;                     
# 启动进程，指定 nginx 启动的工作进程数量，建议按照 cpu 数目来制定，一般等于 cpu 核心数目
error_log /home/wwwlogs/nginx_error.log crit;
# 全局错误日志
pid        logs/nginx.pid;
# 主进程 PID 保存文件
worker_rlimit_nofile 51200;
# 文件描述符数量
events {
    use epoll;
    # 使用 epoll 模型，对于 2.6 以上的内核，建议使用 epoll 模型以提高性能
    worker_connections  1024;
    # 工作进程的最大了解数量
}

http{
    # 网站优化参数
    server {                                        # 某以网站的具体配置信息
        listen       80;                            # 监听端口
        root html;                                  # 网页根目录
        server_name  localhost;                     # 服务器域名
        index    index.html;                        # 默认加载页面
        access_log  logs/host.access.log  main;     # 访问日志保存位置
        location(.*)\.php${
            # 用正则匹配具体的访问对象
        }
        location{
            # 跳转规则
        }
     }
     sercer{
         # 虚拟主机
     }
}
```

## Nginx 相关实验

### 注意事项

#### 注意配置文件中的结尾都有”;” 每次修改完配置文件都需要重启 nginx 才会生效

```bash
pkill -HUP nginx
```

### 实验一：Nginx 的状态统计
#### 安装 nginx 时将 –with-http_stub_status_module 模块开启 修改 nginx 配置文件（写入要访问的 server 标签中）

```bash
location /nginx_status{
stub_status on;
access_log off;
}
```
#### 客户端访问网址
http://服务器IP/nginx_status,
“Active connections” 表示当前的活动连接数,
“server accepts handled requests” 表示已经处理的连接信息,
三个数字一次表示已经处理的连接数、成功的 TCP握手次数、已处理的请求数

### 实验二：目录保护
- 原理和 Apache 的目录保护原理一样
在状态统计的 location 中添加
```bash
auth_basic "欢迎来到 nginx_status!";
auth_basic_user_file
/usr/local/nginx/html/htppasswd.nginx;
```
- 使用 http 的命令 htpasswd 进行用户密码文件的创建（生成在上面指定的位置）
```bash
htpasswd -c /usr/local/nginx/html/htppasswd.nginx 用户名
```

### 实验三：基于 IP 的身份验证（访问控制）
- 实验三：基于 IP 的身份验证（访问控制）

```bash
allow 能通过的 IP地址 例如：192.168.88.1;
deny 不能通过的 IP域 例如：192.168.88.0/24;
(以上这个设置就是设置只能192.168.88.1这个IP地址访问)
```

### 实验四：nginx 的虚拟主机（基于域名）
- 提前准备好两个网站的域名，并且规划好两个网站网页存放目录
- 在 Nginx 主配置文件中并列编写两个 server 标签，并分别写好各自信息
- 注意：先要创建虚拟主机的网页根目录

```bash
server {                                       
        listen       80;                          
        root html/web1;                                
        server_name  www.xxxx.com;                    
        index     index.html index.php; 
        include    enable-php.conf                      
        access_log  logs/web1.access.log  main;    
}
server {                                    
        listen       80;                         
        root html/web2;                                  
        server_name   www.xxxx.com;                     
        index    index.html index.php;  
        include    enable-php.conf                       
        access_log  logs/web2.access.log  main;
        # 注意 这个 main 是日志的格式，需要查看篇日志上方的日志格式设置来设置，也可以根据自己的需求自己写   
}
# 这两个 server 不支持 PHP 解析，如果要支持 PHP 解析就在每个 server 里再添加上 http://www.192.168.11.130/?p=1136 里的 location ~ \.php$ 那段，但是里面 root 后面要与 server 里的 root 路径一样
```

### 实验五：nginx 反向代理
- 什么是代理和反向代理:

1.代理：找别人代替你去完成一件你完不成的事情，代理的对象就是客户端

2.反向代理：替厂家卖东西的人就叫做反向代理，代理的对象是服务器端

- 在另一台机器上安装 apache，启动并填写测试页面
- 在 nginx 服务器的配置文件中添加（写在某一网站的 server 标签内）
```bash
location / {
proxy_pass http://apache服务器的IP地址：80;
}
```
重启 nginx，并使用客户端访问测试

 <img width="1015" height="570" alt="Linux：网络服务_43" src="https://github.com/user-attachments/assets/87692bdb-4eaa-4931-9167-9cc2b05ea879" />

### 实验六：负载调度（负责均衡）
- 负载均衡其实就是将任务分摊到多个操作单元上进行执行，例如 web 服务器、FTP 服务器、企业关键应用服务器和其他关键任务服务器等，从而共同完成工作任务。

使用默认的 r 轮训算法，修改配置文件:
```bash
upstream 服务器池名字 {
server apache1服务器的ip地址;
server apache1服务器的ip地址;
}
```

将刚刚的反向代理标签修改
```bash
location / {
proxy_pass http://上面标签的服务器池名字;
proxy_set_header Host $host;
# 重写请求头部，保证网站所有页面都可以访问成功
}
```

重启 nginx ，并使用客户端访问测试

 <img width="1016" height="571" alt="Linux：网络服务_44" src="https://github.com/user-attachments/assets/5fc5d342-f8e5-449b-9126-3afb2f2e0c2f" />

### 实验七：nginx 实现 https （证书+rewrite）
- 安装 nginx 时，需要将 –with-http_ssl_module 模块开启
- 打开 nginx 配置文件，在对应要进行加密的 server 标签中添加一下内容开启 SSL
```bash
server {
......;
ssl on;
ssl_certificate .crt证书存放的路径;
ssl_certificate_key .key证书存放的路径;
ssl_session_timeout 5m;
ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_cipher "EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5";
}
```

- 生成证书和密钥文件
```bash
1.注意：这只是实验环境可以使用命令生成测试，生产环境必须要在 https 证书厂商注册
2.openssl genrsa -out xxxx.key 1024 # 建立服务器私钥，生成 RSA 密钥
3.openssl req -new -key xxxx.key -out xxxx.csr
# 需要依次输入国家、地区、组织、email。最重要的是有一个 common name，可以写你的名字或者域名。如果为了 https 申请，这个必须和域名吻合，否则会引起浏览器警报。生成的 csr 文件交给 CA 签名后形成服务器端子机的证书
4.openssl x509 -req -days 365 -sha256 -in xxxx.csr -signkey xxxx.key -out xxxx.crt
# 生成签字证书
5.cp xxxx.crt 你 nginx 配置文件填写的位置
6.cp xxxx.key 你 nginx 配置文件填写的位置
# 将私钥和证书复制到制定位置
