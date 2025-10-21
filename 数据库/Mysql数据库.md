# 数据库基础理论
## 什么是数据库？
- 数据：描述事物的符号记录，可以是数字、文字、图形、图像、声音、语言等，数据有多种形式，它们都可以经过数字化后存入计算机。
- 数据库：存储数据的仓库，是长期存放在计算机内、有组织、可共享的大量数据的集合。数据库中的数据按照一定数据模型组织、描述和存储，具有较小的冗余度，较高的独立性和易扩展性，并为各种用户共享，总结为以下几点：
1.数据结构化 
2.数据的共享性高，冗余度低，易扩充
3.数据独立性高
4.数据由 DBMS 统一管理和控制（安全性、完整性、并发控制、故障恢复） # DBMS：数据库管理系统（能够操作和管理数据库的大型软件）

## 数据库与文件系统的区别？
- 文件系统：文件系统是操作系统用于明确存储设备（常见的是磁盘）或分区上的文件的方法和数据结构；即在存储设备上组织文件的方法。操作系统中负责管理和存储文件信息的软件机构称为文件管理系统，简称文件系统。
- 数据库系统：数据库管理系统(Database Management System)是一种操纵和管理数据库的大型软件， 用于建立、使用和维护数据库，简称 DBMS。它对数据库进行统一的管理和控制，以保证数据库的安全性和完整性。
- 对比区别 
    
    1.管理对象不同：文件系统的管理对象是文件，并非直接对数据进行管理，不同的数据结构需要使用不同的文件类型进行保存（举例：txt 文件和 doc 文件不能通过修改文件名完成转换）；而数据库直接对数据进行存储和管理
    
    2.存储方式不同：文件系统使用不同的文件将数据分类（.doc/.mp4/.jpg）保存在外部存储上；数据库系统使用标准统一的数据类型进行数据保存（字母、数字、符号、时间）
    
    3.调用数据的方式不同：文件系统使用不同的软件打开不同类型的文件；数据库系统由 DBMS 统一调用和管理。
 
 <img width="1004" height="392" alt="Linux：网络服务_51" src="https://github.com/user-attachments/assets/14e8114f-b333-442f-bcb5-e283459718d0" />

 - 优缺点总结
    1.由于 DBMS 的存在，用户不再需要了解数据存储和其他实现的细节，直接通过 DBMS 就能获取数据， 为数据的使用带来极大便利。
    2.具有以数据为单位的共享性，具有数据的并发访问能力。DBMS 保证了在并发访问时数据的一致性。
    3.低延时访问，典型例子就是线下支付系统的应用，支付规模巨大的时候，数据库系统的表现远远优于文件系统。
    4.能够较为频繁的对数据进行修改，在需要频繁修改数据的场景下，数据库系统可以依赖 DBMS 来对数据进行操作且对性能的消耗相比文件系统比较小。
    5.对事务的支持。DBMS 支持事务，即一系列对数据的操作集合要么都完成，要么都不完成。在 DBMS上对数据的各种操作都是原子级的。

## 常见数据库有哪些？

- 关系型数据库
    
    1.关系数据库是建立在关系模型基础上的数据库，借助于集合代数等数学概念和方法来处理数据库中的数据。现实世界中的各种实体以及实体之间的各种联系均用关系模型来表示。简单说，关系型数据库是由多张能互相联接的二维行列表格组成的数据库
    
    2.关系模型就是指二维表格模型，因而一个关系型数据库就是由二维表及其之间的联系组成的一个数据组织。当前主流的关系型数据库有 Oracle、DB2、Microsoft SQL Server、Microsoft Access、MySQL、浪潮 K-DB 等
    
    3.实体关系模型简称 E-R 模型，是一套数据库的设计工具，他运用真实世界中事物与关系的观念， 来解释数据库中的抽象的数据架构。实体关系模型利用图形的方式（实体-关系图）来表示数据库的概念设计，有助于设计过程中的构思及沟通讨论

- 非关系型数据库
    
    非关系型数据库：又被称为 NoSQL（Not Only SQL )，意为不仅仅是SQL，是一种轻量、开源、不兼容 SQL 功能的数据库，对 NoSQL 最普遍的定义是“非关联型的”，强调 Key-Value 存储和文档数据库的优点，而不是单纯地反对 RDBMS（关系型数据库管理系统）

