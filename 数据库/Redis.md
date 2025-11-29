# 1.Redis 简介
## 1.1 非关系型数据库-NoSQL
NoSQL(NoSQL = Not Only SQL )，意为反 SQL 运动，是一项全新的数据库革命性运动，2000 年前就有人提出，发展至 2009 年趋势越发高涨。它是指运用非关系型的数据存储，相对于铺天盖地的关系型数据库运用，这一概念无疑是一种全新的思维的注入。<br>
随着互联网web2.0 网站的兴起，传统的关系数据库在应付 web2.0 网站，特别是超大规模和高并发的 SNS 类型的 web2.0 纯动态网站已经显得力不从心，暴露了很多难以克服的问题，而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。NoSQL 数据库的产生就是为了解决大规模数据集合多重数据种类带来的挑战，尤其是大数据应用难题
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
Redis 和Memcached 类似，它支持存储的 value 类型相对更多，与 memcached 一样，为了保证效率，数据都是缓存在内存中，区别是 Redis 会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了 master-slave(主从)同步。
<img width="755" height="425" alt="Linux-Redis_2" src="https://github.com/user-attachments/assets/5fe23cec-9f00-4bc8-9bb7-6a6ea59f161e" />

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
    - 例如：为num1 表创建一个叫 name 字段（key），键值是 liuchuan: redis127.0.0.1:6379>hset num1 name liuchuan
    <img width="485" height="224" alt="Linux-Redis_03" src="https://github.com/user-attachments/assets/f0cfa80d-eaff-4ed2-ae8d-3bdfa519fba7" />
  - hget、hmset、hmget 意义同上近似
  - hdel：删除制定表中的某一个键值对
  - hgetall：列出表中的所有键值对
















