# 示例三——Mysql的实时同步-双机互备\(双master\)

设置方法： 

步一 设 

A 服务服 \(192.168.1.43\) 上 用户为 backup, 123456 , 同步的[数据库](https://www.2cto.com/database/)为test; 

B 服务服 \(192.168.1.23\) 上 用户为 root, 123456, 同步的数据库为test; 

步二 配置[mysql](https://www.2cto.com/database/MySQL/).ini: 

A服务器 

> \#Replication master 
>
> server-id = 10 
>
> log-bin="E:/MySQL/logs/mysql\_binary\_log" 
>
> binlog-do-db=test 
>
> binlog-Ignore-db=information\_schema 
>
> \# 单向备份时 A只需上面部分 B只需下面部分 下面部分同样需加server-id 
>
> \# Replication slave 
>
> master-host="192.168.1.23" 
>
> master-user=root 
>
> master-password="123456" 
>
> master-port=3306 
>
> master-connect-retry=60 
>
> replicate-do-db=test 
>
> replicate-Ignore-db=information\_schema

B服务器 

> \#Replication master 
>
> server-id = 2 
>
> log-bin="c:/mysql5/logs/mysql\_binary\_log" 
>
> binlog-do-db=test 
>
> binlog-Ignore-db=information\_schema 
>
>  
>
> \# Replication slave 
>
> master-host="192.168.1.43" 
>
> master-user=backup 
>
> master-password=123456 
>
> master-port=3306 
>
> master-connect-retry=60 
>
> replicate-do-db=test 
>
> replicate-Ignore-db=information\_schema

============================================================= 

解释: 

3\)binlog-do-db=test 表示需要备份的数据库是test这个数据库, 

如果需要备份多个数据库，那么应该写多行,如下所示: 

binlog-do-db=backup1 

binlog-do-db=backup2 

binlog-do-db=backup3 

解释: 

1\) server-id=2表示本机器的序号, A,B的server-id 不能相同; 

2\)log-bin表示打开binlog,打开该选项才可以通过I/O写到Slave的relay-log,也是可以进行replication的前提; 

其中mysql\_binary\_log是日志文件的名称，mysql将建立不同扩展名，文件名为mysql\_binary\_log的几个日志文件. 

3\) master-host="192.168.1.23" 表示A做slave时的master为192.168.1.23; 

4\) master-user=root 这里表示master上开放的一个有权限的用户,使其可以从slave连接到master并进行复制; 

5\) master-password=123456 表示授权用户的密码; 

6\) master-port=3306 master上MySQL服务Listen3306端口; 

7\) master-connect-retry=60 同步间隔时间; 

8\) replicate-do-db=test 表示同步backup数据库; 

最后重新启动两台机器的mysql. 

------------------------------------------------ 

查看状态 及调试 

1,查看master的状态 

SHOW MASTER STATUS; 

Position 不应为0 

2,查看slave的状态 

show slave status; 

Slave\_IO\_Running \| Slave\_SQL\_Running 这两个字段 应为 YES\|YES. 

show processlist; 

会有两条记录与同步有关 state为 Has read all relay log; waiting for the slave I/O thread to update it 

和s Waiting for master to send event . 

3,错误日志 

MySQL安装目录dataHostname.err 

主服务器上的相关命令： 

show master status 

show slave hosts 

show {master\|binary} logs 

show binlog events 

purge {master\|binary} logs to ’log\_name’ 

purge {master\|binary} logs before ’date’ 

reset master\(老版本flush master\) 

set sql\_log\_bin={0\|1} 

----------------------------------------------------------------------------------- 

从服务器上的相关命令: 

slave start 

slave stop 

SLAVE STOP IO\_THREAD //此线程把master段的日志写到本地 

SLAVE start IO\_THREAD 

SLAVE STOP SQL\_THREAD //此线程把写到本地的日志应用于数据库 

SLAVE start SQL\_THREAD 

reset slave 

SET GLOBAL SQL\_SLAVE\_SKIP\_COUNTER 

load data from master 

show slave status\(SUPER,REPLICATION CLIENT\) 

CHANGE MASTER TO MASTER\_HOST=, MASTER\_PORT=,MASTER\_USER=, MASTER\_PASSWORD= //动态改变master信息 

PURGE MASTER \[before ’date’\] 删除master端已同步过的日志 



