1. install docker 

2. get mongodb images

   `docker search mongo`

​       `docker pull mongo` 

3. run docker container

   `docker run -d -p 27017:27017 -v <your local path>:/data/configdb -v <your local path>:/data/db --name mongo docker.io/mongo`

   Eg: 

   ```shell
   docker run -d -p 27017:27017 -v /Users/lisa/Desktop/mongodb/data/configdb:/data/configdb -v /Users/lisa/Desktop/mongodb/data/db:/data/db --name mongo docker.io/mongo
   
   ```

   

4. check docker container

   `docker ps`

5. Get into docke container

   `docker exec -it mongo mongo admin`

6. 在mongo命令行输入命令创建管理员账户

   `db.createUser({ user: 'lisa', pwd: 'some-initial-password', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });`

   Eg:

```
> db.createUser({ user: 'lisa', pwd: '1234', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });
Successfully added user: {
	"user" : "lisa",
	"roles" : [
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		}
	]
}
```

7. ### 创建数据库并设置用户

   目前为止我们一直都是在操作mongo自带的admin库。正常情况下是不建议直接使用这个库的。我们需要自己创建数据库并设置新库的用户。

   还是先使用`docker exec -it mongo mongo admin`命令进入mongo的命令行页面。
    使用上一步骤创建的管理员账户进行授权

   `db.auth("jsmith","some-initial-password");`

   切换到test库（如不存在会自动创建）

   `use test`

   创建test库下的用户

   `db.createUser({ user: 'test', pwd: '123456', roles: [{ role: "readWrite", db: "test" }] });`

   ![img](https://upload-images.jianshu.io/upload_images/4399845-6997875b8a3afba3.png?imageMogr2/auto-orient/strip|imageView2/2/w/884/format/webp)



## 倒入.bason 文件

1. 在本机的`/data/db` 文件中加文件夹`jr-data`放入.bason 文件

2. ![Screen Shot 2021-10-01 at 12.42.08 pm](/Users/wangyiyi/Library/Application Support/typora-user-images/Screen Shot 2021-10-01 at 12.42.08 pm.png)

3. 进入docker container

   `docker ps` 	查看containerid。例如`efc465148c3e`

   `sudo docker exec -it efc465148c3e /bin/bash`

1. 进入container之后

   到data 文件夹下看到本机上的configdb 和db两个文件夹

   ![Screen Shot 2021-10-01 at 12.39.54 pm](/Users/wangyiyi/Library/Application Support/typora-user-images/Screen Shot 2021-10-01 at 12.39.54 pm.png)

2. 进入db-->jr-data

   看到有user.bson文件就说明数据卷构建成功

   ![Screen Shot 2021-10-01 at 12.41.19 pm](/Users/wangyiyi/Library/Application Support/typora-user-images/Screen Shot 2021-10-01 at 12.41.19 pm.png)

3. 查看当前路径

   ```
   root@efc465148c3e:/data/db/jr-data# pwd
   /data/db/jr-data
   ```

4. 倒入.bason文件到mongodb

   `mongorestore -d jr_db 文件名.bson`

   ```shell
   root@efc465148c3e:/data/db/jr-data# mongorestore -d jr_db User.bson
   2021-10-01T02:26:32.357+0000	checking for collection data in User.bson
   2021-10-01T02:26:32.684+0000	restoring jr_db.User from User.bson
   2021-10-01T02:26:32.892+0000	finished restoring jr_db.User (1 document, 0 failures)
   2021-10-01T02:26:32.894+0000	1 document(s) restored successfully. 0 document(s) failed to restore.
   ```

5. 命令行敲`mongo`

   就可以启动mongodb啦

6. 用之前设置的用户名登陆

   `\> db.auth("lisa","1234");` 到jr_db（你自己创建的db）

   ```shell 
   > use jr_db #没有这个db 会自己创建
   switched to db jr_db
   > show jr_db
   uncaught exception: Error: don't know how to show [jr_db] :
   shellHelper.show@src/mongo/shell/utils.js:1211:11
   shellHelper@src/mongo/shell/utils.js:838:15
   @(shellhelp2):1:1
   ```

   ```shell
   > show collections #查看tables
   User
   > db.User.find();
   { "_id" : ObjectId("5c04689b57842350e256c3c8"), "name" : { "first" : "Admin", "last" : "JiangRen" }, "role" : "student", "tags" : [ ], "isAdmin" : true, "isVerified" : false, "email" : "ozitquan@gmail.com", "password" : "$2a$10$HhUC7AHBO9l8hLYjbIN8.OE1mXe2Byh4gsJ49lPdbCn0pBhdtEFzG", "__v" : 0 }
   > 
   
   ```

   

