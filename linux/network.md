## 网络服务

### NFS
#### server
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

### Samba
#### server
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

### DHCP
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

### FTP
```bash
yum install -y vsftpd
systemctl start vsftpd
systemctl enable vsftpd
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
mount -o loop -t iso9660 xxx.iso /var/ftp/pub
```
