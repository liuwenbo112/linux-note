#### MySql(分支 mariadb)

https://www.cnblogs.com/pyyu/articles/9467289.html

##### 1、安装mariadb

	-yum
	-源码编译安装
	-下载rpm安装
	yum和源码编译安装的区别？
	    1.路径区别-yum安装的软件是他自定义的，源码安装的软件./configure --preifx=软件安装的绝对路径
	    2.yum仓库的软件，版本可能比较低，而源码编译安装，版本可控
	    3.编译安装的软件，支持第三方功能扩展./configure  这里可以加上很多参数，定制功能
			
	yum仓库的区别
		1.阿里云的yum仓库
		2.假设mysql官网，也会提供rpm包，源码包，以及yum源，供给下载
##### 2、配置mariadb的官方yum源，用于自动下载mariadb的rpm软件包，自动安装

	注意点：阿里云提供的yum仓库，和epel源仓库，它也有mariadb，但是版本可能会很低
	
	这个是yum默认的mariadb的版本信息
	mariadb   x86_64      1:5.5.60-1.el7_5      base         8.9 M
	那我们就得选用mariadb的官方yum源，

##### 3、配置官方的mariadb的yum源，手动创建 mariadb.repo仓库文件  **

```
touch /etc/yum.repos.d/mariadb.repo 
然后在mariadb.repo写入如下内容
[mariadb]
name = MariaDB
baseurl = http://mirrors.ustc.edu.cn/mariadb/yum/10.3/centos6-x86/
gpgkey=http://mirrors.ustc.edu.cn/mariadb/yum/RPM-GPG-KEY-MariaDB
gpgcheck=1

检查系统中是否存在 mysql-libs-*.*.*.*
[root@localhost ~]# rpm -qa | grep -i mysql
若有则删除否则跳过此步骤：
rpm -e --nodeps mysql-libs-5.1.71-1.el6.x86_64 
```

##### 4、通过yum安装mariadb软件

```
安装mariadb服务端和客户端  (由于是国外镜像源，因此下载速度可能很慢)
yum install MariaDB-server MariaDB-client -y
```

##### 5、安装完成后，启动mariadb服务端

```
service  mysql start/stop/restart/status  
service  mysql enable    开机启动mysql

当有MariaDB server PID file could not be found!                [失败] 时
则先杀死所有的mysql进程之后再启动

```

##### 6、mysql初始化

```
mysql_secure_installation   这条命令可以初始化mysql，删除匿名用户，设置root密码等等....

具体步骤：看博客https://www.cnblogs.com/pyyu/articles/9467289.html
```

##### 7、mysql常用命令

连接mysql:mysql -u root -p  # root可以换成其他用户

密码：123456

	desc  查看表结构
	create database  数据库名
	create table  表名
	show create  database  库名  		查看如何创建db的
	show create table 表名;			查看如何创建table结构的
	
	#修改mysql的密码
	set password = PASSWORD('redhat');
	
	#创建mysql的普通用户，默认权限非常低 ,%表示所有的主机ip地址均可以登录，by后面是登录的密码
	create user yining@'%' identified by 'yiningzhenshuai';
	登录时输入：mysql -uyining -p
	密码是：yiningzhenshuai
	
	#查询mysql数据库中的用户信息
	use mysql;
	select host,user,password  from user;
##### 8、给（普通）用户添加权限命令

先切换到root用户登录mysql,进行权限设置

```
grant 权限 on 数据库.表名 to 账户@主机名            对特定数据库中的特定表授权
grant 权限 on 数据库.* to 账户@主机名            　　对特定数据库中的所有表给与授权
grant 权限1,权限2,权限3 on *.* to 账户@主机名   　　 对所有库中的所有表给与多个授权
grant all privileges on *.* to 账户@主机名   　　 对所有库和所有表授权所有权限

举例：
[root@master ~]# mysql -uroot -p

MariaDB [(none)]> use mysql;

MariaDB [(none)]> grant all privileges on *.* to yuchao@127.0.0.1;

MariaDB [mysql]> show grants for yuchao@127.0.0.1;

flush privileges;		刷新授权表
```

