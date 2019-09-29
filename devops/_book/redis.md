### 安装redis服务
- wget xxx
- tar xzf xxx.tar.gz
- mv redis /usr/local/redis
- make && make install
- mkdir /etc/redis
- vim /etc/redis/redis.conf
-  daemonize yes (no -> yes)
- cp redis.conf /etc/redis
- redis-server /etc/redis/redis.conf
### 开机启动
echo "/usr/local/bin/redis-server /etc/redis/redis.conf &" >> /etc/rc.local
### 连接redis
* redis-cli -h host -p port -a password
* redis-cli -h 127.0.0.1 -p 6379 -a "mypass"
### 字符串
* SET name "runoob"
* GET name
### 哈希
* HMSET myhash field1 "Hello" field2 "World"
* HGET myhash field1
* HGET myhash field2
### 列表
* lpush runoob redis
* lrange runoob 0 10
### 集合
* sadd runoob redis
* smembers runoob
### 有序集合
* zadd key score member 
* zadd runoob 0 redis
* ZRANGEBYSCORE runoob 0 1000
### 创建字符串 
- set name 'John Smith' 
- set age 18 nx           <-- 不存在才创建 
- set age 18 xx           <-- 存在才设置 
- set age 18 ex 100       <-- 设置过期时间，以秒为单位 
- set age 18 px 100000    <-- 设置过期时间，以毫秒为单位 
### 查看
- keys *                  <-- 列出key
- get age                 <-- 查看值
- ttl age                 <-- 查看键的过期时间
- pttl age                <-- 查看键的过期时间（毫秒）
### 修改/删除键的过期时间
- expire age 100              <-- 100秒后过期
- pexpire age 100123          <-- 100123毫秒后过期
- expireat age 1523257800     <-- 指定过期的时间点(unix时间戳)


