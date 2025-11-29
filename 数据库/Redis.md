# 1.Redis 简介
## 1.1 非关系型数据库-NoSQL
NoSQL(NoSQL = Not Only SQL )，意为反 SQL 运动，是一项全新的数据库革命性运动，2000 年前就有人提出，发展至 2009 年趋势越发高涨。它是指运用非关系型的数据存储，相对于铺天盖地的关系型数据库运用，这一概念无疑是一种全新的思维的注入。<br>
随着互联网web2.0 网站的兴起，传统的关系数据库在应付 web2.0 网站，特别是超大规模和高并发的 SNS 类型的 web2.0 纯动态网站已经显得力不从心，暴露了很多难以克服的问题，而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。NoSQL 数据库的产生就是为了解决大规模数据集合多重数据种类带来的挑战，尤其是大数据应用难题<br>
<img width="864" height="557" alt="Linux-Redis_1png" src="https://github.com/user-attachments/assets/2df522a3-62cd-4631-bebd-d4fd510bb62d" /><br>

## 1.2 NoSQL 的特性？
NoSQL 是 key-value 形式存储，和传统的关系型数据库不一样,不一定遵循传统数据库的一些基本要求，比如说遵循SQL 标准、ACID 属性、表结构等等。
- 这类数据库主要有以下特点：
  - 非关系型的、分布式、开源的、水平可扩展的
  - 处理超大量数据
  - 击碎了性能瓶颈
  - 对数据高并发读写
  - 对海量数据的高效率存储和访问对数据的高扩展性和高可用性

## 1.3 什么是 Redis？
Redis 是一个开源的，先进的 key-value 存储。它通常被称为数据结构服务器，因为键可以包含 string（字符串）、hash（哈希）、list（链表）、set（集合）和 zset（sorted-set–有序集合）。这些数据类型都支持 push/pop、add/remove 及取交集并集和差集及更丰富的操作<br>
Redis 和Memcached 类似，它支持存储的 value 类型相对更多，与 memcached 一样，为了保证效率，数据都是缓存在内存中，区别是 Redis 会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了 master-slave(主从)同步。<br>
<img width="755" height="425" alt="Linux-Redis_2" src="https://github.com/user-attachments/assets/5fe23cec-9f00-4bc8-9bb7-6a6ea59f161e" /><br>

# 2. Redis 操作
## 2.1 Redis 安装
- Redis 官网
  - http://www.redis.cn/
- 下载安装包
  - wget http://download.redis.io/releases/redis-6.0.6.tar.gz
- 解压 Redis 安装包
  - tar -xf redis-6.0.6.tar.gz
- 安装编译环境
  - yum -y install gcc*
  - yum install centos-release-scl
  - yum install devtoolset-7-gcc*
  - scl enable devtoolset-7 bash
- 安装 Redis
  - cd redis-6.0.6
  - make
  - make PREFIX=/usr/local/redis install # 这是指定安装位置，如果没有指定安装位置 PREFIX=/usr/local/redis，则 make install 会把 redis 安装到 /usr/local/bin/目录下
  - mkdir /usr/local/redis/etc
  - cp ./redis.conf /usr/local/redis/etc/ # 复制Redis 的配置文件到/usr/local/redis/etc/下，便于管理。
- 修改配置文件
  - vi    /usr/local/redis/etc/redis.conf
  - 将文件里的daemonize后面的 no 修改为 yes ，这是设置后台启动
- 启动服务
  - /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf（配置文件路径） # 注意：必须制定配置文件位置，否则会提示警告
- 客户端连接
  - /usr/local/redis/bin/redis-cli
  - -h IP # 连接指定的 redis 服务器
  - -p 6379 # 指定 redis 服务器的端口
  - -a 密码 # 使用密码登录
  - -n 数据库号 # 指定连接那个数据库
  - –raw # Redis 支持存储中文
- 停止 Redis
  - /usr/local/redis/bin/redis-cli shutdown
  - pkill -9 redis