##### 9、授予远程登录的权限命令				

```
(root不能远程登录的问题？？)
grant all privileges on *.* to yining@'%';  给yining用户授予所有权限

grant all privileges on *.* to root@'%' identified by 'redhat';  #给与root权限授予远程登录的命令
```

##### 10、数据库的数据备份与恢复 

mysqldump指令是linux的命令不是mysql的命令，在数据备份的时候退出数据库页面

###### 1、mysqldump命令用于备份数据库数据

```
导出当前数据库的所有db，到一个自定义文件中
[root@master ~]# mysqldump -u root -p --all-databases > /data/AllMysql.dump		

2.登录mysql 导入数据
	mysql -u root -p 
	>   source /data/AllMysql.dump 
	
3.通过命令导入数据（与2功能类似）
mysql -uroot -p   <   /data/AllMysql.dump  #在登录时候，导入数据文件，一样可以写入数据
然后重新登录数据库即可
```

###### 2、导出db1、db2两个数据库的所有数据

```
mysqldump -uroot -proot --databases db1 db2 >/tmp/user.sql
```

##### 11、MYSQL主从复制

```
MySQL数据库的主从复制方案，是其自带的功能，并且主从复制并不是复制磁盘上的数据库文件，而是通过binlog日志复制到需要同步的从服务器上。

MySQL数据库支持单向、双向、链式级联，等不同业务场景的复制。在复制的过程中，一台服务器充当主服务器（Master），接收来自用户的内容更新，而一个或多个其他的服务器充当从服务器（slave），接收来自Master上binlog文件的日志内容，解析出SQL，重新更新到Slave，使得主从服务器数据达到一致。
mysql的主从复制架构，需要准备两台机器，并且可以通信，安装好2个mysql，保持版本一致性 
mysql -v 查看数据库版本
 
1.准备主库的配置文件  /etc/my.cnf  MariaDB版本不一样可能有配置文件有差别
写入开启主库的参数
	[mysqld]
	server-id=1 			#标注 主库的身份id
	log-bin=s15mysql-bin		#那个binlog的文件名

2.重启mairadb，读取配置文件
	service mysql restart 

3.查看主库的状态
	mysql -uroot -p 

	show master status;  #这个命令可以查看 日志文件的名字，以及数据起始点 

4.创建用于主从数据同步的账户 % 表示所有的从用户ip均可以 通过  用户名：yuanhao 密码：yuanhaobuxitou
进行主从复制
create user 'yuanhao'@'%' identified by 'yuanhaobuxitou';

5.授予主从同步账号的，复制数据的权限
grant replication slave on *.* to 'yuanhao'@'%';

6.进行数据库的锁表，防止数据写入

flush table with read lock;

7.将数据导出 
mysqldump -u root -p --all-databases >  /opt/zhucong.dump

8.然后将主库的数据，发送给从库
scp /opt/zhucong.dump   root@从库:/opt/

9.此时去从库的mysql上，登录，导入主库的数据，保持数据一致性
mysql -uroot -p 
source /opt/zhucong.dump 



从库的配置
1.写入my.cnf，从库的身份信息
vi /etc/my.cnf 
[mysqld]
server-id=10

2.检查一下主库和从库的 参数信息 

show variables like 'server_id';
show variables like 'log_bin';

3.通过一条命令，开启主从同步
change master to master_host='192.168.13.78',# 主库的ip地址
master_user='yuanhao',
master_password='yuanhaobuxitou',
master_log_file='s15mysql-bin.000001',
master_log_pos=571;

4.开启从库的slave同步
start slave; 

5.查看主从同步的状态
show slave status\G;  

6.查看两条参数 ，确保主从正常
            Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```

#### Redis

https://www.cnblogs.com/pyyu/articles/9365054.html