- 关系型数据库（mysql）的特征及组成结构介绍
        
  -关系型数据库发展历程：
    
    1.层次模型
<img width="500" height="148" alt="Linux：网络服务_52" src="https://github.com/user-attachments/assets/a2e29117-832b-45cf-995e-421aad6f8a79" />
    
    2.网状模型
<img width="261" height="186" alt="Linux：网络服务_53" src="https://github.com/user-attachments/assets/744298b2-af5e-40f1-9b4f-7e590e006262" />
 
 
 - 关系模型(Relation)

    1.关系模型以二维表结构来表示实体与实体之间的联系，关系模型的数据结构是一个“二维表框架”组成的集合。每个二维表又可称为关系。在关系模型中，操作的对象和结果都是二维表。
    2.关系模型是目前最流行的数据库模型。支持关系模型的数据库管理系统称为关系数据库管理系 统，Access 就是一种关系数据库管理系统。图所示为一个简单的关系模型，其中图(a)所示为关系模式，图(b)所示为这两个关系模型的关系，关系名称分别为教师关系和课程关系，每个关系均含 3 个元组，其主码均为“教师编号”。
    3.在关系模型中基本数据结构就是二维表，不用像层次或网状那样的链接指针。记录之间的联系是通过不同关系中同名属性来体现的。例如，要查找“刘晋”老师所上的课程，可以先在教师关系中根据姓名找到教师编号“1984030”，然后在课程关系中找到“1984030”任课教师编号对应的课程名即可。通过上述查询过程，同名属性教师编号起到了连接两个关系的纽带作用。由此可见，关系模型中的各个关系模式不应当是孤立的，也不是随意拼凑的一堆二维表，它必须满足相应的要求。

<img width="261" height="186" alt="Linux：网络服务_53" src="https://github.com/user-attachments/assets/2464374b-cca0-4bc1-8acf-647f6a131f2a" />

- 关系式数据库的组成结构和名词解释
1.数据以表格的形式出现，每行为单独的一条记录，每列为一个单独的字段，许多的记录和字段组成一张表单（table），若干的表单组成库（database）
2.记录(一条数据)：在数据库当中，表当中的行称之为记录
3.字段(id name)：在数据库当中，表当中的列称之为字段
4.MySQL 数据类型
    4.1.数据类型用于指定特定字段所包含数据的规则，它决定了数据保存在字段里的方式，包括分配给字段的宽度，以及值是否可以是字母、数字、日期和时间等。任何数据或数据的组合都有对应的数据类型，用于存储字母、数字、日期和时间、图像、二进制数据等。数据类型是数据本身的特征，其特性被设置到表里的字段。
    4.2.MySQL 常见基础数据类型：
        
        4.2.1.字符串类型（CHAR（0-255 固定长度）,VARCHAR（0-255 可变长度）

        4.2.2.数值类型（INT（整数型）、FLOAT（浮点型））

        4.2.3.日期和时间类型（DATE（年月日）、TIME（时分秒））
5.MySQL 约束类型
    5.1.约束是一种限制，它通过对表的行或列的数据做出限制，来确保表的数据的完整性、唯一性
      
    5.2.主键约束 primary key：主键约束相当于唯一约束+非空约束的组合，主键约束列不允许重复，也不允许出现空值。每个表最多只允许一个主键，建立主键约束可以在列级别创建，也可以在表级别创建。当创建主键的约束时，系统默认会在所在的列和列组合上建立对应的唯一索引。
      
    5.3.外键约束 foreign key：外键约束是保证一个或两个表之间的参照完整性，外键是构建于一个表的两个字段或是两个表的两个字段之间的参照关系。
      
    5.4.唯一约束 unique：唯一约束是指定 table 的列或列组合不能重复，保证数据的唯一性。唯一约束不允许出现重复的值，但是可以为多个 null。同一个表可以有多个唯一约束，多个列组合的约束。在创建唯一约束时，如果不给唯一约束名称，就默认和列名相同。唯一约束不仅可以在一个表内创建，而且可以同时多表创建组合唯一约束。

    5.5.非空约束 not null 与默认值 default：非空约束用于确保当前列的值不为空值，非空约束只能出现在表对象的列上。Null 类型特征：所有的类型的值都可以是 null，包括 int、float 等数据类型。

