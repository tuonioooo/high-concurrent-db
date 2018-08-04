# Mysql Cluster/集群

MySQL Cluster 是MySQL适合于分布式计算环境的高实用、高冗余版本。它采用了NDB Cluster 存储引擎，允许在1个 Cluster 中运行多个MySQL服务器。在MyQL 5.0及以上的二进制版本中、以及与最新的Linux版本兼容的RPM中提供了该存储引擎。（注意，要想获得MySQL Cluster 的功能，必须安装 mysql-server 和 mysql-max RPM）。  
目前能够运行MySQL Cluster 的操作系统有Linux、Mac OS X和Solaris（一些用户通报成功地在FreeBSD上运行了MySQL Cluster ，但MySQL AB公司尚未正式支持该特性）。  
一、MySQL Cluster概述  
MySQL Cluster 是一种技术，该技术允许在无共享的系统中部署“内存中”数据库的 Cluster 。通过无共享体系结构，系统能够使用廉价的硬件，而且对软硬件无特殊要求。此外，由于每个组件有自己的内存和磁盘，不存在单点故障。  
MySQL Cluster 由一组计算机构成，每台计算机上均运行着多种进程，包括MySQL服务器，NDB Cluster 的数据节点，管理服务器，以及（可能）专门的数据访问程序。关于 Cluster 中这些组件的关系，请参见下图：  
![](http://imysql.cn/files/pictures/cluster-components-1.png)  
所有的这些节点构成一个完成的MySQL集群体系。数据保存在“NDB存储服务器”的存储引擎中，表（结构）则保存在“MySQL服务器”中。应用程序通过“MySQL服务器”访问这些数据表，集群管理服务器通过管理工具\(ndb\_mgmd\)来管理“NDB存储服务器”。  
通过将MySQL Cluster 引入开放源码世界，MySQL为所有需要它的人员提供了具有高可用性、高性能和可缩放性的 Cluster 数据管理。  
二、MySQL Cluster 基本概念  
“NDB” 是一种“内存中”的存储引擎，它具有可用性高和数据一致性好的特点。  
MySQL Cluster 能够使用多种故障切换和负载平衡选项配置NDB存储引擎，但在 Cluster 级别上的存储引擎上做这个最简单。MySQL Cluster的NDB存储引擎包含完整的数据集，仅取决于 Cluster本身内的其他数据。  
目前，MySQL Cluster的 Cluster部分可独立于MySQL服务器进行配置。在MySQL Cluster中， Cluster的每个部分被视为1个节点。

* 管理\(MGM\)节点：这类节点的作用是管理MySQL Cluster内的其他节点，如提供配置数据、启动并停止节点、运行备份等。由于这类节点负责管理其他节点的配置，应在启动其他节点之前首先启动这类节点。MGM节点是用命令“ndb\_mgmd”启动的。
* 数据节点：这类节点用于保存 Cluster的数据。数据节点的数目与副本的数目相关，是片段的倍数。例如，对于两个副本，每个副本有两个片段，那么就有4个数据节点。不过没有必要设置多个副本。数据节点是用命令“ndbd”启动的。
* SQL节点：这是用来访问 Cluster数据的节点。对于MySQL Cluster，客户端节点是使用NDB Cluster存储引擎的传统MySQL服务器。通常，SQL节点是使用命令“mysqld –ndbcluster”启动的，或将“ndbcluster”添加到“my.cnf”后使用“mysqld”启动。
* 注释：在很多情况下，术语“节点”用于指计算机，但在讨论MySQL Cluster时，它表示的是进程。在单台计算机上可以有任意数目的节点，为此，我们采用术语“ Cluster主机”。

管理服务器\(MGM节点\)负责管理 Cluster配置文件和 Cluster日志。 Cluster中的每个节点从管理服务器检索配置数据，并请求确定管理服务器所在位置的方式。当数据节点内出现新的事件时，节点将关于这类事件的信息传输到管理服务器，然后，将这类信息写入 Cluster日志。  
此外，可以有任意数目的 Cluster客户端进程或应用程序。它们分为两种类型：

* 标准MySQL客户端：对于MySQL Cluster，它们与标准的（非 Cluster类）MySQL没有区别。换句话讲，能够从用PHP、Perl、C、C++、Java、Python、Ruby等编写的现有MySQL应用程序访问MySQL Cluster。
* 管理客户端：这类客户端与管理服务器相连，并提供了启动和停止节点、启动和停止消息跟踪（仅调试版本）、显示节点版本和状态、启动和停止备份等的命令。

三、开始准备  
1、准备服务器  
现在，我们计划建立有5个节点的MySQL CLuster体系，因此需要用到5台机器，分别做如下用途：

```
		节点(用途)		IP地址(主机名)
管理节点(MGM)		192.168.0.1(db1)
SQL节点1(SQL1)		192.168.0.2(db2)
SQL节点2(SQL2)		192.168.0.3(db3)
数据节点1(NDBD1)	192.168.0.4(db4)
数据节点2(NDBD2)	192.168.0.5(db5)

```

2、注意事项及其他  
每个节点的操作系统都是Linux，下面的描述中将使用主机名，不再使用IP地址来表示。由于MySQL Cluster采用TCP/IP方式连接，并且节点之间的数据传输没有加密，因此这个体系最好只在单独的子网中运行，并且考虑到传输的速率，强烈建议不要跨越公网使用这个体系。所需的MySQL软件请事先在 http://dev.mysql.com/downloads 下载。  
实际上整个体系可以在一个单独的实体计算机上运行成功，当然了，必须设定不同的目录以及端口等，只能作为测试时使用。  
四、开始安装  
1、假定条件  
在每个节点计算机上都采用 nobody 用户来运行Cluster，因此执行如下命令添加相关用户（如果已经存在则略过，且用root用户执行）：

```
	root# /usr/sbin/groupadd nobody
root# /usr/sbin/useradd nobody -g nobody

```

假设已经下载了mysql可直接使用的二进制安装包，且放在 /tmp 下了。  
2、SQL节点和存储节点\(NDB节点\)安装\(即4个机器重复执行以下步骤\)

```
	root# cd /tmp/
root# tar zxf mysql-max-5.0.24-linux-i686.tar.gz
root# mv mysql-max-5.0.24-linux-i686 /usr/local/mysql/
root# cd /usr/local/mysql/
root# ./configure --prefix=/usr/local/mysql
root# ./scripts/mysql_install_db
root# chown -R nobody:nobody /usr/local/mysql/

```

3、配置SQL节点

```
	root# vi /usr/local/mysql/my.cnf

```

然后输入如下内容：

```
[mysqld]
basedir         = /usr/local/mysql/
datadir         = /usr/local/mysql/data
user            = nobody
port            = 3306
socket          = /tmp/mysql.sock
ndbcluster
ndb-connectstring=db1
[MYSQL_CLUSTER]
ndb-connectstring=db1

```

4、配置存储节点\(NDB节点\)

```
	root# vi /usr/local/mysql/my.cnf

```

然后输入如下内容：

```
[mysqld]
ndbcluster
ndb-connectstring=db1
[MYSQL_CLUSTER]
ndb-connectstring=db1

```

5、安装管理节点

```
	root# cd /tmp/
root# tar zxf mysql-max-5.0.24-linux-i686.tar.gz
root# mkdir /usr/local/mysql/
root# mkdir /usr/local/mysql/data/
root# cd mysql-max-5.0.24-linux-i686/bin/
root# cp ndb_mgm* /usr/local/mysql/
root# chown -R nobody:nobody /usr/local/mysql

```

6、配置管理节点

```
		root# vi /usr/local/mysql/config.ini

```

然后输入如下内容：

```
[NDBD DEFAULT]
NoOfReplicas=1
[TCP DEFAULT]
portnumber=3306
#设置管理节点服务器
[NDB_MGMD]
hostname=db1
#MGM上保存日志的目录
datadir=/usr/local/mysql/data/
#设置存储节点服务器(NDB节点)
[NDBD]
hostname=db4
datadir=/usr/local/mysql/data/
#第二个NDB节点
[NDBD]
hostname=db5
datadir=/usr/local/mysql/data/
#设置SQL节点服务器
[MYSQLD]
hostname=db2
#第二个SQL节点
[MYSQLD]
hostname=db3

```

注释： Cluster管理节点的默认端口是1186，数据节点的默认端口2202。从MySQL 5.0.3开始，该限制已被放宽， Cluster能够根据空闲的端口自动地为数据节点分配端口。如果你的版本低于5.0.22，请注意这个细节。  
五、启动MySQL Cluster  
较为合理的启动顺序是，首先启动管理节点服务器，然后启动存储节点服务器，最后才启动SQL节点服务器：

* 在管理节点服务器上，执行以下命令启动MGM节点进程：
 
  ```
  		root# /usr/local/mysql/ndb_mgmd -f /usr/local/mysql/config.ini
	
  ```

  必须用参数“-f”或“--config-file”告诉 ndb\_mgm 配置文件所在位置，默认是在ndb\_mgmd相同目录下。

* 在每台存储节点服务器上，如果是第一次启动ndbd进程的话，必须先执行以下命令：
 
  ```
  		root# /usr/local/mysql/bin/ndbd --initial
	
  ```

  注意，仅应在首次启动ndbd时，或在备份/恢复数据或配置文件发生变化后重启ndbd时使用“--initial”参数。因为该参数会使节点删除由早期ndbd实例创建的、用于恢复的任何文件，包括用于恢复的日志文件。  
  如果不是第一次启动，直接运行如下命令即可：

  ```
  		root# /usr/local/mysql/bin/ndbd
	
  ```

* 最后，运行以下命令启动SQL节点服务器：
 
  ```
  		root# /usr/local/mysql/bin/mysqld_safe --defaults-file=/usr/local/mysql/my.cnf 
  &
  ```

  如果一切顺利，也就是启动过程中没有任何错误信息出现，那么就在管理节点服务器上运行如下命令：

  ```
  		root# /usr/local/mysql/ndb_mgm
  	-- NDB Cluster -- Management Client --
  	ndb_mgm
  >
   SHOW
  	Connected to Management Server at: localhost:1186
  	Cluster Configuration
  	---------------------
  	[ndbd(NDB)]     2 node(s)
  	id=2    @192.168.0.4  (Version: 5.0.22, Nodegroup: 0, Master)
  	id=3    @192.168.0.5  (Version: 5.0.22, Nodegroup: 0)
  	[ndb_mgmd(MGM)] 1 node(s)
  	id=1    @192.168.0.1  (Version: 5.0.22)
  	[mysqld(SQL)]   1 node(s)
  	id=2   (Version: 5.0.22)
  	id=3   (Version: 5.0.22)
	
  ```

具体的输出内容可能会略有不同，这取决于你所使用的MySQL版本。  
注意：如果你正在使用较早的MySQL版本，你或许会看到引用为‘\[mysqld\(API\)\]’的SQL节点。这是一种早期的用法，现已放弃。  
现在，应能在MySQL Cluster中处理数据库，表和数据。  
六、创建数据库表  
与没有使用 Cluster的MySQL相比，在MySQL Cluster内操作数据的方式没有太大的区别。执行这类操作时应记住两点：

* 表必须用ENGINE=NDB或ENGINE=NDBCLUSTER选项创建，或用ALTER TABLE选项更改，以使用NDB Cluster存储引擎在 Cluster内复制它们。如果使用mysqldump的输出从已有数据库导入表，可在文本编辑器中打开SQL脚本，并将该选项添加到任何表创建语句，或用这类选项之一替换任何已有的ENGINE（或TYPE）选项。
* 另外还请记住，每个NDB表必须有一个主键。如果在创建表时用户未定义主键，NDB Cluster存储引擎将自动生成隐含的主键。（注释：该隐含 键也将占用空间，就像任何其他的表索引一样。由于没有足够的内存来容纳这些自动创建的键，出现问题并不罕见）。

下面是一个例子：  
在db2上，创建数据表，插入数据：

```
[db2~]root# mysql -uroot test
[db2~]mysql
>
 create table city(
[db2~]mysql
>
 id mediumint unsigned not null auto_increment primary key,
[db2~]mysql
>
 name varchar(20) not null default ''
[db2~]mysql
>
 ) engine = ndbcluster default charset utf8;
[db2~]mysql
>
 insert into city values(1, 'city1');
[db2~]mysql
>
 insert into city values(2, 'city2');

```

在db3上，查询数据：

```
[db3~]root# mysql -uroot test
[db2~]mysql
>
 select * from city;
+-----------+
|id | name  |
+-----------+
|1  | city1 |
+-----------+
|2  | city2 |
+-----------+

```

七、安全关闭  
要想关闭 Cluster，可在MGM节点所在的机器上，在Shell中简单地输入下述命令：

```
	[db1~]root# /usr/local/mysql/ndb_mgm -e shutdown

```

运行以下命令关闭SQL节点的mysqld服务：

```
	[db2~]root# /usr/local/mysql/bin/mysqladmin -uroot shutdown

```

八、其他  
关于MySQL Cluster更多详细的资料以及备份等请参见MySQL手册的“MySQL Cluster\(MySQL 集群\)”章节。  
参考资料：《MySQL 5.1 中文手册》  
  


技术相关: 

[MySQL基础知识](http://imysql.cn/taxonomy/term/5)

[Cluster/集群](http://imysql.cn/taxonomy/term/22)