```
1.在linux安装redis
通过源码编译安装redis
1.下载源码包
	wget http://download.redis.io/releases/redis-4.0.10.tar.gz
2.解压缩redis
	tar -zxf redis-4.0.10.tar.gz 
3.进入redis源码，直接可以编译且安装
	make && make install 
	
4.可以指定配置文件启动redis
	vim /opt/redis-4.0.10/redis.conf 
	1.更改bind参数，让redis可以远程访问
		# JUST COMMENT THE FOLLOWING LINE.
		
		bind 0.0.0.0
		# Protected mode is a layer of security
		这个位置的bind 127.0.0.1 改成下面
		
		bind 0.0.0.0
		
    2.更改redis的默认端口
    	# Accept connections on the specified port, default is 6379 (IANA #815344).
        # If port 0 is specified Redis will not listen on a TCP socket.
        port 6379 改成 6380 
    
        port 6380
        
    3.使用redis的密码进行登录
    	# use a very strong password otherwise it will be very easy to break.
        requirepass 123456
        # Command renaming.

        requirepass 登录redis的密码 自己设置的为123456
        
    4.指定配置文件启动，这里终端应该进入到/opt/redis-4.0.10目录下运行：
        redis-server redis.conf
        
    5.通过新的端口和密码登录redis
        redis-cli -p 6380
        登录后
        输入：auth 密码
        
        redis还支持交互式的参数，登录数据库
        redis-cli -p 6380  -a  redis的密码  （这个不太安全）

    6.通过登录redis，用命令查看redis的密码
        config set  requirepass  新的密码     	#设置新密码
        config get  requirepass  			#获取当前的密码
```

```
过滤出redis.conf文件的空白行和注释行
grep -v "^#"  redis.conf |   grep  -v "^$"
```



##### 1、redis发布订阅

https://www.cnblogs.com/pyyu/p/10013703.html

```
三个角色，提供的redis命令
1.发布者
	publish  频道  消息		给频道发消息
2.订阅者
	SUBSCRIBE  频道     	订阅频道 
	PSUBSCRIBE 频道*  		支持模糊匹配的订阅
3.频道
	channel  频道名 自定义
```

```
PUBLISH channel msg
    将信息 msg 发送到指定的频道 channel
SUBSCRIBE channel [channel ...]
    订阅频道，可以同时订阅多个频道
UNSUBSCRIBE [channel ...]
    取消订阅指定的频道, 如果不指定频道，则会取消订阅所有频道
PSUBSCRIBE pattern [pattern ...]
    订阅一个或多个符合给定模式的频道，每个模式以 * 作为匹配符，比如 it* 匹配所    有以 it 开头的频道( it.news 、 it.blog 、 it.tweets 等等)， news.* 匹配所有    以 news. 开头的频道( news.it 、 news.global.today 等等)，诸如此类
PUNSUBSCRIBE [pattern [pattern ...]]
    退订指定的规则, 如果没有参数则会退订所有规则
PUBSUB subcommand [argument [argument ...]]
    查看订阅与发布系统状态
注意：使用发布订阅模式实现的消息队列，当有客户端订阅channel后只能收到后续发布到该频道的消息，之前发送的不会缓存，必须Provider和Consumer同时在线
```

##### 2、订阅一个或者多个符合模式的频道

```
发布者：
127.0.0.1:6380> PUBLISH python14 byebye


订阅者：
[root@localhost 桌面]# redis-cli -p 6380
127.0.0.1:6380> auth 123456
OK
127.0.0.1:6380> PSUBSCRIBE python*  
# 表示订阅者可以接收所有以python开头的频道，发出的信息（类似模糊查询）

Reading messages... (press Ctrl-C to quit)
1) "psubscribe"
2) "python*"
3) (integer) 1
1) "pmessage"
2) "python*"
3) "python14"
4) "byebye"

注意注意注意注意：这里模糊匹配订阅者的命令是PSUBSCRIBE，不是SUBSCRIBE

```

##### 3、redis持久化RDB与AOF

https://www.cnblogs.com/pyyu/p/10009493.html

