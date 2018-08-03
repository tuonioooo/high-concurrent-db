# 示例一

## 主从复制

**1、复制的介绍**

MySQL 支持单向、异步复制，复制过程中一个服务器充当主服务器，而一个或多个其它服务器充当从服务器。主服务器将更新写入二进制日志文件，并维护文件的一个索引 以跟踪日志循环。这些日志可以记录发送到从服务器的更新。当一个从服务器连接主服务器时，它通知主服务器从服务器在日志中读取的最后一次成功更新的位置。 从服务器接收从那时起发生的任何更新，然后封锁并等待主服务器通知新的更新。

**请注意当你进行复制时，所有对复制中的表的更新必须在主服务器上进行。否则，你必须要小心，以避免用户对主服务器上的表进行的更新与对从服务器上的表所进行的更新之间的冲突**。

**单向复制**有利于健壮性、速度和系统管理：

* 主服务器/从服务器设置增加了健壮性。主服务器出现问题时，你可以切换到从服务器作为份。
* 通 过在主服务器和从服务器之间切分处理客户查询的负荷，可以得到更好的客户响应时间。SELECT查询可以发送到从服务器以降低主服务器的查询处理负荷。但 修改数据的语句仍然应发送到主服务器，以便主服务器和从服务器保持同步。如果非更新查询为主，该负载均衡策略很有效，但一般是更新查询。
* 使用复制的另一个好处是可以使用一个从服务器执行备份，而不会干扰主服务器。在备份过程中主服务器可以继续处理更新。

MySQL 提供了数据库的同步功能，这对我们实现数据库的冗灾、备份、恢复、负载均衡等都是有极大帮助的

**双向复制**是在单向复制的基础上由建立了一次由slave向master的单向复制，需要自己编写相应的更新策略，否则很容易出现数据不一致的问题。

2、**环境**

用的是radhat 5.1操作系统        mysql5.1.50版本

master    计算机名：ltest      IP地址：172.31.70.51

slave     计算机名：erpdemo      IP地址：172.31.70.95

注意：该源码安装包仅可以安装在Redhat5以上，在Redhat4上编译会出现问题。

**3、mysql的单向复制**

注意 mysql 数据库的版本，两个数据库版本要相同，或者slave比master版本高！这里采用的版本一致。

* **安装Mysql**

1）在ltest、erpdemo上安装mysql软件，通过源码安装

