#<center>mongodb基本操作</center>
##1. 启动mongodb  
###1. 启动MongoDB复制集
```sh
./bin/mongod -f ./conf/27017.config &
./bin/mongod -f ./conf/27018.config &
./bin/mongod -f ./conf/27019.config &
```

###2. 查看MongoDB状态
```sh
cd /home/mongo/bin
./mongo 192.168.178.100:27017
```
进入mongo交互命令行界面后，执行  

```sh
> use admin
> rs.status()
```
## 2. mongodb副本集搭建步骤
###1. 建立数据文件夹
/home/mongo/db 下 三个文件夹 27017 27018 27019
###2. 配置文件
/home/mongo/conf/27017.conf  配置文件如下  

	dbpath = /home/mongo/db/27017  
	logpath = /home/mongo/log/27017.log  
	pidfilepath = /home/mongo/db/27017/27017.pid  
	replSet = mongo_rep  
	bind_ip = baike  
	port = 27017  
	oplogSize = 30720  
	logappend = true  
	fork = true  

###3. 启动mongo 
启动三个mongodb节点

	./bin/mongod -f ./conf/27017.config &
	./bin/mongod -f ./conf/27018.config &
	./bin/mongod -f ./conf/27019.config &

###4. 配置主、副、仲裁节点
	./mongo 192.168.178.100:27017
进入交互命令行后： 
 
	> use admin
	> cfg={ _id:"mongo_rep", members:[ {_id:0, host:'baike:27017',priority:10}, {_id:1,host:'baike:27018',priority:1},   
	           {_id:2,host:'baike:27019',arbiterOnly:true}] };  
	
	> rs.initiate(cfg)             #使配置生效 

## 3. mongodb数据导入和导出
导出集合数据到json文件，在原有服务器集群上进入mongo安装文件夹，执行:
 
	./mongoexport -h 192.168.178.199:30000 -d item_base_analysis -c country -o /home/item_base_analysisAcountry.json --type json

将导出文件导入数据库中，在集群的10服务器上，进入mongo安装文件夹，执行：

	mongoimport --port 30000 -d item_baike_anaysis  -c baike  --file item_baike_analysisAbaike.json --type json

