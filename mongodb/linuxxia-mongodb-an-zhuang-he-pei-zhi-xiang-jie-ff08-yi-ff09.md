# Linux下MongoDB安装和配置详解（一）

原文：

[https://www.cnblogs.com/pfnie/articles/6759105.html](https://www.cnblogs.com/pfnie/articles/6759105.html)

## 一、创建MongoDB的安装路径

在/usr/local/  创建文件夹mongoDB

mkdir mongoDB

![](https://images2015.cnblogs.com/blog/1119433/201704/1119433-20170424210030412-62465254.png)

![](file:///C:/Users/tony/AppData/Local/Temp/enhtmlclip/1119433-20170424210030412-62465254.png)

## 二、上传文件到Linux上的/usr/local/source目录下

1. 我首先在[mongoDB下载路径](https://www.mongodb.org/dl/linux)下载mongoDB下载对应的版本.

2.通过FTP工具将安装包上传到linux机器上面.

![](file:///C:/Users/tony/AppData/Local/Temp/enhtmlclip/1119433-20170424210358303-227373191.png)

## 三、解压文件

1. 进入到/usr/local/source目录:

cd /usr/local/source

2. 运行如下命令: tar -zxvf mongodb-linux-i686-3.2.13-rc0.gz -C /usr/local/mongoDB

![](https://images2015.cnblogs.com/blog/1119433/201704/1119433-20170424210739397-1337671446.png)

![](file:///C:/Users/tony/AppData/Local/Temp/enhtmlclip/1119433-20170424210739397-1337671446.png)

3. 重命名

![](https://images2015.cnblogs.com/blog/1119433/201704/1119433-20170424211035475-119035000.png)

![](file:///C:/Users/tony/AppData/Local/Temp/enhtmlclip/1119433-20170424211035475-119035000.png)

## 四、创建配置文件

1. 创建数据库文件夹

cd /usr/local/mongoDB/mongodbserver

mkdir data

![](https://images2015.cnblogs.com/blog/1119433/201704/1119433-20170427204211850-523478259.png)

2. 创建日志文件夹

cd /usr/local/mongoDB/mongodbserver

mkdir log

![](https://images2015.cnblogs.com/blog/1119433/201704/1119433-20170427204613850-843678893.png)

3. 创建配置文件夹与配置文件

3.1 创建配置文件夹etc

cd /usr/local/mongoDB/mongodbserver

mkdir etc

![](https://images2015.cnblogs.com/blog/1119433/201704/1119433-20170427204740537-412044592.png)

3.2 创建配置文件mongodb.conf

cd /usr/local/mongoDB/mongodbserver/etc

vim mongodb.conf

```
dbpath=/usr/local/mongoDB/mongodbserver/data
logpath=/usr/local/mongoDB/mongodbserver/logs/mongodb.log
port=27017
fork=true
journal=false
storageEngine=mmapv1
```

## 五、启动MongoDB

1. mongodb安装好后第一次进入是不需要密码的，也没有任何用户，通过shell命令可直接进入，cd到mongodb目录下的bin文件夹，执行命令./mongo即可,如下所示:

./mongod --config /usr/local/mongoDB/mongodbserver/etc/mongodb.conf

![](https://images2015.cnblogs.com/blog/1119433/201704/1119433-20170427213355506-683931454.png)

![](file:///C:/Users/tony/AppData/Local/Temp/enhtmlclip/1119433-20170427213355506-683931454.png)

启动成功后，访问[http://npfdev1:27017/](http://npfdev1:27017/), 可以看到:

![](https://images2015.cnblogs.com/blog/1119433/201704/1119433-20170427213447006-163786569.png)

![](file:///C:/Users/tony/AppData/Local/Temp/enhtmlclip/1119433-20170427213447006-163786569.png)

2、添加管理用户\(mongoDB 没有无敌用户root，只有能管理用户的用户 userAdminAnyDatabase\)

利用mongo命令连接mongoDB服务器端：

![](https://images2015.cnblogs.com/blog/1119433/201704/1119433-20170427214410522-78868136.png)

![](file:///C:/Users/tony/AppData/Local/Temp/enhtmlclip/1119433-20170427214410522-78868136.png)

```
> use admin
switched to db admin
> db.createUser( {user: "pfnieadmin",pwd: "123456",roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]});
```

```
成功后，你将会看到:
```

![](https://images2015.cnblogs.com/blog/1119433/201705/1119433-20170507180442148-299759750.png)

![](file:///C:/Users/tony/AppData/Local/Temp/enhtmlclip/1119433-20170507180442148-299759750.png)

注：添加完用户后可以使用show users或db.system.users.find\(\)查看已有用户.

3、添加完管理用户后，关闭MongoDB，并使用权限方式再次开启MongoDB，这里注意不要使用kill直接去杀掉mongodb进程，（如果这样做了，请去data/db目录下删除mongo.lock文件），可以使用db.shutdownServer\(\)关闭.

4、使用权限方式启动MongoDB

在配置文件中添加：auth=true , 然后启动：

![](https://images2015.cnblogs.com/blog/1119433/201704/1119433-20170427215146615-276991171.png)

![](file:///C:/Users/tony/AppData/Local/Temp/enhtmlclip/1119433-20170427215146615-276991171.png)

5、进入mongo shell，使用admin数据库并进行验证，如果不验证，是做不了任何操作的。 

![](https://images2015.cnblogs.com/blog/1119433/201705/1119433-20170507180549507-1781757279.png)

![](file:///C:/Users/tony/AppData/Local/Temp/enhtmlclip/1119433-20170507180549507-1781757279.png)

```
> use admin
> db.auth("pfnieadmin","123456")   #认证，返回1表示成功
```

## 六、将mongod路径添加到系统路径中，方便随处执行mongod命令

1. 在/etc/profile文件中，添加 export PATH=$PATH:/usr/local/mongoDB/mongodbserver/bin

![](https://images2015.cnblogs.com/blog/1119433/201705/1119433-20170504211503226-993997839.png)

![](file:///C:/Users/tony/AppData/Local/Temp/enhtmlclip/1119433-20170504211503226-993997839.png)

2. 执行source /etc/profile，使系统环境变量立即生效

![](https://images2015.cnblogs.com/blog/1119433/201705/1119433-20170504211914757-119873179.png)

![](file:///C:/Users/tony/AppData/Local/Temp/enhtmlclip/1119433-20170504211914757-119873179.png)

## 七、将mongo路径软链到/usr/bin路径下，方便随处执行mongo命令

1. 执行命令: ln -s /usr/local/mongoDB/mongodbserver/bin/mongo  /usr/bin/mongo

![](https://images2015.cnblogs.com/blog/1119433/201705/1119433-20170504212307507-1404959127.png)

## 八、测试是否方便随处执行mongo命令

1. 回到任意路径下,执行mongo命令,连接mongod服务

![](https://images2015.cnblogs.com/blog/1119433/201705/1119433-20170504212619523-362012020.png)

2. 关闭mongod服务,执行db.shutdownServer\(\)

![](https://images2015.cnblogs.com/blog/1119433/201705/1119433-20170507180638586-1984680641.png)

```
2017-04-20T18:32:26.865+0800 E QUERY [thread1] Error: shutdownServer failed: {
"ok" : 0,
"errmsg" : "not authorized on admin to execute command { shutdown: 1.0 }",
"code" : 13
} :
_getErrorWithCode@src/mongo/shell/utils.js:25:13
DB.prototype.shutdownServer@src/mongo/shell/db.js:302:1
@(shell):1:1
```

解决办法，执行下面的语句，添加权限：

```
db.updateUser(
"pfnieadmin",
{
roles : [
{"role" : "userAdminAnyDatabase","db" : "admin"},
{"role" : "dbOwner","db" : "admin"},
{"role" : "clusterAdmin", "db": "admin"}
]
}
)
```

然后在执行db.shutdownServer\(\).

![](https://images2015.cnblogs.com/blog/1119433/201705/1119433-20170507180733195-669494826.png)

或者执行下面的命令关闭：

killall mongod

3. 启动mongod服务

mongod --config /usr/local/mongoDB/mongodbserver/etc/mongodb.conf

![](https://images2015.cnblogs.com/blog/1119433/201705/1119433-20170504214827148-118313394.png)

## 九、MongoDB设置为系统服务并且设置开机启动

1.通过上面简单的操作，我们已经将MongoDB配置文件配置完成，那么接下里我们将为MongoDB设置系统服务。

2.首先添加MongoDB系统服务，命令如下：vim /etc/rc.d/init.d/mongod

3.打开编辑器后，我们将下面的配置粘贴进去，然后保存

```
start() {
/usr/local/mongoDB/mongodbserver/bin/mongod --config /usr/local/mongoDB/mongodbserver/etc/mongodb.conf
}
stop() {
/usr/local/mongoDB/mongodbserver/bin/mongod --config /usr/local/mongoDB/mongodbserver/etc/mongodb.conf --shutdown
}
case "$1" in
start)
start
;;
stop)
stop
;;
restart)
stop
start
;;
*)
echo
$"Usage: $0 {start|stop|restart}"
exit 1
esac
```

4.保存完成之后，添加脚本执行权限，命令如下：chmod +x /etc/rc.d/init.d/mongod

5.启动MongoDB，service mongod start 如下图所示，则说明启动成功:

![](https://images2015.cnblogs.com/blog/1119433/201705/1119433-20170507154716914-744849120.png)

6.可以使用命令service mongod stop关闭MongoDB服务。

7. 验证mongoDB是否启动，输入命令lsof -i :27017，监测端口已经在使用中，所以说启动已经完成。