## 2.2 Redis 常用命令
### 2.2.1 string 类型及操作
- string 是最简单的类型，一个 key 对应一个 value，string 类型是二进制安全的。redis 的 string 可以包含任何数据
  - set：设置 key 对应的值为 string 类型
    - 例如：我们添加一个name=atguigu 的键值对应: redis127.0.0.1:6379>set name atguigu
  - setnx：设置key 对应的值为 string 类型，如果 key 已经存在，返回 0，nx 是 not exist 的意思
  - get：获取 key 对应的 string 值，如果 key 不存在返回 nil
  - mset & mget 同时设置和获取多个键值对
  - incrby：对 key 的值做加加（指定值）操作，并返回新的值
  - del：删除一个已创建的key
### 2.2.2 hash 类型及操作
- Redis hash 是一个 string 类型的 field（字段）和 value 的映射表，它的添加、删除操作都是 0(1) 平均；hash 特别适合用于存储对象，相较于将对象的每个字段存成单个string 类型，将一个对象存储在 hash 类型中会占用更少的内存，并且可以更方便的存取整个对象。
  - hset：设置 hash field  为指定值，如果 key 不存在，则先创建。
    - 例如：为num1 表创建一个叫 name 字段（key），键值是 liuchuan: redis127.0.0.1:6379>hset num1 name liuchuan<br>
    <img width="485" height="224" alt="Linux-Redis_03" src="https://github.com/user-attachments/assets/f0cfa80d-eaff-4ed2-ae8d-3bdfa519fba7" /><br>
  - hget、hmset、hmget 意义同上近似
  - hdel：删除制定表中的某一个键值对
  - hgetall：列出表中的所有键值对

### 2.2.3 list 类型及操作
list 是一个链表结构，主要功能是 push、pop、获取一个范围内的所有值等等，操作中 key 理解为链表的名字。Redis 的 list 类型其实就是一个每个子元素都是 string 类型的双向链表。我们可以通过push、pop 操作从链表的头部或尾部添加删除元素。<br>
<img width="512" height="250" alt="Linux-Redis_04" src="https://github.com/user-attachments/assets/675777fc-54b4-4c79-8e0d-a63069ffd39d" /><br>
- lpush：在 key 对应 list 的头部添加字符串元素
  - redis127.0.0.1:6379>lpush zhangsan zhangsan
  - redis127.0.0.1:6379>lpush zhangsan 18
- lrange：从指定链表中获取指定范围的元素
  - redis127.0.0.1:6379>lrange zhangsan 0 -1
  - 0 -1：此范围代表全部元素，意为从头到尾
  - lpush、rpush、lpop、rpop、lrange 详见图示<br>
  <img width="751" height="489" alt="Linux-Redis_05" src="https://github.com/user-attachments/assets/71e50f85-db76-4168-800f-537ac2683179" /><br>

### 2.2.4 Set 类型及操作
set 是集合，他是 string 类型的无序集合。Set 是通过 hash table 实现的，对集 、交集、差集。通过这些操作我们可以实现社交网站中的好友推荐和blog 的 tag 功能。集合不允许有重复值<br>
<img width="677" height="421" alt="Linux-Redis_06" src="https://github.com/user-attachments/assets/c68ef319-1e58-4a63-8e95-2179f0d3486b" /><br>
- sadd：添加一个或多个元素到集合中
  - redis127.0.0.1:6379>sadd mset 1 2 3 4
- smembers：获取集合里面所有的元素
  - redis127.0.0.1:6379> smembers mset
- srem：从集合中删除指定的一个或多个元素
- spop：随机从集合中删除一个元素，并返回
- scard：获取集合里面的元素个数
- sdiff：返回集合 1 与集合 2 的差集。以集合 1 为主
  - redis127.0.0.1:6379>sdiff mset1 mset2
- sinter：获得两个集合的交集
- sunion：获得指定集合的并集