```
Redis`是一种内存型数据库，一旦服务器进程退出，数据库的数据就会丢失，为了解决这个问题，`Redis`提供了两种持久化的方案，将内存中的数据保存到磁盘中，避免数据的丢失
```

###### 1、RDB持久化

```
redis`提供了`RDB持久化`的功能，这个功能可以将`redis`在内存中的的状态保存到硬盘中，它可以**手动执行。**

也可以再`redis.conf`中配置，定期执行。

RDB持久化产生的RDB文件是一个经过压缩的二进制文件，这个文件被保存在硬盘中，redis可以通过这个文件还原数据库当时的状态。

RDB(持久化)
内存数据保存到磁盘
在指定的时间间隔内生成数据集的时间点快照（point-in-time snapshot）
优点：速度快，适合做备份，主从复制就是基于RDB持久化功能实现

注意注意注意：：：
在redis退出之前一定执行 save命令，进行保存
rdb通过再redis中使用save命令触发 rdb

rdb配置参数：

dir /data/6379/
dbfilename  dbmp.rdb

每过900秒 有1个操作就进行持久化

save 900秒  1个修改类的操作
save 300秒  10个操作
save 60秒  10000个操作

save  900 1
save 300 10
save 60  10000
```

######  2、redis持久化之RDB实践

1.启动redis服务端,准备配置文件

```
在/opt/redis-4.0.10目录下（可以在任意目录下）新建一个s15.conf文件，其中配置以下内容
daemonize yes
port 6379
logfile /data/6379/redis.log
dir /data/6379              #定义持久化文件存储位置
dbfilename  dbmp.rdb        #rdb持久化文件
bind 10.0.0.10  127.0.0.1    #redis绑定地址
requirepass redhat            #redis登录密码
#save 900 1                    #rdb机制 每900秒 有1个修改记录
#save 300 10                    #每300秒        10个修改记录
#save 60  10000                #每60秒内        10000修改记录
```

2.启动redis服务端

```
redis-server s15.conf
```

3.登录redis设置一个key

```
redis-cli -p 6379 -a redhat
```

4.此时检查目录，/data/6379底下没有dbmp.rdb文件

5.通过save触发持久化，将数据写入RDB文件

```
127.0.0.1:6379> set age 18
OK
127.0.0.1:6379> save
OK
```

###### 3、redis持久化之AOF

```
AOF（append-only log file）
记录服务器执行的所有变更操作命令（例如set del等），并在服务器启动时，通过重新执行这些命令来还原数据集
AOF 文件中的命令全部以redis协议的格式保存，新命令追加到文件末尾。
优点：最大程序保证数据不丢
缺点：日志记录非常大
```

1.准备aof配置文件 s15.conf

```
在/opt/redis-4.0.10目录下新建一个s15.conf文件，其中配置以下内容
port 6379
daemonize yes
logfile /data/6379/redis.log
dir /data/6379


appendonly yes
appendfsync everysec

```

2.启动redis服务

```
redis-server /opt/redis-4.0.10/redis.conf
```

3.检查redis数据目录/data/6379/是否产生了aof文件

```
[root@web02 6379]# ls
appendonly.aof  dbmp.rdb  redis.log
```

4.登录redis-cli，写入数据，实时检查aof文件信息

```
[root@web02 6379]# tail -f appendonly.aof
```

5.设置新key，检查aof信息，然后关闭redis，检查数据是否持久化

```
redis-cli -a redhat shutdown

redis-server /opt/redis-4.0.10/redis.conf

redis-cli -a redhat
```

##### 4、redis不重启之rdb数据切换到aof数据

###### 1、准备rdb的redis服务端

```
redis-server   s15-redis.conf (注明这是在rdb持久化模式下)
```

###### 2.切换rdb到aof

```
redis-cli  登录redis，然后通过命令，激活aof持久化
127.0.0.1:6379>  CONFIG set appendonly yes	#用命令激活aof持久化(临时生效，注意写入到配置文件)
OK
127.0.0.1:6379> 
127.0.0.1:6379> 
127.0.0.1:6379>  CONFIG SET save "" 			#关闭rdb持久化
```

