## 搭建shadowsocks

**安装anaconda**
```
wget https://repo.anaconda.com/archive/Anaconda3-5.3.1-Linux-x86_64.sh | sh
export PATH=$PATH:/home/felix/anaconda3/bin
```
**shadowsocks**
```
pip install shadowsocks
```
**设置配置文件**
```bash
# vim /etc/shadowsocks.json
	{
	"server":"0.0.0.0",
	"local_address":"127.0.0.1",
	"local_port":1080,
	"port_password":{
	"7788":"password0",
	"7789":"password1",
	"7790":"password2"
	},
	"timeout":300,
	"method":"rc4-md5",
	"fast_open": false
	}
```
**启动服务以这个文件配置，后台运行**
```
 ssserver -c /etc/shadowsocks.json &
```
**连接代理**
```
windows 连接，下载 shadowsock 软件
根据以上的配置文件，输入ip,端口，密码
```
**设置守护进程**
```bash
#!/bin/bash
# 脚本名字shadowsock.sh
# 运行脚本 nohup bash shadowsock.sh >> nohup.out 2>&1 &
daemon()
{
    while true
    do
        server=`lsof -i:8388` 
        date=`date "+%Y-%m-%d %H:%M:%S"`
        if [ ! "$server" ]
        then
            echo "$date, shadowsock is stoped!"
            nohup ssserver -c /etc/shadowsocks.json >> /dev/null 2>&1 &
	    echo "$date, webserver is starting..."
            sleep 10  
        else
            echo "$date, webserver is running..."
        fi
        sleep 10
    done
}
daemon
```
**linux客户端全局代理**
```
vim /etc/profile

#有用户/密码情况
export http_proxy=http://username:password@yourproxy:8080/
export ftp_proxy=http://username:password@yourproxy:8080/
#无账号
export http_proxy=http://yourproxy:8080/
export ftp_proxy=http://yourproxy:8080/

#刷新
source /etc/profile
```

**http服务器**
```
yum install privoxy  
 
修改文件/etc/privoxy/config

listen-address  :8118  

enable-remote-toggle  1  

forward-socks5 / 127.0.0.1:8388

启动Privoxy即可开启http代理。

service privoxy start

客户端访问

ip:8188
```