### 2.2.5 zset 类型及操作
zset 是 set 的一个升级版本，它在 set 的基础上增加了一个顺序属性，这一属性在添加修改元素的时候可以指定，每次指定后，zset 会自动重新按新的值调整顺序。可以理解为有两列的 mysql 表，一列存的 value，一列存的顺序。操作中 key 理解为 zset 的名字<br>
<img width="641" height="438" alt="Linux-Redis_07" src="https://github.com/user-attachments/assets/8c95f87e-cd08-4836-8202-01f40fd5c7c4" /><br>
- zadd：向一个指定的有序集合中添加元素，每一个元素会对应的有一个分数。你可以指定多个分数/成员组合。如果一个指定的成员已经在对应的有序集合中了，那么其分数就会被更新成最新的，并且该成员会重新调整到正确的位置，以确保集合有序。分数的值必须是一个表示数字的字符串
  - redis127.0.0.1:6379>zadd zset 2 zhangsan 1 lisi 1 wangwu
- zrange：返回有序集合中，指定区间内的成员。其中成员按照 score（分数）值从小到大排序。具有相同 score 值的成员按照字典顺序来排列
  - redis127.0.0.1:6379>zrange zset 0 -1 withscores 注: withscores 返回集合中元素的同时，返回其分数（score）
- zrem：删除有序集合中指定的值
  redis127.0.0.1:6379>zrem zset zhangsan
- zcard：返回有序集合元素的个数

### 2.2.6 其他相关命令
- keys：按照键名查找指定的键。支持通配符（* ?等）
  redis127.0.0.1:6379>keys h*llo
- exists：确认一个键是否存在（1 表示存在）
- del：删除一个键（通用）
- expire：设置一个键（已存在）的过期时间，如果键已经过期，将会被自动删除
- ttl：以秒为单位，返回指定键的剩余有效时间
  - 当 key 不存在时，返回 -2
  - 当 key 存在但没有设置剩余生存时间时，返回 -1,否则，以秒为单位，返回 key 的剩余生存时间
- select：选择一个数据库，默认连接的数据库是 0，可以支持共 16 个数据库。在配置文件中，通过 databases 16 关键字定义
- move：将当前数据库的键移动到指定的数据库中
- type：返回键的类型
- dbsize：返回当前库中键的数量（所有类型）
- save：保存所有的数据。很少在生产环境直接使用 SAVE 命令，因为它会阻塞所有的客户端的请求，可以使用 BGSAVE 命令代替. 如果在BGSAVE 命令的保存数据的子进程发生错误的时,用 SAVE 命令保存最新的数据是最后的手段
- info：获取服务器的详细信息
- config get：获取 redis 服务器配置文件中的参数。支持通配符
- flushdb：删除当前数据库中所有的数据
- flushall：删除所有数据库中的所有数据

## 2.3 Redis 高级应用
### 2.3.1 密码防护
- 给 redis 服务器设置密码
  - 修改配置文件
  - vi /usr/local/redis/etc/redis.conf
    - 找到 requirepass<br>
    <img width="768" height="760" alt="Linux-Redis_08" src="https://github.com/user-attachments/assets/97edbdb2-1e34-4c5a-b549-b041abcbd86a" /><br>
    - 复制并且黏贴 然后改为 requirepass 密码即可
  - 重启 redis
    - pkill redis
    - redis-server redis安装路径/etc/redis.conf
  - 客户端登录
    - redis安装路径/bin/redis-cli -a 你在配置文件设置的密码
    - 或者 redis127.0.0.1:6379> auth 密码

### 2.3.2 主从同步
- Redis 主从复制过程
  - Slave 与 master 建立连接，发送 sync 同步命令
  - Master 会启动一个后台进程，将数据库快照保存到文件中，同时 master 主进程会开始收集新的写命令并缓存
  - 后台完成保存后，就将此文件发送给 slave
  - Slave 将此文件保存到硬盘上
- 主服务器给自己设置好密码即可（iptables&SELinux 关闭）
  - 从服务器修改配置文件，用来连接主服务器
  - 注意：从服务器也需要安装 redis
    - 老版本
      - 从服务器
        - slaveof <masterip> <msterport>           #主服务器的 IP 和端口
        - masterauth <masterpass>    #主服务器的密码(主服务器要设置好密码)
    - 新版本 redis 5.* 以上
      - 主服务器
        - 找到 bind 127.0.0.1 注释掉，或者修改为本机的 IP 地址(重启)
      - 从服务器
        - replicaof    <masterip> <msterport>      #主服务器的 IP 和端口
        - masterauth <masterpass>    #主服务器的密码(主服务器要设置好密码)
  - 重启从服务器，然后测试（可通过 info 命令获取当前服务器身份类型）

