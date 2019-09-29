特点

1. 可以存储不同结果的数据
1. 主要用来存储JSON格式的数据
1. 存储的文档就是一个对象,是json扩展Bson形式

安装

1. 下载地址:https://www.mongodb.com/download-center#community
1. 可视化工具:https://robomongo.org/

    linux下:
    
    1. 解压下载好的文件
    2. 将解压好的文件mv到/usr/local/mongodb
    3. 将可执行文件添加到环境变量:
        1. export PATH=/usr/local/mongodb/bin:$PATH

    windows下:

    1. 启动服务:
         1. mongod --dbpath D:\MongoDB\data
    1. 匹配mongo为window下的系统服务:
         1. mongod --dbpath D:\MongoDB\data --install
         1. mongod --dbpath D:\MongoDB\data --logpath=D:\MongoDB\logs\mongodb.log --logappend
    1. 如果服务还是创建不成功，可以尝试：
         1. sc create MongoDB binPath= "C:\Program Files\MongoDB\Server\3.6\bin\mongod.exe --service 
          --dbpath D:\mongodb\data --logpath=D:\mongodb\logs\mongodb.log  --logappend"
    1. 启动服务:net start MongoDB 如果要删除服务:sc delete MongoDB

基本操作:

	1) 查看数据库:show dbs
	2) 使用数据库:use 数据库名
	3) 创建一个集合:db.createCollection("first")
	4) 查看当前数据库下的集合数:show collections
	5) 删除一个集合:db.集合名.drop()
	6) 插入一条数据:db.集合名.insert({"name":"张三"})
	7) 查询:
		a)db.集合名.find()
		b)db.集合名.findOne()
	8) 更新:db.集合名.update({"name":"张三"},{$set:{"name":"李四"}})
	9) 删除:db.集合名.remove({"name":"张三"})

运算符

	1) 大于/小于:db.集合名.find({"age":{$gte:18}})
	2) or的使用:db.集合名.find({$or:[{"name":"张三"},{"age":{$gte:18}}]})

正则

	db.集合名.find({"name":{$regex:"三"}})

分页操作

	1) db.集合名.find().limit(5) 读取指定文档
	2) db.集合名.find().skip(4) 跳过指定文档,表示从第五条开始查询

排序

	1. db.集合名.find().sort({"age":1}) # -1降序 1是正序

统计个数

	1. db.集合名.count()

Python操作

	1) pip install pymongo
	2) 连接数据:
		1. conn = pymongo.MongoClient(host=host,port=27017)
		1. db = conn.数据库名
		1. coll = db.集合名

常规操作:

insert update remove find  find_one

**MongoDB使用**

```
use test 			              创建数据库
db.createCollection("runoob") 	  创建集合

show tables 			          查看集合
show colletions 		          查看集合

document=({"name":"菜鸟教程"，    定义数据变量
	  list:[1,3],likes:100})
db.myco12.insert(document)	      插入数据

db.myco12.drop()                  删除集合

db.myco12.update({likes:100},     更新所有文档，multi表示匹配所有
  {$set:{likes:10}},{multi:true})

db.collection.find(query, projection)   	     查询语句
db.col.find({key1:value1,key2:value2}).pretty()  查文档,pretty表示易读
```

**MongoDB连接python**

```
pip install pymongo

import pymongo
 
myclient = pymongo.MongoClient("mongodb://localhost:27017/")   连接MongoDB

mydb = myclient["runoobdb"]                                    创建数据库
mycol = mydb["sites"]                                          创建集合

mydict = { "name": "RUNOOB", "alexa": "10000", "url": "https://www.runoob.com" }
mycol.insert_one(mydict)                                       插入文档

mylist = [
  { "name": "Taobao", "alexa": "100", "url": "https://www.taobao.com" },
  { "name": "QQ", "alexa": "101", "url": "https://www.qq.com" },
  { "name": "Facebook", "alexa": "10", "url": "https://www.facebook.com" },
  { "name": "知乎", "alexa": "103", "url": "https://www.zhihu.com" },
  { "name": "Github", "alexa": "109", "url": "https://www.github.com" }
]
mycol.insert_many(mylist)                                     插入多个文档

myquery = { "name": { "$regex": "^F" } }
newvalues = { "$set": { "alexa": "123" } }
mycol.update_many(myquery, newvalues)                         修改所有文档

```

**更多参考**

[菜鸟教程](https://www.runoob.com/python3/python-mongodb-delete-document.html)