###### 3、将aof操作，写入到配置文件，永久生效，下次重启后生效

	port 6379
	daemonize yes 
	logfile /data/6379/redis.log
	dir /data/6379  
	
	#dbfilename   s15.rdb
	#save 900 1  
	#save 300 10 
	#save 60  10000 
	
	appendonly yes
	appendfsync everysec

###### 4、测试aof数据持久化 ,杀掉redis，重新启动

```
kill 进程PID
redis-server s15-redis.conf 
```

###### 5、写入数据，检查aof文件

##### 5、redis主从同步

https://www.cnblogs.com/pyyu/p/10012904.html

###### 1.准备三个redis配置文件，通过端口的区分，启动三个redis数据库实例，然后配置主从复制

redis-6379.conf 

```
port 6379
daemonize yes
pidfile /data/6379/redis.pid
loglevel notice
logfile "/data/6379/redis.log"
dbfilename dump.rdb
dir /data/6379


检查数据库信息
info
```

redis-6380.conf 

```
#通过命令快速生成配置文件
sed "s/6379/6380/g" redis-6379.conf > redis-6380.conf 

再把 slaveof  127.0.0.1  6379  写入redis-6380.conf   #指明主库的身份ip 和端口
echo "slaveof  127.0.0.1  6379" >> redis-6380.conf
```

redis-6381.conf 

```
#通过命令快速生成配置文件
sed "s/6379/6381/g" redis-6379.conf > redis-6381.conf 

echo "slaveof  127.0.0.1  6379" >> redis-6381.conf
```

###### 2、检查主从状态

检查数据库信息   info ；检查主从信息：info replication

从库：

```
127.0.0.1:6380> info replication
127.0.0.1:6381> info replication
```

主库：

```
127.0.0.1:6379> info replication
```

###### 3、启动三个数据库实例，检测redis主从同步方案

```
redis-server redis-6379.conf
redis-server redis-6380.conf
redis-server redis-6381.conf
```

###### 4、测试写入数据，主库写入数据，检查从库数据

```
主
127.0.0.1:6379> set name chaoge


从
127.0.0.1:6380>get name 
127.0.0.1:6381>get name 
```

###### 5、手动进行主从复制故障切换

```
#关闭主库6379
redis-cli -p 6380 shutdown
```

检查从库主从信息，此时master_link_status:down 

```
redis-cli -p 6380 
127.0.0.1:6380>info replication

redis-cli -p 6381 
127.0.0.1:6381>info replication
```

###### 6、关闭6380的从库身份

```
既然主库挂了，我想要在6380 6381之间选一个新的主库
redis-cli -p 6380
127.0.0.1:6380>info replication
127.0.0.1:6380>slaveof no one
```

###### 7、.将6381设为6380的从库

```
6381连接到6380：
[root@localhost redis-4.0.10]# redis-cli -p 6381
127.0.0.1:6381> SLAVEOF no one
127.0.0.1:6381> SLAVEOF 127.0.0.1 6380
```

###### 8、检查6380，6381的主从信息，测试新的主从数据同步

##### 6、redis 哨兵

https://www.cnblogs.com/pyyu/p/9718679.html

###### 1、什么是哨兵呢？

```
保护redis主从集群，正常运转，当主库挂掉之后，自动的在从库中挑选新的主库，进行同步
```

###### 2、配置哨兵

1、启动三个redis数据库实例（三个配置文件，通过端口区分）

```
[root@localhost liu]# redis-server redis-6379.conf
[root@localhost liu]# redis-server redis-6380.conf
[root@localhost liu]# redis-server redis-6381.conf
```

2、准备三个哨兵，准备三个哨兵的配置文件(仅仅是端口的不同26379,26380,26381) ，

先新建/var/redis/data/文件夹

第一个哨兵文件：redis-sentinel-26379.conf

	port 26379  
	dir /var/redis/data/
	logfile "26379.log"
	daemonize yes  # 表示后台运行
	sentinel monitor s15master 127.0.0.1 6379 2
	sentinel down-after-milliseconds s15master 30000
	sentinel parallel-syncs s15master 1
	sentinel failover-timeout s15master 180000
