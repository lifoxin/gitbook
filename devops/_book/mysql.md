**安装mysql5.7**

	wget http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
	yum -y install mysql57-community-release-el7-11.noarch.rpm
	yum install mysql-community-server
**修改密码**

	vim /etc/my.cnf
	   [mysqld]
	   #取消密码登录
	   skip-grant-tables

	systemctl start  mysqld
	mysql -uroot -p 
	>update user set authentication_string=password('123456') where user='root';
	>flush privileges;
**再次登录修改密码**

	SET PASSWORD = PASSWORD('123456');
**修改密码强度限制**

	set global validate_password_policy=0
	set global validate_password_mixed_case_count=2
	SET PASSWORD = PASSWORD('123456')

**union all的用法**
	
	select a,b,c from table1
	union all
	select ca,cb,cc from table2


**基本用法**

	create table if not exists student (id int,name varchar(10));
	drop table 表名称                                                   
	truncate table 表名称                             
	delete from 表名称 where 列名称 = 值
	INSERT INTO 表名称 (列1, 列2,...) VALUES (值1, 值2,....)            
	DELETE FROM 表名称 WHERE 列名称 = 值                                
	UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值                  
	
**添加用户授权**

	grant select,insert,update,delete on book.* to test2@localhost identified by “abc”;   
**datepart,convert,substring**

	2018-08-11 00:00:00.0
	DATEPART(hour,senttime)  		取小时数
	convert(varchar,senttime,23) 	年月日，字段中时间
	
	substring(a.content,1,70)       截取内容字段
	substring(a.content,71,140)
	substring(a.content,141,210)
	
	SUM(CASE WHEN LEN(a.content) < 70 THEN 1            计算短信条数
	    ELSE CEILING(( LEN(a.content) / 67.0 ))
	    END)

**having**

	在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。   


**导入数据库**

	USE database;
	SOURCE table.sql;

**相关支持**

[W3school](http://www.w3school.com.cn/sql/index.asp)



