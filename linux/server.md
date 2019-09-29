### 搭建邮箱服务器 
alternatives --config mta 
然后直接回车即可。

检查一下是不是已经设置成功了。

alternatives --display mta
第一行可以看到mta的状态。 例如：mat - status is manual.就是ok了。

#### 安装 postfix 提供发件服务
yum install postfix

vim /etc/postfix/main.cf 
```
myhostname = server1 //主机名
mydomain = lifoxin.com	//设置本地网络的邮件域	
myorigin = $mydomain //要外发邮件时发件人的邮件域名
inet_interfaces = all		//设置postfix监听的网络端口
mydestination = $myhostname,localhost.$mydomain,localhost,$mydomain//设置可接收邮件的主机名或域名
mynetworks = 192.168.68.0/24, 127.0.0.0/8		//收发客户端的地址
relay_domains = $mydestination	//设置可转发来自哪些域名或主机名的邮件
home_mailbox = Maildir/ 	    //邮件存储的位置
```

**启动服务**

systemctl start postfix

systemctl enable postfix

#### 安装 dovecot 提供收件

yum install dovecot

vim /etc/dovecot/dovecot.conf
```
listen = *  //不使用ipv6
```
vim /etc/dovecot/conf.d/10-mail.conf
```
mail_location = maildir:~/Maildir //指定邮件存储格式和位置
```
vim /etc/dovecot/conf.d/10-ssl.conf 
```
ssl = no   //不使用ssl
```
**启动服务**

systemctl start dovecot

systemctl enable dovecot

#### 测试验证
1. windows 使用foxmai 客户端
2. 登录账户是linux用户，账号是（用户+域名）即可
3. 也可以在linux环境使用telnet命令发送邮件

[参考邮件服务器](https://blog.csdn.net/wq962464/article/details/84864750)

### 搭建DNS 

**安装环境**
```
yum install -y bind bind-utils

cp /usr/share/doc/bind-9.11.4/sample/etc/named.conf /etc/
cp /usr/share/doc/bind-9.11.4/sample/var/named/named.ca /var/named/
cp /usr/share/doc/bind-9.11.4/sample/var/named/named.localhost /var/named/lifoxin.com.zone

chown root.named /etc/named.conf
chown root.named /var/named/named.ca
chown root.named /var/named/lifoxin.com.zone
```
**修改配置文件**

vim /etc/named.conf
```
options
{
        directory               "/var/named";           // "Working" directory
        dump-file               "data/cache_dump.db";
        statistics-file         "data/named_stats.txt";
        memstatistics-file      "data/named_mem_stats.txt";
        recursing-file          "data/named.recursing";
        secroots-file           "data/named.secroots";

        listen-on port 53       { any; };

        allow-query             { any; };
        allow-query-cache       { any; };
        recursion yes;
        dnssec-enable no;
        dnssec-validation no;

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";

        managed-keys-directory "/var/named/dynamic";
};

zone "." IN {
        type hint;
        file "/var/named/named.ca";
};

zone "lifoxin.com" IN {
        type master;
        file "lifoxin.com.zone";
};

zone "68.168.192.in-addr.arpa" IN {
        type master;
        file "192.168.68.zone";
};

```
vim /var/named/lifoxin.com.zone
```
$TTL 1D
@       IN SOA  lifoxin.com. mail.lilifoxin.com. (
                0       ; serial
                1D      ; refresh
                1H      ; retry
                1W      ; expire
                3H )    ; minimum

        NS      dns.lifoxin.com.
ftp     IN      A       192.168.68.130
www     IN      A       192.168.68.130
mail    IN      A       192.168.68.130
dns     IN      A       192.168.68.130
xx      IN      A       192.168.68.100
yy      IN      A       192.168.68.200
```
vim /var/named/192.168.68.zone
```
$TTL    1D
@       IN      SOA     lifoxin.com. mail.lifoxin.com. (
                10      ; serial
                1D      ; refresh
                1H      ; retry
                1W      ; expire
                3H )    ; minimum 

        NS        dns.lifoxin.com.
130     IN   PTR  ftp.lifoxin.com.
130     IN   PTR  www.lifoxin.com.
130     IN   PTR  mail.lifoxin.com.
130     IN   PTR  dns.lifoxin.com.
100     IN   PTR   xx.lifoxin.com.
200     IN   PTR   yy.lifoxin.com.
```
**最后**
```
systemctl stop firewalld.service
systemctl start named
systemctl enable named
```
**客户端测试**
```
echo "nameserver 192.168.68.130" >> /etc/resolv.conf
nslookup xx.lifoxin.com
```
**本机测试**
```
echo "nameserver 127.0.0.1" >> /etc/resolv.conf
nslookup yy.lifoxin.com
nslookup dns.lifoxin.com
```
[参考正向解析](https://blog.51cto.com/13701082/2340793)


### 搭建NFS
#### server1
```bash
yum install -y nfs-utils rpcbind
useradd -u 1003 felix
mkdir /var/{web,cloud}
chmod a+w /var/web
cat /etc/exports
	/var/web/ client1(rw,asynv,no_root_squash)
	/var/cloud  *(ro,sync)
```
#### client1
```bash
showmount -e server
mkdir /var/web
useradd -u 1003 felix
mount server:/var/web/ /var/web/  #手动挂载
echo "server:/var/web /var/web/ nfs defaults 0 0"   #开机自动挂载
chmod a+w /var/web
su felix
cd /var/web;touch text.txt  #显示有权限写入文件
```
#### client2
```bash
mkdir /var/cloud
useradd -u 1003 felix
mount server:/var/cloud /var/cloud
cd /var/cloud ; touch test  #显示拒绝，默认root权限会自动映射为 nfsnobody 账号，普通账号权限保留
```

### 搭建Samba
#### server1
```bash
yum install -y samba
mkdir /common ; chmod 777 /commom
setenfore 0
sed -i "/SELINUX=/c SELINUX=disable" /etc/sysconfig/selinux
vim /etc/samba/smb.conf
	[share]
    	comment = share        // 共享的文件夹
    	path = /database     // 共享文件的目录
    	public = yes           // 是否公共属性
    	writable = yes
    	browseable=yes
    	available=yes
    	guest ok=yes    
 	security = share   // 变成共享的
```
#### client
```bash
 apt-get install cifs-utils
 mount -t cifs //192.168.0.103/Public /mnt/samba/ -o guest
```

### 搭建DHCP
```bash
yum install -y dhcp
vim /etc/dhcp/dhcpd.conf
	subnet 10.0.0.0 netmask 255.255.255.0{
	range 10.0.0.1 10.0.0.254
	option domain-name-servers 11.114.114.114;
	option routers 10.0.0.1;
	default-lease-time 600;
	max-lease-time 7200;
	next-server 10.0.0.1;
	filename "pxelinux.0"
	}
systemctl start dhcpd
systemctl enable dhcpd
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
```

### 搭建FTP
```bash
yum install -y vsftpd
systemctl start vsftpd
systemctl enable vsftpd
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
mount -o loop -t iso9660 xxx.iso /var/ftp/pub
```
