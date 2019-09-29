* 防火墙 iptable

```
service iptables start  
service iptables stop  
service iptables save  
service iptables reload  
service iptables status  

关闭所有的80端口
开启ip段192.168.1.0/24端的80口
开启ip段211.123.16.123/24端ip段的80口
iptables -I INPUT -p tcp --dport 80 -j DROP 
iptables -I INPUT -s 192.168.1.0/24 -p tcp --dport 80 -j ACCEPT
iptables -I INPUT -s 211.123.16.123/24 -p tcp --dport 80 -j ACCEPT

编辑防火墙白名单
[root@localhost ~]# vim /etc/sysconfig/iptables
增加下面一行代码
-A INPUT -p tcp -m state -- state NEW -m tcp --dport 80 -j ACCEPT

保存退出，重启防火墙
[root@localhost ~]# service iptables restart
[root@localhost ~]# service iptables stop
```
* 主机网关

```
多网卡在主机，设置只有一个网关，其余是添加路由

vi /etc/rc.d/rc.local在最后一行添加如下内容：
    #添加默认网关 192.168.1.1
    route add default gw 192.168.1.1  
    #1.0网段走192.168.1.1网关走 eth0为要走的网卡。
    route add -net 192.168.1.0/24 gw 192.168.1.1 eth0 
    #2.0网段走2.254网关、通过eth1这个网卡走。
    route add -net 192.168.2.0/24 gw 192.168.2.254 eth1 
    #3.0网段走 2.254网关、通过eth1网卡走。
    route add -net 192.168.3.0/16 gw 192.168.2.254 eth1 
```
* 网络排除

	1. 设备，ip，网关，路由，vpn,代理，认证，域名解析dns,防火墙等

* 分区异常

```
linux里的文件被删除后，空间没有被释放是因为在Linux系统中.
通过rm或者文件管理器删除文件将会从文件系统的目录结构上解除链接(unlink)。
然而如果文件是被打开的(有一个进程正在使用)，那么进程将仍然可以读取该文件，磁盘空间也一直被占用。

1. 先df -lh查看一下磁盘使用状况
2. 找到被删除文件所在的分区,eg.opt分区
3. 查看被删除了的所有文件：lsof -n /opt |grep deleted
4. sftp-serv  8195  root  5r  REG  104,6 8214888448 786452 /opt/xxx (deleted)
5. kill 8195
6. 再运行lsof -n /opt |grep delete，应该没上面的结果了。
7. 再运行df -lh看是不是空间已经释放了？
```
