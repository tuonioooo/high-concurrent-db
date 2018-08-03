# Oracle数据库——[百科](https://baike.baidu.com/item/Oracle数据库/3710800?fr=aladdin)

## 概述

Oracle Database，又名Oracle RDBMS，或简称Oracle。是甲骨文公司的一款关系数据库管理系统。它是在数据库领域一直处于领先地位的产品。可以说Oracle数据库系统是目前世界上流行的关系数据库管理系统，系统可移植性好、使用方便、功能强，适用于各类大、中、小、微机环境。它是一种高效率、可靠性好的 适应高吞吐量的数据库解决方案。

## 系统简介

ORACLE数据库系统是美国ORACLE公司（甲骨文）提供的以分布式数据库为核心的一组软件产品，是目前最流行的客户/服务器\(CLIENT/SERVER\)或B/S体系结构的数据库之一。比如SilverStream就是基于数据库的一种中间件。ORACLE数据库是目前世界上使用最为广泛的数据库管理系统，作为一个通用的数据库系统，它具有完整的数据管理功能；作为一个关系数据库，它是一个完备关系的产品；作为分布式数据库它实现了分布式处理功能。但它的所有知识，只要在一种机型上学习了ORACLE知识，便能在各种类型的机器上使用它。

Oracle数据库最新版本为Oracle Database 12c。Oracle数据库12c 引入了一个新的多承租方架构，使用该架构可轻松部署和管理数据库云。此外，一些创新特性可最大限度地提高资源使用率和灵活性，如Oracle Multitenant可快速整合多个数据库，而Automatic Data Optimization和Heat Map能以更高的密度压缩数据和对数据分层。这些独一无二的技术进步再加上在可用性、安全性和大数据支持方面的主要增强，使得Oracle数据库12c 成为私有云和公有云部署的理想平台。

## 支持平台

在2001年发布的Oracle9i之前，甲骨文公司把他们的数据库产品广泛的移植到了不同的平台上。近期，甲骨文公司巩固了一小部分的操作系统平台。

截止至2015年01月，甲骨文公司的Oracle10g/11g/12c支持以下的操作系统和硬件：

* AppleMac OS X Server:PowerPC
* HPHP-UX:PA-RISC,Itanium
* HPTru64 UNIX:Alpha
* HPOpenVMS: Alpha, Itanium
* IBMAIX5L:IBM POWER
* IBMz/OS:zSeries
* Linux:x86,x86-64, PowerPC, zSeries, Itanium
* MicrosoftWindows: x86, x86-64, Itanium
* SunSolaris:SPARC, x86, x86-64

## 特点

1、完整的数据管理功能：

1）数据的大量性

2）数据的保存的持久性

3）数据的共享性

4）数据的可靠性

2、完备关系的产品：

1）信息准则---关系型DBMS的所有信息都应在逻辑上用一种方法，即表中的值显式地表示；

2）保证访问的准则

3）视图更新准则---只要形成视图的表中的数据变化了，相应的视图中的数据同时变化

4）数据物理性和逻辑性独立准则

3、分布式处理功能：

ORACLE数据库自第5版起就提供了分布式处理能力，到第7版就有比较完善的分布式数据库功能了，一个ORACLE分布式数据库由oraclerdbms、sql\*Net、SQL\*CONNECT和其他非ORACLE的关系型产品构成。

4、用ORACLE能轻松的实现数据仓库的操作。

这是一个技术发展的趋势，不在这里讨论。

优点

■ 可用性强

■ 可扩展性强

■ 数据安全性强

■ 稳定性强

## 工具简介

Navicat for Oracle是一套专为Oracle设计的强大数据库管理及开发工具。它可以用于任何版本的Oracle数据库，并支援大部份Oracle的功能，包括触发器、索引、检视等。

·Toad for Oracle是一款老牌的Oracle开发管理工具，比任何一款Oracle开发管理工具功能更多，并针对使用者不同的角色有多个分支版本。版本包括：Toad DBA Suite for Oracle是一款专门为Oracle DBA管理Oracle数据库工具, Toad Development Suite for Oracle是一款专门为Oracle开发工具， Toad DBA Suite for Oracle – Exadata Edition是一款专门为Oracle Exadata一体服务器及Oracle数据库管理工具, Toad DBA Suite for Oracle - RAC Edition是一款专门为Oracle搭建集群RAC的DBA管理工具

## 逻辑结构

它由至少一个表空间和数据库模式对象组成。这里，模式是对象的集合，而模式对象是直接引用数据库数据的逻辑结构。模式对象包括这样一些结构：表、视图、序列、存储过程、同义词、索引、簇和数据库链等。逻辑存储结构包括表空间、段和范围，用于描述怎样使用数据库的物理空间。

总之,逻辑结构由逻辑存储结构\(表空间,段,范围,块\)和逻辑数据结构\(表、视图、序列、存储过程、同义词、索引、簇和数据库链等\)组成,而其中的模式对象\(逻辑数据结构\)和关系形成了数据库的关系设计。

段（Segment）：

是表空间中一个指定类型的逻辑存储结构，它由一个或多个范围组成，段将占用并增长存储空间。

其中包括：

数据段：用来存放表数据；

索引段：用来存放表索引；

临时段：用来存放中间结果；

回滚段：用于出现异常时，恢复事务。

范围（Extent）：是数据库存储空间分配的逻辑单位，一个范围由许多连续的数据块组成，范围是由段依次分配的，分配的第一个范围称为初始范围，以后分配的范围称为增量范围。

数据块（Block）：

是数据库进行IO操作的最小单位，它与操作系统的块不是一个概念。oracle数据库不是以操作系统的块为单位来请求数据，而是以多个Oracle数据库块为单位。

![](/assets/import-oracle-01.png)

## 文件结构

数据库的物理存储结构是由一些多种物理文件组成，主要有数据文件、控制文件、重做日志文件、归档日志文件、参数文件、口令文件、警告文件等。

控制文件：存储实例、数据文件及日志文件等信息的二进制文件。alter system set control\_files=‘路径’。V$CONTROLFILE。

数据文件：存储数据，以.dbf做后缀。一句话：一个表空间对多个数据文件，一个数据文件只对一个表空间。dba\_data\_files/v$datafile。

日志文件：即Redo Log Files和Archivelog Files。记录数据库修改信息。ALTER SYSTEM SWITCH LOGFILE; 。V$LOG。

参数文件：记录基本参数。spfile和pfile。

警告文件：show parameter background\_dump\_dest---使用共享服务器连接

跟踪文件：show parameter user\_dump\_dest---使用专用服务器连接



## Oracle具体详细教程文档

Oracle 教程：[https://www.w3cschool.cn/oraclejc/oraclejc-dxgu2qqt.html](https://www.w3cschool.cn/oraclejc/oraclejc-dxgu2qqt.html) 