6.MySQL 索引

    索引是一个单独的、物理的数据库结构，它是某个表中一字段或若干字段值的集合。表的存储由两部分组成，一部分用来存放数据，另一部分存放索引页面。通常，索引页面相对于数据页面来说小得多。数据检索花费的大部分开销是磁盘读写，没有索引就需要从磁盘上读表的每一个数据页，如果有索引，则只需查找索引页面就可以了。所以建立合理的索引，就能加速数据的检索过程。

7.MySQL 锁
    
    7.1.数据库是一个多用户使用的共享资源。当多个用户并发地存取数据时，在数据库中就会产生多个事务同时存取同一数据的情况。若对并发操作不加控制就可能会读取和存储不正确的数据，破坏数据库的一致性。

    7.2.加锁是实现数据库并发控制的一个非常重要的技术。当事务在对某个数据对象进行操作前，先向系统发出请求，对其加锁。加锁后事务就对该数据对象有了一定的控制，在该事务释放锁之前，其他的事务不能对此数据对象进行更新操作。

8.MySQL 的存储引擎
    
    8.1.存储引擎就是存储数据，建立索引，更新查询数据等等技术的实现方式。存储引擎是基于表的， 而不是基于库的。所以存储引擎也可被称为表类型。Oracle，SqlServer 等数据库只有一种存储引擎。MySQL 提供了插件式的存储引擎架构。所以 MySQL 存在多种存储引擎，可以根据需要使用相应引擎， 或者编写存储引擎。
    
    8.2.MYISAM：默认引擎、插入和查询速度较快，支持全文索引，不支持事务、行级锁和外键约束等功能

    8.3.INNODB：支持事务、行级锁和外键约束等功能

    8.4.MEMORY：工作在内存中，通过散列字段保存数据，速度快、不能永久保存数据

9.事务（Transaction）是并发控制的基本单位。

    9.1.可以把一系列要执行的操作称为事务，而事务管理就是管理这些操作要么完全执行，要么完全不执行

    9.2.经典案例：银行转账工作，从一个账号扣款并使另一个账号增款，这两个操作要么都执行，要么都不执行。所以，应该把它们看成一个事务。事务是数据库维护数据一致性的单位，在每个事务结束时，都能保持数据一致性

    9.3.注意：mysql 中并不是所有的数据引擎都支持事务管理的，只有 innodb 支持事务管理。

# MySQL 基础操作

- MySQL 常见版本
    
    1.MySQL Community Server 社区版本，开源免费，但不提供官方技术支持。

    2.MySQL Enterprise Edition 企业版本，需付费，可以试用 30 天。

    3.MySQL Cluster 集群版，开源免费。可将几个 MySQL Server 封装成一个 Server。

    4.MySQL Cluster CGE 高级集群版，需付费