### 2.3.3 数据持久化
Redis 是一个支持持久化的内存数据库，也就是说需要经常将内存中的数据同步到硬盘来保证持久化
- snapshotting（快照）–默认方式
  - RDB 持久化方式能够在指定的时间间隔能对你的数据进行快照存储。是默认的持久化方式。这种方式是将内存中数据以快照的方式写入到二进制文件中，默认的文件名为 dump.rdb。这种  持久化方式被称为快照 snapshotting（快照）
    - 过了 900 秒并且有 1 个key 发生了改变 就会触发save 动作
    - 过了 300 秒并且有 10 个 key 发生了改变 就会触发 save 动作
    - 过了 60 秒并且至少有 10000 个 key 发生了改变 也会触发 save 动作
  - 在 redis.conf 文件中 dir ./定义了数据库文件的存放位置，默认是当前目录。所以每次重启 redis 服务所在的位置不同，将会生成新的 dump.rdb 文件；建议服务器搭建完成时先修改快照文件保存位置
    - vim redis安装路径/etc/redis.conf
      - 找到 dir ./<br>
      <img width="762" height="734" alt="Linux-Redis_09" src="https://github.com/user-attachments/assets/b8f53e85-ee05-4906-9988-0626c9221747" /><br>
      - 把 ./ 修改成要存放的路径
- append-only file（缩写 aof）
  - 使用AOF 会让你的Redis 更加耐久: 你可以使用不同的持久化策略：每次写的时候备份、每秒备份、无备份。使用默认的每秒备份策略,Redis 的性能依然很好(备份是由后台线程进行处理的,主线程会尽力处理客户端请求),一旦出现故障，你最多丢失 1 秒的数据
  - 打开 redis.conf 配置文件开启 AOF 持久化
    - vim redis安装路径/etc/redis.conf
    - appendonly no 默认不使用 AOF 持久化（450 行）将no 改成 yes
    - appendfsync always 有写操作，就马上写入磁盘。效率最慢，但是最安全
    - appendfsync everysec  默认，每秒钟写入磁盘一次
    - appendfsync no 不进行 AOF 备份，将数据交给操作系统处理。最快，最不安全
    - 测试：重启 redis 服务，登录 client 添加一个键值，退出然后 ls 命令查看下是否生成 appendonly.aof
      - 可以用 cat 查看

## 2.4 MySQL+NoSQL(Redis)
- 安装 gcc 编译环境
  - yum -y install gcc*
  - yum install centos-release-scl
  - yum install devtoolset-7-gcc*
  - scl enable devtoolset-7 bash
- 安装需要的软件包
  - 文件在服务器的 root 目录下
- 解压文件并进入文件夹
  - tar -xf redis-mysql.tar.gz
  - cd redis-mysql
  - yum -y install *
- 配置网站 nginx 并启动 nginx
  - vim /etc/nginx/nginx.conf<br>
  <img width="419" height="66" alt="Linux-Redis_10" src="https://github.com/user-attachments/assets/84e478ce-8868-4bf5-b214-c40f8d825323" /><br>
  - vim /etc/nginx/conf.d/default.conf<br>
  <img width="770" height="471" alt="Linux-Redis_11" src="https://github.com/user-attachments/assets/71e40b6a-634a-41bc-9091-4cd87d178e9c" /><br>
  - 启动 nginx
  - service nginx start
- 修改 php-fpm 配置文件
  - vim /etc/php-fpm.d/www.conf<br>
  <img width="764" height="209" alt="Linux-Redis_13" src="https://github.com/user-attachments/assets/e7cd6353-305d-41c2-b305-f03a554a8178" /><br>
- 启动 php-fpm 和数据库
  - service php-fpm start
  - service mysqld start
  - mysql_secure_installation # mysql 初始化
- 测试网站和 php 的连通性
  - mkdir /www
  - vim /www/index.php
```bash
<? php
phpinfo();
```
<img width="767" height="417" alt="Linux-Redis_14" src="https://github.com/user-attachments/assets/b079cba5-6501-4405-9fcd-525395f84b8e" /><br>