mysql软件可以在[http://www.mysql.com](http://www.mysql.com/)上下载，下来通过ftp或则其他的软件上传到服务器上：

2）在linux系统中添加运行Mysql的用户和组

```
[root@ltest ~]# groupadd mysql
[root@ltest ~]# useradd -g mysql mysql
```

3）解压缩源码包

```
[root@ltest ~]# tar -zxvf mysql-5.1.50.tar.gz
[root@ltest ~]# cd mysql-5.1.50
```

4）配置编译

配置mysql的安装目录

```
[root@ltest mysql-5.1.50]# ./configure --prefix=/usr/local/mysql）编译并安装
[root@ltest mysql-5.1.50]#make     #编译
[root@ltest mysql-5.1.50]#make install
```

5）装载原始授权到数据库

```
[root@ltest mysql-5.1.50]#./scripts/mysql_install_db
```

6）copy mysql的配置文件到/etc目录

```
[root@ltest mysql-5.1.50]# cp support-files/my-medium.cnf /etc/my.cnf
```

7）copy mysql的启动脚本到资源目录

```
[root@ltest mysql-5.1.50]#cp ./support-files/mysql.server /etc/rc.d/init.d/mysqld
```

8）添加mysql服务，让系统启动时自动启动mysql服务

```
[root@ltest mysql-5.1.50]#chmod +x /etc/rc.d/init.d/mysqld
[root@ltest mysql-5.1.50]#chkconfig --level 235 mysqld on
```

9）更改目录属主

```
[root@ltest mysql-5.1.50]#chown -R mysql.mysql /var/lib/mysql
```

10）设置环境变量

在 /etc/profile添加一行就 ，在运行mysql的时候就不用输入很长的路经了

```
export PATH=$PATH:/usr/local/mysql/bin
```

11）启动mysql服务

```
[root@ltest mysql-5.1.50]#service mysqld start
```

到这里mysql软件就按转完毕，erpdemo上的mysql安装按照上面的就行了。

* **配置mysql的配置文件**

1）进入mysql命令行，为slave用户添加同步专用权限

输入密码 ，就进入到mysql命令行中了，一般刚装好的没有密码。

\#给与从服务器用户replication的同步权限

```
[root@ltest ~]# mysql
Welcome to the MySQL monitor. Commands end with ; or /g.
Your MySQL connection id is 3 to server version: 5.1.50
Type 'help;' or '/h' for help. Type '/c' to clear the buffer.
mysql>

mysql> GRANT REPLICATION SLAVE,REPLICATION CLIENT,RELOAD,SUPER ON *.* TO 'replication'@'172.31.70.95' IENTIFIED BY '123456';
```

\#如果需要的话添加管理用户，通过mysql的客户端来测试同步的情况

```
mysql> Grant ALL PRIVILEGES ON *.* TO li@'%' IDENTIFIED BY '123456';
```

\#刷新权限，使设置生效

```
mysql>Flush privileges;
```

2）配置ltest 的/etc/my.cnf配置文件

创建更新日志的目录并给mysql用户的权限

```
[root@ltest ~]# mkdir /var/log/mysql
[root@ltest ~]# chown -R mysql.mysql /var/log/mysql
```

修改配置文件中如下内容，如果没有添加上去：

| log-bin=mysql-bin | 启动二进制日志系统 |
| :--- | :--- |
| binlog-do-db=test | 二进制需要同步的数据库名 |
| server-id = 1 | 本机数据库ID 标示为主，该部分还应有一个server-id=Master\_id选项，其中master\_id必须为1到232–1之间的一个正整数值 |
| log-bin=/var/log/mysql/updatelog | \#设定生成log文件名，这里的路径没有mysql目录要手动创建并给于它mysql用户的权限。 |
| binlog-ignore-db=mysql | \# 避免同步mysql用户配置，以免不必要的麻烦 |

3）停止数据库，并将本地需要同步数据库打包拷贝到从数据库上

```
[root@ltest ~]#service mysqld stop                        #停止mysql的服务
[root@ltest ~]#tar -cvf /root/db.tar /usr/local/mysql/test                #备份主服务器需要同步的数据库
[root@ltest ~]#scp /root/db.tar root@172.31.70.95:/usr/local/mysql              #通过远程拷贝到从服务器上，通过这个拷贝的时候需要输入erpdemo的root密码。
[root@ltest ~]#Service mysqld start                                            #启动主服务器mysql服务
```

二、同步slave 从服务器配置

1）配置slave服务器/etc/my.cnf文件

将以下配置启用：

| server-id = 2 | 从服务器ID号，不要和主ID相同 ，如果设置多个从服务器，每个从服务器必须有一个唯一的server-id值，必须与主服务器的以及其它从服务器的不相同。可以认为server-id值类似于IP地址：这些ID值能唯一识别复制服务器群集中的每个服务器实例。 |
| :--- | :--- |
| master-host = 172.31.70.51 | 指定主服务器IP地址 |
| master-user = replication | 指定在主服务器上可以进行同步的用户名 |
| master-password = 123456 | 密码 |
| master-port = 3306 | 同步所用的端口 |
| master-connect-retry=60 | 断点重新连接时间 |
| replicate-ignore-db=mysql | 屏蔽对mysql库的同步，以免有麻烦 |
| replicate-do-db=test | 同步数据库名称 |

2）装载主服务器数据库:

```
[root@erpdemo ~]#cd /var/lib/mysql             #进入数据库库文件主目录
[root@erpdemo ~]#tar -xvf db.tar               #解压缩
[root@erpdemo ~]#service mysqld start          #启动从数据库服务
```

三、查询配置

在ltest上进入mysql的命令行

用下面的命令查看

```
[root@ltest ~]# mysql               #进入mysql命令行

mysql> show master status;          #显示(不同主机结果不同)

+------------------+----------+-------------------+------------------+

| File | Position | Binlog_Do_DB | Binlog_Ignore_DB |

+------------------+----------+-------------------+------------------+

| updatelog.000028 | 313361 |db1 | mysql |

+------------------+----------+-------------------+------------------+

（同步之前如果怀疑主从数据不同步可以采取：上面冷备份远程拷贝法或者在从服务器上命行同步方法）
```

在从服务器执行MySQL命令下：

> mysql&gt; slave stop；             \#先停止slave服务
>
> mysql&gt; CHANGE MASTER TO MASTER\_LOG\_FILE='updatelog.000028',MASTER\_LOG\_POS=313361;
>
> \#根据上面主服务器的show master status的结果，进行从服务器的二进制数据库记录回归，达到同步的效果
>
> mysql&gt;slave start;                      \#启动从服务器同步服务  
> mysql&gt; show slave status/G;
>
> 用show slave status/G;看一下从服务器的同步情况  
> Slave\_IO\_Running: Yes  
> Slave\_SQL\_Running: Yes

如果都是yes，那代表已经在同步

利用mysql的客户端来测试，要比在命令行方便的多。

更多详细信息以及参数设置,请参考MySQL 5.0 Manual手册.

**4、进行mysql 双向同步配置**

a、 先修改原slave 服务器配置

1）配置原slave服务器/etc/my.cnf文件（加粗斜体为添加内容）

| server-id = 2 | 从服务器ID号，不要和主ID相同 |
| :--- | :--- |
| master-host = 172.31.70.51 | 指定主服务器IP地址 |
| master-user =  replication | 制定在主服务器上可以进行同步的用户名 |
| master-password = 123 | 密码 |
| master-port = 3306 | 同步所用的端口 |
| master-connect-retry=60 | 断点重新连接时间 |
| replicate-ignore-db=mysql | 屏蔽对mysql库的同步 |
| replicate-do-db=db1 | 同步数据库名称 |
| _**log-bin=/var/log/mysql/updatelog**_ | _**设定生成log文件名**_ |
| _**binlog-do-db=db1**_ | _**设置同步数据库名**_ |
| _**binlog-ignore-db=mysql**_ | _**避免同步mysql用户配置，以免不必要的麻烦**_ |

2）重新启动mysql服务，创建一个同步专用账号

输入密码 ，就进入到mysql命令行中了，一般刚装好的没有密码。

> \[root@ltest ~\]\# mysql  
> Welcome to the MySQL monitor. Commands end with ; or /g.  
> Your MySQL connection id is 3 to server version: 5.1.50  
> Type 'help;' or '/h' for help. Type '/c' to clear the buffer.  
> mysql&gt;  
>   
> mysql&gt; GRANT REPLICATION SLAVE,REPLICATION CLIENT,RELOAD,SUPER ON \*.\* TO ['replication'@'192.168.1.2'](http://www.cnblogs.com/mailto:) IDENTIFIED BY '123456';\#给与从服务器用户replication的同步权限  
> mysql&gt; Grant ALL PRIVILEGES ON \*.\* TO [li@'%'](mailto:li@) IDENTIFIED BY '123456';\#如果需要的话添加管理用户，通过mysql的客户端来测试同步的情况  
> mysql&gt;Flush privileges;  
> \#刷新权限，使设置生效

b、 修改原master主服务器的my.cnf,添加如下内容（加粗斜体为添加部分）

| log-bin=mysql-bin | 启动二进制日志系统 |
| :--- | :--- |
| binlog-do-db=db1 | 二进制需要同步的数据库名 |
| server-id = 1 | 本机数据库ID 标示为主 |
| log-bin=/var/log/mysql/updatelog | \#设定生成log文件名 |
| binlog-ignore-db=mysql | \# 避免同步mysql用户配置，以免不必要的麻烦 |
| _**master-host = 172.31.70.95**_ | _**设置从原slave数据库同步更新**_ |
| _**master-user = repl**_ | _**更新用户**_ |
| _**master-password = 123**_ | _**密码**_ |
| _**master-port = 3306**_ | _**端口**_ |
| _**replicate-do-db=test**_ | _**需要更新的库**_ |
|  |  |

启动mysql服务

\[root@ltest ~\]\#service mysqld restart

在erpdemo服务器执行MySQL命令符下：

> mysql&gt; show master status;
>
> 看看有无作为主服务器的信息
>
> +------------------+----------+-------------------+------------------+
>
> \| File \| Position \| Binlog\_Do\_DB \| Binlog\_Ignore\_DB \|
>
> +------------------+----------+-------------------+------------------+
>
> \| updatelog.000028 \| 313361 \|test \| mysql \|
>
> +------------------+----------+-------------------+------------------+
>
> 在ltest服务器执行MySQL命令下：
>
> \[root@ltest ~\]\#mysql                           \#进入mysql命令行
>
> mysql&gt; slave stop;            \#先停止slave服务
>
> mysql&gt; CHANGE MASTER TO MASTER\_HOST='192.168.1.2',MASTER\_USER='replication',MASTER\_PASSWORD='123456',MASTER\_PORT=3306MASTER\_LOG\_FILE='updatelog.000028',MASTER\_LOG\_POS=313361;  
>   
> \#根据上面主服务器的show master status的结果，进行从服务器的二进制数据库记录回归，达到同步的效果
>
> mysql&gt; slave start;   \#启动从服务器同步服务

c、 测试

1）在ltest服务器上进入mysql命令行

> \[root@ltest ~\]\#mysql
>
> mysql&gt;SHOW SLAVE STATUS/
>
> Slave\_IO\_Running: Yes
>
> Slave\_SQL\_Running: Yes

此处Slave\_IO\_Running ,Slave\_SQL\_Running 都应该是yes,表示从库的I/O,Slave\_SQL线程都正确开启.

表明数据库正在同步。

2）在erpdemo服务器上进入mysql命令行，用 show slave status；查看

> \[root@ltest ~\]\#mysql
>
> mysql&gt;SHOW SLAVE STATUS/
>
> Slave\_IO\_Running: Yes
>
> Slave\_SQL\_Running: Yes
>
> 此处Slave\_IO\_Running ,Slave\_SQL\_Running 都应该是yes,表示从库的I/O,Slave\_SQL线程都正确开启.表明数据库正在同步。

3）这里我找到了一个mysql的客户端。利用在mysql上建立的管理用户登陆数据库，可以直接在表中写入值，去另一个数据库上看能不能刷新出来，在那里数据库上写入的数据。

