### nginx
相关支持

- [中文文档](http://www.nginx.cn/doc)
- [开源官网](http://nginx.org)

**安装**

1. 安装依赖库
2. yum install -y gcc openssl openssl-devel pcre pcre-devel zlib zlib-devel
3. ./configure --prefix=/user/local/nginx 安装路径
4. make && make install 

**进程**

ps -ef |grep nginx 

1. master进程读配置文件并维护worker进程
2. worker进程对请求进行实际处理

**负载均衡**

1.http模块配置需要代理的服务

    负载算法
    1. ip_hash算法，ip绑定服务器
    2. 权重算法 weight
    3. 最小连接算法 least_conn
```bash
upstream jeffhardy.cn {         #指定域名
	#ip_hash; 
	#least_conn;
	server 1.1.1.1:80 weight=3; #权重算法
	server 2.2.2.2:80 weight=1;
	server 3.3.3.3:80 backup;   #其他挂了才启用
	server 4.4.4.4:80 down;     #表示当前sever不参与负载
}
```
2. server模块配置转发代理
```bash
location / {
	proxy_pass http://jeffhardy.cn  #配置转发代理域名，上同域名
}
```

**动态分离**

```bash
location ~ .*(css|js|img|images){
	root /opt/static;
	#proxy_pass http://static.jeffhardy.cn  如果需要静态资源转发
}
```
**虚拟主机**

1. 一个主机可以多配几个网站，server标签就是一个虚拟主机
2. 在server下，添加文件配置多个虚拟主机
```bash
include /usr/local/nginx/vhost/nginx.conf；
```
3. 虚拟主机区分：

    1. 端口区分 listen
    2. 域名区分 server_name

**配置域名自动跳转到Https访问**

```bash
# vim /usr/local/nginx/conf/nginx.conf
server {
listen 80;
server_name yuor_domain; 
rewrite ^(.*)$ https://$server_name$1 permanent;
....
....
}
```

**配置 ssl**

```bash
# vim /usr/local/nginx/conf/nginx.conf
server {
	ssl_certificate_key /usr/local/nginx/cert/jeffhardy.cn.key;
	ssl_certificate /usr/local/nginx/cert/jeffhardy.cn_bundle.crt;
	....
}
```

**配置 http 文件下载**

```bash
# vim /usr/local/nginx/conf/nginx.conf
location / {
	autoindex on;
	autoindex_exact_size on;
	autoindex_localtime on;
	charset utf-8;
}
```

**查看 nginx 进程**

```bash
ab -c 100 -n 1000 http://localhost//          多进程访问这个网页
ps axo pid,cmd,psr | grep nginx
watch -n0.5 'ps axo pid,cmd,psr | grep nginx'  动态查看 nginx 内核
```

**https证书**

```
Letsencrypt 申请免费的SSL证书
https://certbot.eff.org/lets-encrypt/centosrhel7-nginx
https://www.qcloud.com/product/ssl

HTTPS相关检测工具
https://myssl.com/ssl.html

HTTPS SSL评级网站，查询到下面这个域名支持v4和v6
https://www.ssllabs.com/ssltest/analyze.html?d=secret.imbusy.me
```
