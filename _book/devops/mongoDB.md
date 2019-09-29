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