- 安装 redis
  - wget http://download.redis.io/releases/redis-6.0.6.tar.gz
  - tar -xf redis-6.0.6.tar.gz
  - cd redis-6.0.6
  - make
  - make PREFIX=/usr/local/redis install # 这是指定安装位置，如果没有指定安装位置 PREFIX=/usr/local/redis，则 make install 会把 redis 安装到 /usr/local/bin/目录下
  - mkdir /usr/local/redis/etc
  - cp ./redis.conf /usr/local/redis/etc/ # 复制Redis 的配置文件到/usr/local/redis/etc/下，便于管理。
  - vi /usr/local/redis/etc/redis.conf
    - 将文件里的daemonize后面的 no 修改为 yes ，这是设置后台启动
- 安装提供 php 和 redis 联系的软件<br>
<img width="768" height="171" alt="Linux-Redis_15" src="https://github.com/user-attachments/assets/29dd9097-877a-45ae-a8ab-82c5821702d2" /><br>
  - 安装<br>
  <img width="759" height="331" alt="Linux-Redis_16" src="https://github.com/user-attachments/assets/5c1d5677-3190-450e-8dfa-f488bd34f387" /><br>
- 让 php 支持 redis
  - vim /etc/php.ini<br>
  <img width="767" height="235" alt="Linux-Redis_17" src="https://github.com/user-attachments/assets/a488b324-0509-4221-87e8-c61ab2fbc5ee" /><br>
  - service php-fpm restart
  - 在浏览器输入服务器 IP 地址看看测试页面有没有 redis 这个模块<br>
  <img width="761" height="99" alt="Linux-Redis_18" src="https://github.com/user-attachments/assets/f351069e-272d-4add-8a84-331b9842b68c" /><br>
- 进入 mysql
  - mysql -uroot -p
  - mysql>create database mytest; # 创建 mystest 这个库
  - mysql>use mytest # 选择 mystest 这个库
  - mysql>create table test(id int,name char(20)); # 创建 test 这个表
  - mysql>insert into test values (1,’wang’),(2,’zhang’),(3,’li’),(4,’liu’),(5,’zhu’); # 往 test 这个表里插入数据
  - select * form test; # 查看 test 这个表里的所有数据
- 编写redis脚本
  - vim /www/redis-mysql.php
```bash
<?php
		ini_set("display_errors", "On");
		error_reporting(E_ALL | E_STRICT);
		//开启debug 
	
	
		// mysql 库: mytest 表：test 
        $redis = new redis();
        $redis->connect('127.0.0.1',6379);
        $query = "select * from test limit 5";
        for ($key=1;$key<=5;$key++)
        {
                if (!$redis->get($key))
				//判断redis中是否有1 2 3 4 5 的键，没有连接数据库查询mytest库的test表，然后插入到redis中
                {
                        $connect = mysql_connect('127.0.0.1','root','123456');
                        mysql_select_db(mytest);
                        $result = mysql_query($query);
						var_dump ($result);
                        while ($row = mysql_fetch_assoc($result))
                        {
                                $redis->setex($row['id'],30,$row['name']);
								//从MySQL中获取的资源插入到redis中，并设置有效时间为30s
                        }
                        $myserver = 'mysql';
                        break;
                }
                else
				//判断redis中是否有1 2 3 4 5 的键，有直接打印redis中的 1 2 3 4 5 键的值。
                {
                        $myserver = "redis";
                        $data[$key] = $redis->get($key);
                }
        }
                        echo $myserver;
                        echo "<br>";
                        for ($key=1;$key<=5;$key++)
                        {
                                echo "number is <b><font color=#FF0000>$key</font></b>";
                                echo "<br>";
                                echo "name is <b><font color=#FF0000>$data[$key]</font></b>";
                                echo "<br>";
                        }
?>
```
- 开启 redis
  - /usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf（配置文件路径）
- 在浏览器输入服务器 IP/redis-mysql.php 查看<br>
<img width="1091" height="380" alt="Linux-Redis_19" src="https://github.com/user-attachments/assets/44cae47a-feaa-4224-b7e5-273a8bb67ddc" />


















