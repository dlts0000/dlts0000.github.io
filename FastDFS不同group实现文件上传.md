# <center>FastDFS不同group实现文件上传
## 1. 原来的docker启动方式
首先在home下面创建文件夹`fdfs_data`  
下面再创建三个文件夹
`fastdfs_tracker` `fastdfs_storage` `fastdfs_storage_data`

trakcer启动方式：

	docker run -d --name tracker --net=host -v /home/fdfs_data:/data/fast_data ImageID sh tracker.sh
storage启动方式：

	docker run -d --name storage --net=host -v /home/fdfs_data:/data/fast_data -e TRACKER_IP=192.168.1.109:22122 -e GROUP_NAME=group1 Image_ID sh storage.sh
	
## 2. 原来的文件上传方式
python客户端上传文件:

	# -*- coding:utf-8 -*-
	from fdfs_client.client import *
	import time	
	client_file = './fdfs/client.conf'
	test_file = './fdfs/123.jpg'
	
	def func_1():
	    client = Fdfs_client(client_file)
	    # upload
	    ret_upload = client.upload_by_filename(test_file)
	    print ret_upload
	    
	if __name__ == '__main__':
   		func_3()
   
## 3. 多group的启动方式
对于多个group，我们采用配置文件挂载进docker的方式实现不同group的文件配置，其中主要的修改包括storage端口的修改，本group文件路径的修改，nginx的配置修改等。
### 3.1  tracker启动方式不变
	docker run -d --name tracker --net=host -v /home/fdfs_data:/data/fast_data ImageID sh tracker.sh
### 3.2 storage配置1(/etc/fdfs/storage.conf)

在group1 中 `storage.conf`为：

	# the name of the group this storage server belongs to
	group_name=group3
	
	# the storage server port
	port=23000（默认）
	
	# the base path to store data and log files
	base_path=/data/fast_data/fastdfs_storage  (不同group一致即可)
	
	# store_path#, based 0, if store_path0 not exists, it's value is base_path
	# the paths must be exist
	store_path0=/data/fast_data/fastdfs_storage_data  (不同group一致即可)
	
	# tracker_server can ocur more than once, and tracker_server format is
	# "host:port", host can be hostname or ip address
	tracker_server=192.168.1.101:22122
	
	# the port of the web server on this storage server
	http.server_port=8888（默认）

在group3中  `storage.conf`主要配置为

	# the name of the group this storage server belongs to
	group_name=group3
	
	# the storage server port
	port=23001(重点修改，同一台服务器设置为不同，不同台服务器采用默认即可)
	
	# the base path to store data and log files
	base_path=/data/fast_data/fastdfs_storage  (不同group一致即可)
	
	# store_path#, based 0, if store_path0 not exists, it's value is base_path
	# the paths must be exist
	store_path0=/data/fast_data/fastdfs_storage_data  (不同group一致即可)
	
	# tracker_server can ocur more than once, and tracker_server format is
	# "host:port", host can be hostname or ip address
	tracker_server=192.168.1.101:22122(重点修改)
	
	# the port of the web server on this storage server
	http.server_port=8887(重点修改，同一台服务器设置为不同，不同台服务器采用默认即可)
### 3.2 storage配置2(/etc/fdfs/mod_fastdfs.conf)
在group1中，	`mod_fastdfs.conf`主要配置为

	# the base path to store log files
	base_path=/data/fast_data/fastdfs_storage
	
	# FastDFS tracker_server can ocur more than once, and tracker_server format is
	# "host:port", host can be hostname or ip address
	# valid only when load_fdfs_parameters_from_tracker is true
	tracker_server=192.168.1.101:22122
	
	# the port of the local storage server
	# the default value is 23000
	storage_server_port=23001
	
	# the group name of the local storage server
	group_name=group3
	
	# path(disk or mount point) count, default value is 1
	# must same as storage.conf
	store_path_count=1
	
	# store_path#, based 0, if store_path0 not exists, it's value is base_path
	# the paths must be exist
	# must same as storage.conf
	store_path0=/data/fast_data/fastdfs_storage_data  (不同group一致即可)
	
	# set the group count
	# set to none zero to support multi-group on this storage server
	# set to 0  for single group only
	# groups settings section as [group1], [group2], ..., [groupN]
	# default value is 0
	# since v1.14
	group_count = 1
	
	[group1]
	group_name=group3
	storage_server_port=23001
	store_path_count=1
	store_path0=/data/fast_data/fastdfs_storage_data （不同group一致即可）

	
	

