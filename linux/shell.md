## shell 

### 常见
```
$()  是执行里面的命令
${}  是调用这个变量的值
$(cd `dirname $0`;echo `pwd`)   表示当前路径
export Parent_dir=$(dirname $(cd `dirname $0`;echo `pwd`))  
    导入环境变量，表示当前路径的父路径，即上一层路径
切换用户只执行一条命令的可以用: su - user -c command
切换用户执行一个shell文件可以用:su - user -s /bin/bash shell.sh
#!/bin/sh
user="webmaster"
if [ `whoami` != "${user}" ]; then
        exec su - "${user}" -c "sh /tmp/check.sh"
		# su - ${user} -c sh /tmp/check.sh
fi
```

### 备份数据库脚本
```bash
#!/bin/bash
# 数据库认证
 user=""
 password=""
 host=""
 db_name=""
# 其它
 backup_path="/path/to/your/home/_backup/mysql"
 date=$(date +"%d-%b-%Y")
# 设置导出文件的缺省权限
 umask 177
# Dump数据库到SQL文件
 mysqldump --user=$user --password=$passwd --host=$host $db_name > $path/$db_name-$date.sql
# 删除30天之前的就备份文件
 find $backup_path/* -mtime +30 -exec rm {} \;
```

### 守护进程
```bash
#!/bin/sh
# nohup ./monitor.sh >> monitor.log 2>&1 &
function daemon()
{
    while true
    do
        server=`lsof -i:8080`  #服务器占用端口为8080，通过查看8080端口是否占用来判断服务是否启动
        date=`date "+%Y-%m-%d %H:%M:%S"`
        if [ ! "$server" ]
        then
            echo "$date, webserver is stoped!"
            nohup sh startserver.sh >> nohup.out 2>&1 &  #通过nohup命令后台运行服务
            echo "$date, webserver is starting..."
            sleep 10  #启动后等待10s
        else
            echo "$date, webserver is running..."
        fi
        sleep 10
    done
}
daemon

time=`date "+%Y-%m-%d %H:%M:%S"`
server=`ps -ef | grep -v grep |grep shadow`
usleep=10000
main(){
if [ -z "$server" ]
then
	/home/felix/anaconda3/bin/python /home/felix/anaconda3/bin/ssserver -c   \
    /etc/shadowsocks.json >> /tmp/shadowsock.log 2>&1 &
else
	echo "$time shadowsock is running" >> /tmp/normal.time
fi
}
main
wait
echo "All is ok"
exit 0
```

### 备份日志
```bash
#!/bin/bash
bakdir=mylog
date=`date +%F`
cd /var
tar zxf ${bakdir}_${date}.tar.gz ${bakdir}
sleep 1
ftp -n << -EOF
open 192.168.1.1
user aaa bbb
put mylog_*.tar.gz
bye
EOF
rm -rf mylog_*.tar.gz

crontab -e
00 05 * * * /bin/bash /root/mylogbak.sh
```

### 求参数的大小和平均值
```bash
#!/bin/bash
proc=`basename $0`
usage()
{
    printf "usage: %s data1 ,,, datan\n" "proc"
}
if [ $# -lt 3 ];then
    usage
    exit 1
fi

max=$1
min=$1
sum=0
for i in $@
do
    [ $max -lt $i ] && max=$i
    [ $min -gt $i ] && min=$i
    let sum+=i
done
echo "max=$max"
echo "min=$min"
echo "ibase=10; scale=2; $sum/$#" | bc
```

### check ip
```bash
ip=$(cat ip.list)
for ip in $ip;do
    if  ping -c 1 $ip &> /dev/null; then
    	 echo $ip >> correct.ip;
    fi
done
echo "ping over" 
```

### 查看80端口
```bash
for ip in 192.168.6.{0..255};do
  if telnet $ip 80 &> /dev/null; then
    echo $ip >> ip.list
fi
done
```

### email
```
#使用客户端配置发送邮件
#格式 python3 send_mail.py "805986238@qq.com" sub lalallalal

sudo yum install -y mailx sendmail
vim /etc/mail.rc
set from=foxin.li@inin88.com
set smtp=mail.inin88.com
set smtp-auth-user=foxin.li@inin88.com
set smtp-auth-password= 邮箱授权码
set smtp-auth=login

mail -s "Hello from mzone.cc by shell" admin@mzone.cc  空格，后面是内容
mail -s "them" admin@mzone.cc < mail.txt
echo "content | mail -s "them" admin@mzone.cc
```

### 检查远程端口脚本
```bash
#!/bin/sh
PortNum=`nmap 10.0.0.189 -p 80|grep open|wc -l`
if [ $PortNum -eq 1 ]
then
 echo "mysqld is running."
else
 echo "mysqld is stoped."
fi
```
