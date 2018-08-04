# MongoDB 常用语法和命令

开启用户认证步骤





1.修改配置文件 /usr/local/mongodb-linux-x86\_64-rhel62-3.6.5/etc/mongodb.conf 打开auth的注释，设置auth=true

2.kill进程，重新启动 ./mongod --config /usr/local/mongodb-linux-x86\_64-rhel62-3.6.5/etc/mongodb.conf

3.添加管理员：

   

使用命令mongo进入命令行 ，创建第一个用户，该用户需要有用户管理权限，这里设置其角色为root



&gt; use admin

switched to db admin



&gt; db.createUser\({user:"root",pwd:"root",roles:\["root"\]}\)

Successfully added user: { "user" : "root", "roles" : \[ "root" \] }



新增的用户在system.users中，执行命令前，需要先认证



&gt; db.auth\("root", "root"\)

1

&gt; db.getCollectionNames\(\)



\[ "system.users", "system.version" \]



4.添加数据库用户:



第一个用户添加完成后，便需要认证才能继续添加其他用户:



db.auth\("root", "root"\)



&gt; use runoob

switched to db runoob



&gt; db.createUser\(

   {

    user: "runoob",

    pwd:  "runoob",

    roles: \[{ role: "dbOwner", db: "runoob" }\]

   }

\)

Successfully added user: {

        "user" : "runoob",

        "roles" : \[

                {

                        "role" : "dbOwner",

                        "db" : "runoob"

                }

        \]

}



查看用户



use admin



db.system.users.find\(\)



      





新建用户 3.0版本以后的语法：





use runoob

db.createUser\(

   {

    user: "runoob",

    pwd:  "runoob",

    roles: \[ "readWrite", "dbAdmin" \]

   }

\)



删除用户：





db.system.users.remove\({user:"java1"}\);



用户授权验证：





db.auth\('runoob','runoob'\)  //  输出结果1，用户存在，验证成功





服务器启动命令：





./mongod --config /usr/local/mongodb-linux-x86\_64-rhel62-3.6.5/etc/mongodb.conf

./mongod --config /usr/local/mongodb-linux-x86\_64-rhel62-3.6.5/etc/mongodb.conf --auth

