# <center>mongo-connetor操作</center>
## 1. 启动mongo-connector
	cd /home/connectors
	mongo-connector -c baike.json &
	mongo-connector -c multimedia.json &

## 2.mongo-connector配置
	{
	    "__comment__": "Configuration options starting with '__' are disabled",
	    "__comment__": "To enable them, remove the preceding '__'",
	
	    "mainAddress": "192.168.178.100:27017",
	    "oplogFile": "/home/connectors/baike/oplog.timestamp",
	    "noDump": true,
	    "batchSize": 50,
	    "verbosity": 1,
	    "continueOnError": false,
	
	    "logging": {
	        "type": "file",
	        "filename": "/home/connectors/baike/mongo-connector.log",
	        "__format": "%(asctime)s [%(levelname)s] %(name)s:%(lineno)d - %(message)s",
	        "__rotationWhen": "D",
	        "__rotationInterval": 1,
	        "__rotationBackups": 10,
	
	        "__type": "syslog",
	        "__host": "localhost:514"
	    },
	
	    "authentication": {
	        "__adminUsername": "username",
	        "__password": "password",
	        "__passwordFile": "mongo-connector.pwd"
	    },
	
	    "__comment__": "For more information about SSL with MongoDB, please see http://docs.mongodb.org/manual/tutorial/configure-ssl-clients/",
	    "__ssl": {
	        "__sslCertfile": "Path to certificate to identify the local connection against MongoDB",
	        "__sslKeyfile": "Path to the private key for sslCertfile. Not necessary if already included in sslCertfile.",
	        "__sslCACerts": "Path to concatenated set of certificate authority certificates to validate the other side of the connection",
	        "__sslCertificatePolicy": "Policy for validating SSL certificates provided from the other end of the connection. Possible values are 'required' (require and validate certificates), 'optional' (validate but don't require a certificate), and 'ignored' (ignore certificates)."
	    },
	
	    "__fields": ["field1", "field2", "field3"],
	
	    "namespaces": {
		"item_baike.baike": {
	            "rename": "item_baike.baike"
	        }
	    },
	
	    "docManagers": [
	        {
	            "docManager": "elastic2_doc_manager",
	            "targetURL": "192.168.178.100:9200",
	            "bulkSize": 50,
		    "autoCommitInterval": 1,
	            "__uniqueKey": "_id",
	            "__autoCommitInterval": null
	        }
	    ]
	}

	