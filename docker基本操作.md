# docker基本操作
### 1. docker离线安装与启动
离线安装docker时，``install_docker``文件夹中有``my_install.sh``，执行my_install.sh 可以将对应的文件拷贝到 /usr/local/bin目录中  

	cd /usr/local/bin
	./dockerd &
### 2. docker基本操作
查看所有镜像  
	
	docker images
查看运行的所有容器

	docker ps
查看所有容器

	docker ps -a

将docker镜像保存为文件

	docker save -o ./ner_app.tar ner_app:latest
将docker镜像文件(xx.tar)导入为镜像

	docker load --input ner_app.tar

进入镜像执行操作

	docker run -it --name my_name  --net=host -v /home/xxx:/home/xxx ner_app:latest /bin/bash

将修改后的容器保存为镜像

	docker commit <container_ID> elias/python2.7:latest

重新启动停止的容器

	docker start <container_ID>

进入正在运行的容器

	docker exec -it <container_ID> /bin/bash

停止正在运行的容器

	docker stop <container_ID>

删除停止的容器

	docker rm <container_ID>

删除镜像

	docker rmi  <image_ID>
	
## 3. 基于docker启动命名实体识别服务
ner服务中代码采用flask框架  代码已经写到镜像里，无需外挂，端口为5000

	docker run -d --name ner -p 5000:5000 ner_app:latest
	
## 4. 启动fastdfs文件系统服务
fastdfs 需要预先在/home创建文件夹  ``fdfs_data`` 以及子文件夹 ``fastdfs_storage`` ``fastdfs_storage_data`` ``fastdfs_tracker``

启动tracker  

	docker run -d --name tracker --net=host -v /home/fdfs_data:/data/fast_data ebd6ecf2296f sh tracker.sh

启动 storage和nginx (在同一个镜像中)

	docker run -d --name storage --net=host -v /home/fdfs_data:/data/fast_data -e TRACKER_IP=192.168.178.148:22122 -e GROUP_NAME=group1 ebd6ecf2296f sh storage.sh
	
## 5. 启动django主工程 
系统代码放置到``/home/Book/newdevelop``中  
其中``newdevelop``工程文件中有``start.sh``文件  
内容包括:

	source /etc/profile
	cd /home/Book/newdevelop
	python manage.py runserver 0.0.0.0:8006
其中 source /etc/profile 是为了解决 docker中的默认编码问题 必须执行

执行命令 

	docker run -d --name=baike_ll --net=host -v /home/Book/newdevelop:/home/Book/newdevelop elias/python2.7:latest sh /home/Book/newdevelop/start.sh
