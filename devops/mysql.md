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

**MySQL集群解决方案(主从复制、PXC集群、MyCat、HAProxy)**

[视频讲解参考](https://www.bilibili.com/video/av53652808)

1. 由两个mycat中间件 ，选择主从读写分离，做两个主从复制（docker集群）来数据分片， 14--15集（mycat开源）
2. haproxy 也使用docker,安装和配置，代理两个中间件。17集（haproxy开源）
3. 3个pxc做mysql，可读可写，docker模拟
4. 综合练习（2个pxc集群和分片，1个主从复制，2个mycat中间件，1个haproxy）都用docker模拟，（23集开始抄源码）

1 **mysql主从复制**

mysql 中的主从复制结构利用主服务器上的二进制日志来传输数据。

主服务器上启用了二进制日志后，主服务器上对数据库的改变会被记录在二进制日志中，从服务器把主服务器的二进制日志拿过来，应用到自己的数据库上，就达到了把主服务器的数据复制过来的目的。

**一主一从复制的配置方法**
```
假设主服务器IP 为10.1.1.1
    从服务器IP 为10.1.1.2

1. 准备工作
1.1 在主服务器和从服务器上互相绑定对方的主机名
1.2 同步两台服务器的时间
1.3 确保防火墙没有阻挡双方之间的通信，可以暂时关闭双方的防火墙
1.4 关闭SELinux
1.5 同步两台服务器之间的数据，如果是未上线的服务器，可以省略此步

2. 主服务器的配置
2.1 在主服务器上启用二进制日志，并指定一个大于零的server-id，参考配置如下
    [mysqld]
    log-bin=mysql-bin
    server-id=1
	log-slave-updates=on
2.2 重启主服务器
2.3 创建一个专用的用户，给从服务器使用，并做最小授权。
    create user repl@10.1.1.2 identified by 'abc';
    grant replication slave on *.* to repl@10.1.1.2;
2.4 记录下当前的二进制日志的文件名和位置，后面需要在从服务器上使用
    show master status;
    假设这里得到的文件名是binlog.000004, 位置是421


3. 从服务器的配置
3.1 测试到主服务器的连接，确保能成功登录
    mysql -h 10.1.1.1 -urepl -pabc
3.2 在从服务器上，指定一个大于零的server-id，此值不能与主服务器相同，参考配置如下
    [mysqld]
    server-id=2
	log-slave-updates=on
3.3 重启从服务器
3.4 root登录mysql，查看从服务器状态，如果已经开启，就把它关闭
    show slave status\G
    stop slave;
3.5 配置连接主服务器所需的信息
mysql> change master to
    -> master_host='192.168.68.130',
    -> master_port=3306,
    -> master_user='repl',
    -> master_password='abc',
    -> master_log_file='binlog.000004',
    -> master_log_pos=421;
3.6 启动从服务器线程，并查看状态
    start slave;
    show slave status\G
```

**双主复制的配置方法**

严格来说，mysql 只有一种复制结构，就是主从结构，其它的复制结构都是不同形式的主从结构而已。
双主复制的配置方法实质上是做两个主从复制。

假设主A 服务器IP 为10.1.1.1
    主B 服务器IP 为10.1.1.2

1. 以A 为主，B 为从做一个主从复制
2. 以B 为主，A 为从做另外一个主从复制

注意事项
双主复制结构中，如果两个主同时修改相同的数据，就有可能造成数据不通步现象，所以做双主复制的话，必须避免两边写相同的数据。参考思路：可以在主A 中写数据库db1，在主B 中写数据库db2。

**二进制日志**

服务器把对数句库的改变，比如插入，删除，更新，创建用户等操作，以二进制的形式存储到日志文件中，这种日志文件就是二进制日志。

**对二进制日志的查询**

mysqlbinlog <binlog file>

用 mysqlbinlog 解析出来的数据可以直接用工具mysql应用于数据库上。二进制日志常用于数据库之间的复制和备份恢复。
不同版本的服务器，binlog 的格式可能存在兼容问题，所以，在涉及binlog 的应用中，应考虑服务器版本的兼容性。

使用二进制日志恢复误删除的数据
```
1. 服务器启用二进制日志
   log-bin=logbin
2. 在服务器上创建一个数据库，创建一个表，插入一些数据，然后把表删除
3. 新创建一个数据库服务器
4. 用和服务器版本一致的mysqlbinlog 工具解析服务器的二进制日志，找到灾难发生的位置
   mysqlbinlog /tmp/binlog.000003 --stop-pos 3496 | less
5. 把所需的二进制日志解析出来，传给mysql 客户端，即可达到恢复数据的目的
   mysqlbinlog /tmp/binlog.000003 --stop-pos 3496
   mysqlbinlog /tmp/binlog.000003 --stop-pos 3496 | mysql -h 127.0.0.1 -P3311 -uroot
```
2 **配置 mycat 中间件读写分离/数据分片**

1. 官网下载 Mycat-server-1.6.7.3-release-20190828135747-linux.tar.gz
2. 新建一个目录mycat，解压到这个目录改名为 mycat01
3. 安装jdk，设置JAVA_HOME

vim conf/server.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mycat:server SYSTEM "server.dtd">
<mycat:server xmlns:mycat="http://io.mycat/">
        <system>
                <property name="nonePasswordLogin">0</property>
                <property name="useHandshakeV10">1</property>
                <property name="useSqlStat">0</property>
                <property name="useGlobleTableCheck">0</property>
                <property name="sequnceHandlerType">2</property>
                <property name="subqueryRelationshipCheck">false</property>
                <property name="processorBufferPoolType">0</property>
                <property name="handleDistributedTransactions">0</property>
                <property name="useOffHeapForMerge">0</property>
                <property name="memoryPageSize">64k</property>
                <property name="spillsFileBufferSize">1k</property>
                <property name="useStreamOutput">0</property>
                <property name="systemReserveMemorySize">384m</property>
                <property name="useZKSwitch">false</property>
        </system>
        <user name="felix" defaultAccount="true">
                <property name="password">Ab123456.</property>
                <property name="schemas">slaveDb</property>
        </user>
 <!--schemas 是数据库名称,mysql授权
 grant all privileges on *.* to felix@"*" identified by "Ab123456.";flush privileges;-->
</mycat:server>
```
vim conf/schema.xml
```
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">
        <schema name="slaveDb" checkSQLschema="true" sqlMaxLimit="100">
                <table name="test" dataNode="dn1" rule="mod-long" />
        </schema>
        
        <!--dataNode分片关系，如果有两个cluster就可以数据分片-->
        <dataNode name="dn1" dataHost="cluster1" database="slaveDb" />
        
        <!--dataHost配置信息连接，可以有多个主从复制的信息实现数据分片-->
        <dataHost name="cluster1" maxCon="1000" minCon="10" balance="3"
           writeType="1" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
           <!--balance=3 是读写分离类型-->
          <heartbeat>select user()</heartbeat>
          <writeHost host="w1" url="192.168.68.130:3306" user="root" password="123456">
           	<readHost host="w1R1" url="192.168.68.128:3306" user="root" password="123456" />
         </writeHost>
         
        </dataHost>
</mycat:schema>
```
vim conf/rule.xml
```
<property name="count">1</property>
```
测试启动
```
cd /bin
./mycat console
./startup_nowrap.sh
```
客户端连接
```
192.168.68.130
felix
Ab123456.
8066
mysql -ufelix -pAb123456. -P8066 -h 192.168.68.130
```

3 **配置haproxy均衡负载**

3.1 搭建haproxy
```
1. yum install haproxy
2. vim /etc/haproxy/haproxy.cfg

listen admin_stats
    bind 0.0.0.0:4001
    mode  http
    stats uri   /dbs
    stats realm Global\ statistics
    stats auth  admin:admin123

listen  proxy-mysql
    bind 0.0.0.0:4002
    mode tcp
    balance roundrobin
    option  tcplog
    server  mycat_1  192.168.68.130:8066  check  port  8066  maxconn  2000
    server  mycat_2  192.168.68.130:8067  check  port  8067  maxconn  2000
```
3.2 搭建多节点 mycat
```
cp mycat mycat2 -R

vim mycat/conf/wrapper.conf
#设置jmx端口
warpper.java.additional.7=-Dcom.sun.management.jmxremote.port=1985

vim mycat/conf/server.xml
#设置服务端口以及管理端口
<property name="serverPort">8067</property>
<property name="managerPort">9067</property>
```
3.3 连接测试
```
3.1 启动两个mycat
    ./startup_nowrap.sh && tail -f ../logs/mycat.log
3.2 启动haproxy
    systemctl start haproxy
3.3 连接http://192.168.68.130:4001/dbs,查看状态
3.4 连接haproxy写入数据
    mysql -ufelix -pAb123456. -P4002 -h 192.168.68.130
```

### 综合myql集群应用

>两个pxc集群，一个主从复制集群，两个mycat中间件连接这三个集群,
一个haproxy均衡负载连接两个mycat中间件。

```
                                  |-----pxc1
                            |-----|-----pxc2
             |----mycat-----|     
             |              |     |-----pxc3
   haproxy---|              |-----|-----pxc4
             |              |
             |----mycat-----|-----|-----master 
                                  |-----slave
```
**搭建pxc**

1 拉取镜像

    docker pull percona/percona-xtradb-cluster:5.7
    docker tag percona/percona-xtradb-cluster:5.7 pxc

2 创建数据卷（存储路径：/var/lib/docker/volumes）

    docker volume create lvm1
    docker volume create lvm2
    docker volume create lvm3
    docker volume create lvm4

3 创建网络

`docker network create --subnet=172.30.0.0/24 pxc-network`

4 创建容器

    pxc1
    
    docker create -p 13306:3306 -v lvm1:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root \
    -e CLUSTER_NAME=pxc --name=pxc_node1 --net=pxc-network --ip=172.30.0.2 pxc
    
    pxc2（增加了CLUSTER_JOIN参数）
    
    docker create -p 13307:3306 -v lvm2:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root \
    -e CLUSTER_NAME=pxc --name=pxc_node2 -e CLUSTER_JOIN=pxc_node1 --net=pxc-network  \
    --ip=172.30.0.3 pxc
    
    pxc3
    
    docker create -p 13308:3306 -v lvm3:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root \
    -e CLUSTER_NAME=pxc --name=pxc_node3 --net=pxc-network --ip=172.30.0.4 pxc
    
    pxc4（增加了CLUSTER_JOIN参数）
    
    docker create -p 13309:3306 -v lvm4:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root \
    -e CLUSTER_NAME=pxc --name=pxc_node1 -e CLUSTER_JOIN=pxc_node3 --net=pxc-network \
    --ip=172.30.0.5 pxc

5 启动 (集群的第一个节点先启动再启动其他节点)

    docker start pxc_node1 && docker logs -f pxc_node1
    docker start pxc_node2 && docker logs -f pxc_node2
    docker start pxc_node3 && docker logs -f pxc_node3
    docker start pxc_node4 && docker logs -f pxc_node4

6 查看集群节点,结果是2为正常

`show status like 'wsrep_cluster%'`

**搭建主从复制**

1 拉取镜像

    docker pull percona:5.7.23
    docker tag percona:5.7.23 mysql

2 创建master目录

    mkdir /data/mysql/master/{conf,data} -p
    cd /data/mysql/master
    chmod 777 * -R

3 创建配置文件

    cd /data/mysql/master/conf
    vim my.cnf
     [mysqld]
     log-bin=mysql-bin
     server-id=1

4 创建容器

`docker create --nmae mysql-master  -v /data/mysql/master/data:/var/lib/mysql \
-v /data/mysql/master/conf:/etc/my.cnf.d -p 23306:3306 -e MYSQL_ROOT_PASSWORD=root mysql`

5 启动

`docker start mysql-master && docker logs -f mysql-master`

6 创建同步账号以及授权,看二进制信息

    docker exec -it mysql-master /bin/bash
    create user 'felix'@'%' identified by 'Ab123456.';
    grant replication slave on *.* to 'felix'@'%';
    flush privileges;
    show master status;

7 创建master目录

    mkdir /data/mysql/slave/{conf,data} -p
    cd /data/mysql/slave
    chmod 777 * -R

8 创建配置文件

    cd /data/mysql/slave/conf
    vim my.cnf
    [mysqld]
    log-bin=mysql-bin
    server-id=2

9 创建容器

`docker create --nmae mysql-slave  -v /data/mysql/slave/data:/var/lib/mysql \
-v /data/mysql/slave/conf:/etc/my.cnf.d -p 23307:3306 -e MYSQL_ROOT_PASSWORD=root mysql`

10 启动,设置master相关信息

    docker start mysql-slave && docker logs -f mysql-slave
    docker exec -it mysql-slave /bin/bash
    CHANGE MASTER TO
     master_host='192.168.68.130',
     master_user='felix',
     master_password='Ab123456.',
     master_port=23306,
     master_log_file='mysql-bin.000002',
     master_log_pos=648;

11 启动同步并查看slave状态

    start slave
    show slave status;

**搭建mycat**

1 第一个mycat节点

    mkdir /data/{mycat1,mycat2}
    cd /data/mycat1

2 修改配置server.xml，设置自己的管理用户和端口18067
```
同上，配置用户名，密码，数据库,端口
<property name="serverPort">18067</property>
<property name="managerPort">19067</property>
```

3 修改配置schema.xml，连接集群接口
```
schema.xml:
配置数据表
<schema name="cluster" checkSQLschema='false' sqlMaxLimit="100">
  <table name="cluster_pxc" dataNode="dn1,dn2" rule="mod-long"/>
  <table name="cluster_slave" dataNode="dn3"/>
</schma>

配置分片关系：
<dataNode name="dn1" dataHost="cluster1" database="cluster"/>
<dataNode name="dn2" dataHost="cluster2" database="cluster"/>
<dataNode name="dn3" dataHost="cluster3" database="cluster"/>

配置连接关系 #balance=2表示读写一致,随机分发
<dataHost name="cluster1" maxCon="1000" minCon="10" balance="2"  
         writeType="1" dbType="mysql" dbDriver="native" swithType="1"
         slaveThreshold="100">
    <hearbeat>select user() </hearbeat>
    <writeHost host="w1" url="192.168.68.130:13306" user="root" passord="root">
       <readHost host"W1R1" url="192.168.68.130:13307" user="root" password="root"/>
    </writeHost>
</dataHost> 

<dataHost name="cluster2" maxCon="1000" minCon="10" balance="2"  
         writeType="1" dbType="mysql" dbDriver="native" swithType="1"
         slaveThreshold="100">
    <hearbeat>select user() </hearbeat>
    <writeHost host="w2" url="192.168.68.130:13308" user="root" passord="root">
       <readHost host"W2R1" url="192.168.68.130:13309" user="root" password="root"/>
    </writeHost>
</dataHost> 
 
<dataHost name="cluster3" maxCon="1000" minCon="10" balance="3"  
         writeType="1" dbType="mysql" dbDriver="native" swithType="1"
         slaveThreshold="100">
    <hearbeat>select user() </hearbeat>
    <writeHost host="w3" url="192.168.68.130:23306" user="root" passord="root">
       <readHost host"W3R1" url="192.168.68.130:23307" user="root" password="root"/>
    </writeHost>
</dataHost> 
```
4 修改rule.xml信息，有两个大集群dataNode

`<property name="count">2</property>`

5 wrapper.conf 设置jmx端口

`wrapper.java.additional.7=-Dcom.sun.management.jmxremote.port=11985`

6 启动

`./startup_nowrap.sh && tail -f ../logs/mycat.log`

7 复制相同的 mycat 做负载

    cp /data/mycat1 /data/mycat2 -R
    
    vim wrapper.conf
    #设置jmx端口
    wrapper.java.additional.7=-Dcom.sun.management.jmxremote.port=11986
    
    vim server.xml
    #设置服务端口和管理端口
    <property name="serverPort">18068</property>
    <property name="managePort">19068</property>
    
    ./start_nowrap.sh && tail -f ../logs/mycat.log

**搭建haproxy**

1 vim /etc/haproxy/haproxy.cfg

    listen proxy-mysql
        bind 0.0.0.:4002
        mode tcp 
        balance roundrobin
        option tcplog
        server  mycat1  192.168.68.130:18067  chech port 18067  maxconn  2000
        server  mycat2  192.168.68.130:18068  chech port 18068  maxconn  2000
2 启动

`docer start haproxy && docker logs -f haproxy`

**存储过程**

>触发器是某件事触发后自动调用

>存储过程是主动调用的，且功能比触发器更加强大。
只有首次执行需经过编译和优化步骤，后续被调用可以直接执行，速度快

1 **创建存储过程**
```
CREATE PROCEDURE 存储名称(参数列表)
BEGIN
    一组合法的 SQL 语句(存储过程体)
END $

参数列表：
参数模式  参数名   参数类型
举例:
IN arg1 VARCHAR(20)

参数模式：
IN        传入参数   
OUT       返回值     
INOUT     传入参数和返回值

如果存储过程体只有一句话，BEGIN END可以省略
存储过程体每条SQL语句结尾要求加分号

存储过程结尾可以使用 DELIMITER 重新设置
语法：
DELIMITER 结束标记

DELIMITER $
```
2 **调用存储过程**

`CALL 存储过程(实参列表)$`

3 **查看存储过程信息**

`show create procedure myp2;`

4 **删除存储过程**

语法：drop procedure 存储过程名称

`drop procedure myp3;`

5 **使用实例**
```
#创建存储过程实现，用户是否登录成功
create procedure myp1(in username varchar(20), in password varchar(20))
begin
    declare result int default 0; #声明并初始化
    
    select count(*) into rsult #赋值
    from admin
    where admin.username = username
    and admin.password = password;

    select if(result>0,'成功','失败') #使用
end $

call myp1('felix','1234')

#传入女神名，返回男神名和魅力值
create procedure myp2(in beautyName varchar(20), out boyName varchar(20), out usercp int)
begin
    select boys.boyname, boys.usercp into boyname,usercp
    from boys
    right join
    beauty b on b.boyfriend_id = boys.id
    where b.name=beautyName;
end $

call myp2('小昭',@name,@cp)$
select @name,@cp$

#传入a和b,然后翻倍返回
create procedure myp3(inout a int, inout b int)
begin
    set a=a*2
    set b=b*2
end $

set @m=10$
set @n=20$
call myp3(@m,@n)$
select @m,@n$

#传入两个女神生日，返回大小
create procedure myp4(in birth1 datetime, in birth2 datetime, out result int)
begin
    select datediff(birth1,birth2) into result;
end $

call myp4('1993-03-24',now(),@result)
select @result

#传入日期，返回格式化xx年xx月xx日
create procedure myp5(in mydate datetime,out strDate varchar(50))
begin
    select date_format(mydate,'%y年%月%日') into strDate;
end $

call myp5(now(),@str)$
select @str$

#传入女神，返回格式化女神 and 男神
drop procedure myp6 $
create procedure myp6(in beautyName varchar(20),out str varchar(50))
begin
    select concat(beautyName,'and',ifnull(boyName,'null')) into str
    from boys bo
    right join beauty b on b.boyfriend_id = bo.id
    where b.name=beautyName
end $

call myp6('小昭'，@str)$
select @str$

#传入起始索引和条目数，查询记录
create procedure myp7(in startIndex int,in size int)
begin
    select * from beauty limit startIndex,size;
end

call myp7(3,5)$
```

**相关支持**

[W3school](http://www.w3school.com.cn/sql/index.asp)