其他两个哨兵文件，只是端口不一样，通过指令生成：

```
sed "s/26379/26380/g" redis-sentinel-26379.conf >> redis-sentinel-26380.conf
sed "s/26379/26381/g" redis-sentinel-26379.conf >> redis-sentinel-26381.conf
```

3、启动三个哨兵

```
redis-sentinel redis-sentinel-26379.conf 
redis-sentinel redis-sentinel-26380.conf 
redis-sentinel redis-sentinel-26381.conf 
```

4、检查哨兵的通信状态

```
redis-cli -p 26379  info sentinel 
```

5、杀死一个redis主库，6379节点，等待30s以内，检查6380和6381的节点状态

```
kill 6379主节点
redis-cli -p 6380 info replication 
redis-cli -p 6381 info replication 
如果切换的主从身份之后，（原理就是更改redis的配置文件，切换主从身份）
```

	6、恢复6379节点的数据库，查看是否将6379添加为新的slave身份

##### 7、redis-cluster

https://www.cnblogs.com/pyyu/p/9844093.html

redis-cluster安装配置

###### 1.准备6个redis数据库实例，准备6个配置文件redis-{7000....7005}配置文件

​	

```
-rw-r--r-- 1 root root 151 Jan  2 19:26 redis-7000.conf
-rw-r--r-- 1 root root 151 Jan  2 19:27 redis-7001.conf
-rw-r--r-- 1 root root 151 Jan  2 19:27 redis-7002.conf
-rw-r--r-- 1 root root 151 Jan  2 19:27 redis-7003.conf
-rw-r--r-- 1 root root 151 Jan  2 19:27 redis-7004.conf
-rw-r--r-- 1 root root 151 Jan  2 19:27 redis-7005.conf

redis-7000.conf配置文件内容：


port 7000
daemonize yes
dir "/opt/redis/data"
logfile "7000.log"
dbfilename "dump-7000.rdb"
cluster-enabled yes   #开启集群模式
cluster-config-file nodes-7000.conf　　#集群内部的配置文件
cluster-require-full-coverage no　　#redis cluster需要16384个slot都正常的时候才能对外提供服务，##换句话说，只要任何一个slot异常那么整个cluster不对外提供服务。 因此生产环境一般为no

其他只是端口不同：利用命令修改： sed "s/7000/7001/g" redis-7000.conf >> redis-7001.conf   依次类推。
```

###### 2、启动6个redis数据库实例

```
[root@localhost s15rediscluster]# redis-server redis-7000.conf 
[root@localhost s15rediscluster]# redis-server redis-7001.conf 
[root@localhost s15rediscluster]# redis-server redis-7002.conf 
[root@localhost s15rediscluster]# redis-server redis-7003.conf 
[root@localhost s15rediscluster]# redis-server redis-7004.conf 
[root@localhost s15rediscluster]# redis-server redis-7005.conf 
```

###### 3.配置ruby语言环境，脚本一键启动redis-cluster 

	1.下载ruby语言的源码包，编译安装
		wget https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.1.tar.gz
		
	2.解压缩
		./configure --prefix=/opt/ruby/	   释放makefile
		make && make install     编译且安装
	3.下载安装ruby操作redis的模块包
		wget http://rubygems.org/downloads/redis-3.3.0.gem
	4.配置ruby的环境变量
	echo $PATH
	vim /etc/profile
	写入最底行(根据自己存放的位置自行改变)
	PATH=$PATH:/opt/ruby/bin/
	读取文件
	source /etc/profile 
	
	5.通过ruby的包管理工具去安装redis包，安装后会生成一个redis-trib.rb这个命令
	一键创建redis-cluster 其实就是分配主从关系 以及 槽位分配 slot槽位分配
	redis-trib.rb create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
	
	6.检查节点主从状态
	redis-cli -p 7000  info replication 
	
	7.向redis集群写入数据，查看数据流向
	redis-cli -p 7000 -c  #这里会将key自动的重定向，放到某一个节点的slot槽位中,-c表示redis-cluster集群
	set  name  s15 
	set  addr shahe  