在group3，`mod_fastdfs.conf`主要配置为

	# the base path to store log files
	base_path=/data/fast_data/fastdfs_storage
	
	# FastDFS tracker_server can ocur more than once, and tracker_server format is
	# "host:port", host can be hostname or ip address
	# valid only when load_fdfs_parameters_from_tracker is true
	tracker_server=192.168.1.101:22122
	
	# the port of the local storage server
	# the default value is 23000
	storage_server_port=23001（重点修改）
	
	# the group name of the local storage server
	group_name=group3（重点修改）
	
	# path(disk or mount point) count, default value is 1
	# must same as storage.conf
	store_path_count=1
	
	# store_path#, based 0, if store_path0 not exists, it's value is base_path
	# the paths must be exist
	# must same as storage.conf
	store_path0=/data/fast_data/fastdfs_storage_data
	
	# set the group count
	# set to none zero to support multi-group on this storage server
	# set to 0  for single group only
	# groups settings section as [group1], [group2], ..., [groupN]
	# default value is 0
	# since v1.14
	group_count = 1
	
	[group1]
	group_name=group3
	storage_server_port=23001
	store_path_count=1
	store_path0=/data/fast_data/fastdfs_storage_data
### 3.3 storage中nginx配置(/etc/nginx/conf/nginx.conf)
在group1中，`nginx.conf`配置为

	 server {
        listen       7001;  (重点修改，不同服务器可以一致)
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        location  /group3/M00 {     （重点修改，group名字与实际group一致）
                    alias /data/fast_data/data;
                    ngx_fastdfs_module;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


在group3中，`nginx.conf`配置为 

	 server {
        listen       7002;  (重点修改，不同服务器可以一致)
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        location  /group3/M00 {     （重点修改，group名字与实际group一致）
                    alias /data/fast_data/data;
                    ngx_fastdfs_module;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
    
## 4.启动脚本（tracker.sh, storage.sh）
### 4.1 原docker中的tracker.sh
	#!/bin/sh
	/data/fastdfs/tracker/fdfs_trackerd /etc/fdfs/tracker.conf
	tail -f /data/fast_data/fastdfs_tracker/logs/trackerd.log
### 4.2 原docker中的storage.sh
	#!/bin/sh
    sed "s/^.*tracker_server=.*$/tracker_server=$TRACKER_IP/" /etc/fdfs/storage.conf > storage.conf
    sed "s/^.*group_name=.*$/group_name=$GROUP_NAME/" storage.conf > storage_.conf
    cp storage_.conf /etc/fdfs/storage.conf
    /data/fastdfs/storage/fdfs_storaged /etc/fdfs/storage.conf
    sed "s/^.*tracker_server=.*$/tracker_server=$TRACKER_IP/" /etc/fdfs/mod_fastdfs.conf > mod_fastdfs.conf
    sed "s/^.*group_name=.*$/group_name=$GROUP_NAME/" mod_fastdfs.conf > mod_fastdfs_.conf
    cp mod_fastdfs_.conf /etc/fdfs/mod_fastdfs.conf
    /etc/nginx/sbin/nginx
    tail -f /data/fast_data/fastdfs_storage/logs/storaged.log
### 4.3 挂载方式的tracker.sh
	#!/bin/sh
	/data/fastdfs/tracker/fdfs_trackerd /etc/fdfs/tracker.conf
	tail -f /data/fast_data/fastdfs_tracker/logs/trackerd.log
### 4.4 挂载方式的storge.sh
	#!/bin/sh
    /data/fastdfs/storage/fdfs_storaged /etc/fdfs/storage.conf
    /etc/nginx/sbin/nginx
    tail -f /data/fast_data/fastdfs_storage/logs/storaged.log
说明：  
tracker启动方式不变  
由于storage中无需通过数据传入的方式修改ip、port、group_name等参数,因此storage.sh脚本只需启动相关的命令即可。

## 5 挂载模式的启动方式
首先在home下面创建文件夹`fdfs_data_2`  
下面再创建三个文件夹
`fastdfs_tracker` `fastdfs_storage` `fastdfs_storage_data`

storage启动方式为:

	docker run -d --name storage_3 --net=host -v /home/fdfs_data_2:/data/fast_data -v /home/fdfs_config2/fdfs:/etc/fdfs/ -v /home/fdfs_config2/conf:/etc/nginx/conf -v /home/fdfs_config2/start:/home/fdfs_config2/start c33892d16c9e sh /home/fdfs_config2/start/my_storage.sh
	
## 6 多group的python文件操作
### 6.1 根据文件获取trakcer原理
将配置文件中的tracker相关的参数读入tracker字典中

	def get_tracker_conf(conf_path='client.conf'):
    cf = Fdfs_ConfigParser()
    tracker = {}
    try:
        cf.read(conf_path)
        timeout = cf.getint('__config__', 'connect_timeout')
        tracker_list = cf.get('__config__', 'tracker_server')
        if isinstance(tracker_list, str):
            tracker_list = [tracker_list]
        tracker_ip_list = []
        for tr in tracker_list:
            tracker_ip, tracker_port = tr.split(':')
            tracker_ip_list.append((tracker_ip, tracker_port))
        tracker['host_tuple'] = tuple(tracker_ip_list)
        tracker['timeout'] = timeout
        tracker['name'] = 'Tracker Pool'
    except:
        raise
    return tracker
### 6.2 查看groups
	# coding=utf-8
	from fdfs_client.client import Fdfs_client, fdfs_check_file, DataError, Tracker_client
	from fdfs_client.utils import Fdfs_ConfigParser
	from fdfs_client.connection import ConnectionPool
	
	def func_1():
    	trackers = get_tracker_conf(conf_path='./fdfs/client.conf')
    	tracker_pool = ConnectionPool(**trackers)
    	tc = Tracker_client(tracker_pool)
    	# store_serv = tc.tracker_query_storage_stor_without_group()
    	groups = tc.tracker_list_all_groups()
    	print(groups)
    	
结果示例：

	{'Groups count': 3, 'Groups': [<fdfs_client.tracker_client.Group_info object at 0x106797dd0>, <fdfs_client.tracker_client.Group_info object at 0x106797f10>, <fdfs_client.tracker_client.Group_info object at 0x106797f50>]}

### 6.3 基于group的文件上传
	def upload_by_file_name(filename, meta_dict = None):
    trackers = get_tracker_conf(conf_path='./fdfs/client.conf')
    tracker_pool = ConnectionPool(**trackers)

    isfile, errmsg = fdfs_check_file(filename)
    if not isfile:
        raise DataError(errmsg + '(uploading)')
    tc = Tracker_client(tracker_pool)
    # store_serv = tc.tracker_query_storage_stor_without_group()
    store_serv = tc.tracker_query_storage_stor_with_group(group_name='group2')
    groups = tc.tracker_list_all_groups()
    print(groups)

    old_client = Fdfs_client('./fdfs/client.conf')

    return old_client.get_storage(store_serv).storage_upload_by_filename(tc, store_serv, filename, meta_dict)
    
上述函数其实就是对原来自带的client.upload_by_filename(test_file)函数的改造，替换其中的一个函数，源码为：

    def upload_by_filename(self, filename, meta_dict = None):
	    '''
	    Upload a file to Storage server.
	    arguments:
	    @filename: string, name of file that will be uploaded
	    @meta_dict: dictionary e.g.:{
	        'ext_name'  : 'jpg',
	        'file_size' : '10240B',
	        'width'     : '160px',
	        'hight'     : '80px'
	    } meta_dict can be null
	    @return dict {
	        'Group name'      : group_name,
	        'Remote file_id'  : remote_file_id,
	        'Status'          : 'Upload successed.',
	        'Local file name' : local_file_name,
	        'Uploaded size'   : upload_size,
	        'Storage IP'      : storage_ip
	    } if success else None
	    '''
	    isfile, errmsg = fdfs_check_file(filename)
	    if not isfile:
	        raise DataError(errmsg + '(uploading)')
	    tc = Tracker_client(self.tracker_pool)
	    store_serv = tc.tracker_query_storage_stor_without_group()
	    return self.get_storage(store_serv).storage_upload_by_filename(tc, 	store_serv, filename, meta_dict)
	    
结果示例：
	
	{'Status': 'Upload successed.', 'Storage IP': '192.168.1.114', 'Remote file_id': 'group2/M00/00/00/wKgBclueZ-6ABJXTAABSnLua4bo791.jpg', 'Group name': 'group2', 'Local file name': './123.jpg', 'Uploaded size': '20.00KB'}