- MySQL 安装部署
    
    MySQL：MySQL 客户端程序
   
    MySQL-Server：MySQL 服务器端程序

    1.源代编译安装
    
    编译工具：configure、cmake、make
      
      数据库常用的配置选项：
      
      ```bash
      -DCMAKE_INSTALL_PREFIX=/PREFIX——–指定安装路径（默认的就是/usr/local/mysql）
      -DMYSQL_DATADIR=/data/mysql——–mysql 的数据文件路径
      -DSYSCONFDIR=/etc——–配置文件路径
      -DWITH_INNOBASE_STORAGE_ENGINE=1——–使用INNOBASE 存储引擎
      -DWITH_READLINE=1——–支持批量导入 mysql 数据
      -DWITH_SSL=system——–mysql 支持 ssl
      -DWITH_ZLIB=system——–支持压缩存储
      -DMYSQL_TCP_PORT=3306——–默认端口 3306
      -DENABLED_LOCAL_INFILE=1——–启用加载本地数据
      -DMYSQL_USER=mysql——–指定 mysql 运行用户
      -DMYSQL_UNIX_ADDR=/tmp/mysql.sock——–默认套接字文件路径
      -DEXTRA_CHARSETS=all——–是否支持额外的字符集
      -DDEFAULT_CHARSET=utf8——–默认编码机制
      -DWITH_DEBUG=0 DEBUG——–DEBUG 功能设置
      ```

    2.常见资料

        服务：mysqld
        端口：3306
        主配置文件：/etc/my.cnf
        初始化脚本：mysql_install_db
        启动命令：mysqld_safe
        yum 安装默认数据目录 ：/var/lib/mysql
        套接字文件：/var/lib/mysql/mysql.sock
        进程文件：/var/run/mysqld/mysqld.pid # 当意外关闭数据库时，再开启时假如开启不了，找到这个，删除再启动

    3.MySQL 登录及退出命令
        
        设置密码：mysqladmin  -uroot password 密码
        登录：mysql -u 用户名 -p 密码 -P 端口 -S 套接字文件
        -p 用户密码 
        -h 登陆位置（主机名或 ip 地址）
        -P 端口号（3306 改了就不是了）
        -S 套接字文件（/var/lib/mysql/mysql.sock）
        退出:
        exit
        ctrl+d

    4.MySQL 管理命令
        
        创建登录用户: mysql>create user 用户名@’%’ identified by ‘密码’;  # %:指任意的远程终端
        测试用户登录: mysql -u用户名 -p密码 -h mysql服务器IP地址
        用户为自己更改密码: mysql>set password=password(‘新密码’);
        root 用户为其他用户找回密码: mysql>set password for 用户ming@’%’=password(‘新密码’);
        root 找回自己的密码并修改: 
            关闭数据库，修改主配置文件（/etc/my.cnf）添加：skip-grant-tables
            vim /etc/my.cnf
            在 symbolic-links=0 下面添加 skip-grant-tables
            启动数据库，空密码登录并修改密码
            mysql -uroot
            mysql>update mysql.user set password=password(‘新密码’) where user=’root’;
            删除 skip-grant-tables,重启数据库验证新密码
        创建查询数据库
            mysql>create database 库名; 创建数据库
            mysql>show databases 库名; 查看数据库
        创建数据表
            mysql>use 库名; 选择要使用的数据库
            mysql>create table 表名(id int ,name char(30)); 创建表，并添加 id 和 name 字段以及类型
            mysql>describe 表名; 查看表结构（字段）
        创建复杂点的数据表
            mysql>create table 表名 (->id int unsigned not null auto_increment, # 字段要求为正数、且自增长、主键
            ->name char(30) not null default ‘ ‘,  # 字符型长度 30 字节，默认值为空格
            ->age int not null default 0, # 字段默认值为 0
            ->primary key (id)); # 设置 id 为主键
            mysql>describe 表名; # 查看表结构（字段）
        插入数据
            先选择要插入数据表所在的库
            mysql>insert into 表名 (id,name,age) values (1,’zhangsan’,21)； # 指明插入字段和数据
            mysqll>select * from 表名; # 查询表的所有数据
            mysql>insert into 表名 values (2,’lisi’,20)； # 按顺序插入指定字段
            mysql>insert into 表名 values (3,’wangwu’)； # 未声明年龄
            mysql>insert into 表名 values (4,’zhao’,19)，(5,’sun’,25)； #插入多条数据
        将表 a2 的数据复制到表 b1
            mysql>select * from a1; # 查询表的所有数据
            mysql>insert into a1 (id,name) select id,name from a2; # 查询 a2 值，并写入到 a1
            mysql>select * from a1;
        删除数据库
            mysql>drop database 库名;
            mysql>show databases;
        删除数据表
            mysql>drop table 表名;
            mysql>show table;
        删除表里的数据记录
            mysql>delete from 表名 where id=5; # 删除 id=5 的记录
            mysql>delete from 表名 where between 23 and 25;    #删除年龄在 23-25 之间的
        注意：库和表的删除用 drop,记录删除用 delete
        修改表中的数据
            mysql>update 表名 set age=21 where id=3;
        修改数据表的名称
            mysql>alter table 旧表名 rename 新表名;
        修改数据表的字段类型
            mysql>describe 表名;
            mysql>alter table 表名 modify name char(50); # 将表里的 name 字段类型该为char(50)
            describe 表名;
        修改数据表的字段类型详情
            mysql>describe 表名;
            mysql>alter table 表名 change name username char(50) not null default ‘ ‘; # 将表里的 name 字段改名为 username，字段类型char(50)，默认为空
            mysql>describe 表名;
        添加字段
            mysql>describe 表名;
            mysql>alter table 表名 add time datetime; # 给表添加一个 time 字段，字段类型是 datetime
            mysql>describe 表名; # 添加位置默认在末尾
            mysql>alter table 表名 add birthday year first; # 将字段 birthday，字段类型 year 添加表的字段到第一列
            mysql>alter table 表名 add sex nchar(1) after id; # 将字段 sex ，字段类型 nchar(1) 添加到表的指定字段 id 后
        删除字段
            mysql>alter table 表名 drop 字段名;
        Mysql 用户授权
            mysql>select user from mysql.user;
            mysql>grant all on 库名.表名 to 用户名@’%’；# 给已存在用户授权
            mysql>grant all on 库名.表名 to 用户名@’%’ identified by ‘密码’; # 创建用户并授权
        取消 abc 用户的删除库、表、表中数据的权限
            mysql>revoke drop,delete on 库名.表名 from abc@’%’； # 取消删除权限（登录 abc 测试）
            mysql>show grants for abc@’%’； # 查看指定用户的授权
            mysql>show grants for 其他用户@’%’；

    5.备份和还原
        
        5.1mysqldump 备份
            备份
            mysqldump -u 用户名 -p 数据库名 > /备份路径/备份文件名（备份整个数据库）
            mysqldump -u 用户名 -p 数据库名 表名 > /备份路径/备份文件名（备份数据表）
            备份多个库：–databases 库 1，库 2
            备份所有库：–all-databases
            备份多个表：库名 表 1 表 2
            还原
            mysql 数据库 < 备份文件
            注意：还原时，若导入的是某表，请指定导入到哪一个库中
        mysqlhotcopy 备份
            备份
            mysqlhotcopy –flushlog -u=’用户’ -p=’密码’ –regexp=正则 备份目录
            还原
            cp -a 备份目录 数据目录（/var/lib/mysql）
        mysqldump 和 mysqlhotcopy 示例
             mysqldump
                 把数据库 aa 备份到/root 目录下
                 mysqldump –uroot –p aa > ~/aa.sql
                 模拟数据库 aa 丢失（删除数据库 aa）
                     mysql -uroot -p 密码
                     mysql>drop database aa;
                通过 aa.sql 文件还原（指定导入到哪个库中）
                    mysql –uroot –p 库名 < aa.sql
                备份多个数据库（–databases）
                    mysqldump –uroot –p –databases aa test > abc.sql
                    还原（先模拟丢失）
                        mysql –uroot –p < abc.sql
            mysqlhotcopy
                备份有规则的数据库
                    mysql -uroot -p密码
                    mysql>create database a1; # 连续创建三个a 开头的数据库
                    mysqlhotcopy –flushlog –u=‘root’ –p=‘456’ –regexp=^a # 备份以 a 开头的数据库
                还原（先模拟丢失）
                    mysql -uroot -p 密码
                    mysql>drop database a1;
                    cp –a /你执行备份所在的路径 /var/lib/mysql/ # 复制产生的文件到数据库目录下
            mysql-binlog 日志备份
                二进制日志（log-bin 日志）：所有对数据库状态更改的操作（create、drop、update 等）
                修改 my.cnf 配置文件开启 binlog 日志记录功能
                    vim /etc/my.cnf 添加下面选项
                    log-bin=mysql-bin #启动二进制日志
                按时间还原
                –start-datetime
                –stop-datetime
                格式
                    mysqlbinlog –start-datetime ‘YY-MM-DD HH:MM:SS’ –stop-datetime ‘YY-MM-DDHH:MM:SS’ 二进制日志 | mysql -uroot -p
                按文件大小还原
                    –start-position
                    –stop-position
                mysql-binlog 日志备份示例
                    开启二进制日志
 <img width="416" height="165" alt="Linux：网络服务_55" src="https://github.com/user-attachments/assets/e0b37447-a241-45ed-8f52-ea27931cd1bb" />





