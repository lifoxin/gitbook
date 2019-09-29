**安装redis服务**
	
	wget xxx
	tar xzf xxx.tar.gz
	mv redis /usr/local/redis
	make && make install
	mkdir /etc/redis
	vim /etc/redis/redis.conf
		daemonize yes (no -> yes)
	cp redis.conf /etc/redis
	redis-server /etc/redis/redis.conf

**开机启动**

`echo "/usr/local/bin/redis-server /etc/redis/redis.conf &" >> /etc/rc.local`

**连接redis**
	
	redis-cli -h host -p port -a password
	redis-cli -h 127.0.0.1 -p 6379 -a "mypass"

**字符串**

	SET name "runoob"
	GET name

**哈希**

	HMSET myhash field1 "Hello" field2 "World"
	HGET myhash field1
	HGET myhash field2

**列表**

	lpush runoob redis
	lrange runoob 0 10

**集合**
	
	sadd runoob redis
	smembers runoob

**有序集合**
	
	zadd key score member 
	zadd runoob 0 redis
	ZRANGEBYSCORE runoob 0 1000

**创建字符串** 
	
	set name 'John Smith' 
	set age 18 nx           <-- 不存在才创建 
	set age 18 xx           <-- 存在才设置 
	set age 18 ex 100       <-- 设置过期时间，以秒为单位 
	set age 18 px 100000    <-- 设置过期时间，以毫秒为单位 

**redis集群**

1 安装

    wget http://download.redis.io/releases/redis-5.0.5.tar.gz
    tar xzf redis-5.0.5.tar.gz
    cd redis-5.0.5
    make

2 创建别名

    ln -s /root/redis-5.0.5/src/redis-cli /usr/local/bin/redis-cli
    ln -s /root/redis-5.0.5/src/redis-server /usr/local/bin/redis-server

3 准备工作

    3.1 创建cluster目录，分别再创建7000-7008目录
    3.2 分别修改 redis.conf 配置
        端口           port 700?
		监听主机       bind 0.0.0.0
        保护		   protected-mode no
        后台运行       daemonize yes
        pid文件路径    pidfile  当前路径/redis.pid
        持久化路径     dir  当前路径/
        内存优化策略   maxmemory-policy  volatile-lru
        关闭AOF模式    appendonly no
        开启集群       cluster-enabled  yes
        集群中状态信息 cluster-config-file  当前路径/nodes.conf
        修改超时时间   cluster-node-timeout 15000
    3.3 把redis.conf文件放在7000-7008目录中
    
4 启动关闭

    #!/bin/sh
    redis-server 7000/redis.conf 
    redis-server 7001/redis.conf 
    redis-server 7002/redis.conf 
    redis-server 7003/redis.conf 
    redis-server 7004/redis.conf 
    redis-server 7005/redis.conf 
    redis-server 7006/redis.conf 
    redis-server 7007/redis.conf 
    redis-server 7008/redis.conf 
 	
    #!/bin/sh
    redis-cli -p 7000 shutdown 
    redis-cli -p 7001 shutdown 
    redis-cli -p 7002 shutdown 
    redis-cli -p 7003 shutdown 
    redis-cli -p 7004 shutdown 
    redis-cli -p 7005 shutdown
    redis-cli -p 7006 shutdown 
    redis-cli -p 7007 shutdown 
    redis-cli -p 7008 shutdown

5 创建集群

    最小的集群单位为3个主节点,搭建策略一主两从
    主机三台,端口:7000/7001/7002
    从机六台,端口:7003/7004/7005/7006/7007/7008
`redis-cli --cluster create --cluster-replicas 2 ip:7000 ip:7001 ip:7002 ip:7003 ip:7004 ip:7005 ip:7006 ip:7007 ip:7008`
>运行后输入"yes"

6  redis主从复制,2.2.2.2是主redis服务器
  
`redis-server redis.conf --port 6379 --slaveof 2.2.2.2 6379`

7 测试
	
	redis-cli -p 7000 -c
    set msg0 "hello world"
    set msg1 "hello world"
    set msg2 "hello world"
    set msg3 "hello world"

