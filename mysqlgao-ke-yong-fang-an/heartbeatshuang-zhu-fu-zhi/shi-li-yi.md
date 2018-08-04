# 示例一——[heartbeat实现MySQL双机高可用](https://www.linuxzen.com/heartbeatshi-xian-mysqlshuang-ji-gao-ke-yong.html)

对于一个网站或一个企业最重要的无疑就是数据,那么数据库的数据安全无疑就更加重要,所以我们必须保证数据库的数据完整,这里就介绍使用heartbeat来实现MySQL双机高可用.

当我们的MySQL数据库故障或MySQL数据库服务器出现故障的时候我们希望有一个备用能自动代替主MySQL数据来完成当前的任务,当主MySQL服务器恢复故障的时候备用的能切换到备用等待下一次故障出现.这里我们就结合故障检测HA来实现.

HA会定时发送心跳包检测主备服务器的健康状态,当主服务器出现故障时会自动将vip切换到备用服务器,由备用服务器执行主服务器的任务,MySQL要实现这样的功能就必须保证主备服务器的数据一致.这就要用到MySQL主从双机.本文使用环境: 系统:CentOS 5.5 32位 主MySQL: ip 192.168.3.101/24 主机名:master.org 备用MySQL:192.168.3.102/24   主机名:slave.org vip:192.168.3.103/24 MySQL:mysql-5.0.95.tar.gz heartbeat:Heartbeat-3-0-7e3a82377fa8.tar.bz2



