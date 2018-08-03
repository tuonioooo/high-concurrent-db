# 示例二

大型网站为了软解大量的并发访问，除了在网站实现分布式负载均衡，远远不够。到了数据业务层、数据访问层，如果还是传统的数据结构，或者只是单单靠一台服务器扛，如此多的数据库连接操作，数据库必然会崩溃，数据丢失的话，后果更是 不堪设想。这时候，我们会考虑如何减少数据库的联接，一方面采用优秀的代码框架，进行代码的优化，采用优秀的数据缓存技术如：memcached,如果资金丰厚的话，必然会想到假设服务器群，来分担主数据库的压力。Ok切入今天微博主题，利用MySQL主从配置，实现读写分离，减轻数据库压力。这种方式，在如今很多网站里都有使用，也不是什么新鲜事情，今天总结一下，方便大家学习参考一下。

概述：搭设一台Master服务器（win8.1系统，Ip：192.168.0.104），搭设两台Slave服务器（虚拟机——一台Ubuntu，一台 Windows Server 2003）

原理：主服务器（Master）负责网站NonQuery操作，从服务器负责Query操作，用户可以根据网站功能模特性块固定访问Slave服务器，或者自己写个池或队列，自由为请求分配从服务器连接。主从服务器利用MySQL的二进制日志文件，实现数据同步。二进制日志由主服务器产生，从服务器响应获取同步数据库。

具体实现：

1、在主从服务器上都装上MySQL数据库，windows系统鄙人安装的是mysql\_5.5.25.msi版本，Ubuntu安装的是mysql-5.6.22-linux-glibc2.5-i686.tar

windows安装mysql就不谈了，一般地球人都应该会。鄙人稍微说一下Ubuntu的MySQL安装，我建议不要在线下载安装，还是离线安装的好。大家可以参考[ http://www.linuxidc.com/Linux/2013-01/78716.htm](http://www.linuxidc.com/Linux/2013-01/78716.htm) 这位不知道大哥还是姐妹，写的挺好按照这个就能装上。在安装的时候可能会出现几种现象，大家可以参考解决一下：

（1）如果您不是使用root用户登录，建议 su - root 切换到Root用户安装，那就不用老是 sudo 了。

（2）存放解压的mysql 文件夹，文件夹名字最好改成mysql

（3）在./support-files/mysql.server start 启动MySQL的时候，可能会出现一个警告，中文意思是启动服务运行读文件时，忽略了my.cnf文件，那是因为my.cnf的文件权限有问题，mysql会认为该文件有危险不会执行。但是mysql还会启动成功，但如果下面配置从服务器参数修改my.cnf文件的时候，你会发现文件改过了，但是重启服务时，修改过后的配置没有执行，而且您 list一下mysql的文件夹下会发现很多.my.cnf.swp等中间文件。这都是因为MySQL启动时没有读取my.cnf的原因。这时只要将my.cnf的文件权限改成my\_new.cnf的权限一样就Ok，命令：chmod 644 my.cnf就Ok

![](https://images0.cnblogs.com/blog/157872/201412/141051539312957.png)

（4）Ubuntu中修改文档内容没有Vim，最好把Vim 装上，apt-get install vim,不然估计会抓狂。

这时候我相信MySQL应该安装上去了。

2、配置Master主服务器

（1）在Master MySQL上创建一个用户‘repl’，并允许其他Slave服务器可以通过远程访问Master，通过该用户读取二进制日志，实现数据同步。

```
mysql>create user repl; //创建新用户
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'192.168.0.%' IDENTIFIED BY 'mysql';#repl用户必须具有REPLICATION SLAVE权限，除此之外没有必要添加不必要的权限，密码为mysql。说明一下192.168.0.%，这个配置是指明repl用户所在服务器，这里%是通配符，表示192.168.0.0-192.168.0.255的Server都可以以repl用户登陆主服务器。当然你也可以指定固定Ip。
```

（2）找到MySQL安装文件夹修改my.Ini文件。mysql中有好几种日志方式，这不是今天的重点。我们只要启动二进制日志log-bin就ok。

 在\[mysqld\]下面增加下面几行代码

```
server-id=1   //给数据库服务的唯一标识，一般为大家设置服务器Ip的末尾号
log-bin=master-bin
log-bin-index=master-bin.index
```

（3）查看日志

mysql&gt; SHOW MASTER STATUS;  
+-------------------+----------+--------------+------------------+  
\| File \| Position \| Binlog\_Do\_DB \| Binlog\_Ignore\_DB \|  
+-------------------+----------+--------------+------------------+  
\|master-bin.000001\| 1285 \| \| \|  
+-------------------+----------+--------------+------------------+  
1 row in set \(0.00 sec\)

重启MySQL服务

3、配置Slave从服务器（windows）

（1）找到MySQL安装文件夹修改my.ini文件，在\[mysqld\]下面增加下面几行代码

```
[mysqld]
server-id=2
relay-log-index=slave-relay-bin.index
relay-log=slave-relay-bin 
```

重启MySQL服务

（2）连接Master

change master to master\_host='192.168.0.104', //Master 服务器Ip  
master\_port=3306,  
master\_user='repl',  
master\_password='mysql',  
master\_log\_file='master-bin.000001',//Master服务器产生的日志  
master\_log\_pos=0;

（3）启动Slave

start slave;

4、Slave从服务器（Ubuntu）

（1）找到MySQL安装文件夹修改my.cnf文件,vim my.cnf

![](https://images0.cnblogs.com/blog/157872/201412/141118216966295.png) s



（2） ./support-files/myql.server restart 重启MySQL服务  ,  ./bin/mysql 进入MySQL命令窗口 

（3）连接Master

change master to master\_host='192.168.0.104', //Master 服务器Ip  
master\_port=3306,  
master\_user='repl',  
master\_password='mysql',   
master\_log\_file='master-bin.000001',//Master服务器产生的日志  
master\_log\_pos=0;

（4）启动Slave

start slave;

OK所有配置都完成了，这时候大家可以在Master Mysql 中进行测试了，因为我们监视的时Master mysql  所有操作日志，所以，你的任何改变主服务器数据库的操作，都会同步到从服务器上。创建个数据库，表试试吧。。。



