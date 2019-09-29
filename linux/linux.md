用户修改密码，不需要确认一次

echo "密码" | passwd --stdin 用户名

**不同vlan互通**

1. 每个VLAN都可以配置一个三层 vlanif 逻辑接口，而这些 vlanif 接口就作为对应 vlan 内部用户主机的缺省网关， 通过三层交换机内部的 ip路由功能可以实现同一交换机上不同 vlan 的三层互通，不同交换机上不同 vlan间的三层互通需要配置各VLANIF接口所在网段间的路由。(简单说就是三层交换机开启路由功能)

2. Linux开启路由转发功能（简单说就是依赖第三台主机做转发）[配置](https://www.linuxidc.com/Linux/2016-12/138661.htm)

**xargs**

xargs 又称管道命令，构造参数等,简单的说 就是把 其他命令的给它的数据 传递给它后面的命令作为参数

主要参数

    -d 为输入指定一个定制的分割符
    -i 用 {} 代替 传递的数据
    -I string 用string来代替传递的数据-n[数字] 设置每次传递几行数据
    -n 选项限制单个命令行的参数个数
    -t 显示执行详情
    -p 交互模式
    -P n 允许的最大线程数量为n
    -s[大小] 设置传递参数的最大字节数(小于131072字节)
    -x 大于 -s 设置的最大长度结束 xargs命令执行
 
 ```
 find ./ -name "*.json" | xargs grep baidu               #查找所有含有baidu关键词的json文件
 find .| xargs grep "this is a test for finding" 2>null  #在test.txt文本中找到了字符串并打印了该行
 ls |grep .php | xargs -i mv {} {}.bak                   #将当前目录下php文件,改名字
 cat file | xargs                                        #转换单行输出
 cat file | xargs -n3                                    #限制每行输出三个
 ```
**磁盘常用阵列**

多块硬盘通过不同方式组成硬盘组（逻辑磁盘），以提高性能和数据备份技术

1. RAID-0 别名条带，性能好，但不安全，损坏的数据不能恢复，推荐个人使用
2. RAID-1 别名镜像，安全，可用，全备份，存储成本高，推荐用服务器，数据库使用
3. RAID-10 别名镜像阵列条带，性能与安全兼顾，存储成本高，推荐银行、金融使用
4. RAID-3 别名专用奇偶条带，安全高，读写速度慢，视频编辑，大型数据库
5. RAID-5 别名分奇偶位条带，性能，安全，成本兼顾，推荐金融、数据库

**find 命令**

find 路径 -命令参数 [输出形式]

	例子
	find . -name "*.swp" 查找临时文件
	rm `find . -name "*.swp"`  删除临时文件

[man手册](http://linux.51yip.com/)

** 图像化 & VNC-Server **
	
	wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
	yum makecache
	yum groupinstall "X Window System" 
	yum groupinstall "GNOME Desktop"
	reboot
	#查看当前界面 multi-user.target 非图形化界面 
	systemctl get-default  
	#设置图形化界面
	systemctl set-default graphical.target 
	
	systemctl stop firewalld.service
	setenforce 0
	yum install tigervnc-server tigervnc-server-module
	cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
	
	vim /etc/systemd/system/vncserver@:1.service
	[Unit]
	Description=Remote desktop service (VNC)
	After=syslog.target network.target
	[Service]
	Type=forking
	User=root
	ExecStart=/usr/bin/vncserver :1 -geometry 1280x1024 -depth 16 -securitytypes=none \
			  -fp /usr/share/X11/fonts/misc
	ExecStop=/usr/bin/vncserver -kill :1
	[Install]
	WantedBy=multi-user.target

	systemctl enable vncserver@:1.service
	vncpasswd
	systemctl start vncserver@:1.service

**linux 后台命令**

commond &

后台执行命令

ctrl + z

可以将一个正在前台执行的命令放到后台，并且暂停

jobs

查看当前有多少在后台运行的命令

fg

将后台中的命令调至前台继续运行

bg

将一个在后台暂停的命令，变成继续执行

nohup

1. 用途：不挂断地运行命令。
1. 语法：nohup Command [ Arg … ] [　& ]

**vim 添加目录，安装插nerdtree**

1. 官网：http://www.vim.org/scripts/script.php?script_id=1658
2. GitHib：https://github.com/scrooloose/nerdtree

安装

1. unzip *.zip && mkdir ~/.vim 
2. cp nerdtree/* ~/.vim/ -rf

使用

1. :NERDTree 打开
1. vim /etc/vimrc  设置快捷键F3打开
1. map <F3> :NERDTreeMirror<CR>
1. map <F3> :NERDTreeToggle<CR> 

常用命令

1. "ctrl+w"，光标自动在左右侧窗口切换
2.  和编辑文件一样，通过h j k l移动光标定位
3. 打开关闭文件或者目录，如果是文件的话，光标出现在打开的文件中
4. go 效果同上，不过光标保持在文件目录里，类似预览文件内容的功能
5. i和s可以水平分割或纵向分割窗口打开文件，前面加g类似go的功能
6. t 在标签页中打开
7. T 在后台标签页中打开
8. p 到上层目录
9. P 到根目录
10. K 到同目录第一个节点
11. J 到同目录最后一个节点
12. m 显示文件系统菜单（添加、删除、移动操作）
13. ? 帮助
14. q 关闭

**vim 自动添加头部**
```bash
# file in /etc/vimrc and change it
autocmd BufNewFile ###.sh,###.py exec ":call SetTitle()"
func SetTitle()
	if &filetype == 'sh'
		call setline(1,"#! /bin/bash")
		call setline(2,"")
	elseif &filetype == 'python'
		call setline(1,"#! /usr/bin/env python3")
		call setline(2,"# coding=utf-8")
		call setline(3,"")
		call setline(4,"def main():")
		call setline(5,"")
		call setline(6,"    pass")
		call setline(7,"")
		call setline(8,"")
		call setline(9,"if __name__ == \"__main__\":")
		call setline(10,"")
		call setline(11,"   main()")
	endif
endfunc
```
**VT码**

```
\033[6;31m Hello World. \033[0m   变红色

```
**tmux**
```
常用到的几个组合键：
ctrl+b ?            显示快捷键帮助
ctrl+b 空格键       采用下一个内置布局，这个很有意思，在多屏时，用这个就会将多有屏幕竖着展示
ctrl+b !            把当前窗口变为新窗口
ctrl+b "           模向分隔窗口
ctrl+b %            纵向分隔窗口
ctrl+b o            跳到下一个分隔窗口。多屏之间的切换
ctrl+b 上下键      上一个及下一个分隔窗口
ctrl+b c           创建新窗口
ctrl+b n           选择下一个窗口
ctrl+b x          关闭某个窗口
```
**批量注释**
	1. ctrl + v 选择多行的第一列
	1. 按大写 i,进入插入模式
	1. 输入 # ，再 esc 两次
**vim 批量缩进**
	1. 光标先移动到需要开始选中的地方
	2. SHIFT + V 这时候已经选中当前行了
	3. 按V的手指松开，SHIFT键不放开，按上下键选中
	4. 接下来就能<，>缩进了
**vim代码乱码**
	1. 在拷贝前输入:set paste (这样的话，vim就不会启动自动缩进，而只是纯拷贝粘贴）
	2. 拷贝完成之后，输入:set nopaste (关闭paste)

**修改PS1显示红色的 felix~**
	1. export PS1="\[\e[35;1m\]felix~: \[\e[0m\]"

**rsync** 
```
rsync -av ./  /des/                               #同步当前文件到
rsync -av ./ -e "ssh -p xxx" root@hkserver:/data/ #同步远程主机
```
**ps , pgrep**
```
ps -aux  | grep -v grep | grep ping  
ps -elf       #查后台进程         
killall -1 httpd   #数字1
lsof -i:80
pgrep -f http   # 查看http相关的进程pid
pgrep -x httpd    #精确匹配httpd进程pid
pgrep -ln httpd   #主进程的pid
pgrep -lo httpd   #最先启动的进程pid
# 查看进程打开的文件
ps -ef | grep crond |grep -v grep | awk '{print $2}'  得出pid
lsof -p pid
```
**top**
```
top 动态查看负载，3分钟更新一次
1  查看各个内核负载
M  查看内存占比排序（倒序）
P  查看cpu占比排序（倒序）
top -d 2 -n 3 -p 3306  #查找固定进程的情况，每两秒刷新一次，刷新3次
```
**cat**
```bash
cat > felix < /etc/passwd    #把/etc/passwd的内容重定向到 felix,从标准输入重定向标准输出到 felix
cat > felix                 #从键盘中重定向输出到 felix
asdf
asdfsdf
```
**awk**
```bash
cat data|awk '{sum+=$1} END {print "Sum = ", sum}'
cat data|awk '{sum+=$1} END {print "Average = ", sum/NR}'
cat data|awk 'BEGIN {max = 0} {if ($1>max) max=$1 fi} END {print "Max=", max}'
awk 'BEGIN {min = 1999999} {if ($1<min) min=$1 fi} END {print "Min=", min}'
```
**sed**
```
grep -n abc xxx.txt   包含有abc的一行
sed -n '9,15p' passwd   第9行到第15行
sed -n '9p;15p' passwd  第9行和第15行
tail -n 2 command.txt | head -n 1    最后两行的第一行
head -c 20 log2014.log    前20个字节
head -c -32 log2014.log   除了最后32个字节以外的内容
```
**sort,cut**

```
sourt 对文本行进行去重并统计重复次数 (uniq命令加-c选项可以实现对重复次数进行统计) 
$ sort test.txt | uniq -c 

cut 命令可以按列操作文本行。
可以看出前面的重复次数占8个字符，因此，可以用命令cut -c 9- 取出每行第9个及其以后的字符。  
$ sort test.txt | uniq -c | sort -rn | cut -c 9-  
```
**tar**

```
tar -czvf test.tar.gz ### --exclude c.log --exclude logs
tar –exclude /etc/a/b -zPcvf /home/a/a.gz /etc/a
```
**diff**
```
diff log2013.log log2014.log  -y -W 50
-W<宽度>或--width<宽度> 　在使用-y参数时，指定栏宽。
-y或--side-by-side 　以并列的方式显示文件的异同之处。
"|"表示前后2个文件内容有不同
"<"表示后面文件比前面文件少了1行内容
">"表示后面文件比前面文件多了1行内容
```
**EOF**
```
# 从终端输入重定的到 felix
felix   << EOF
       en
       conf t
       interface range f0/0-f0/20  #进入端口，可修改
       no dot
       end                  
       wr
       exit
EOF
```
**linux版本**
```bash
cat /proc/version
cat /etc/issue
uname -a
cat /etc/redhat_release
```
**yum**
```bash
#yum的配置文件(/etc/yum.repos.d/###.repo)
[kyo]       仓库配置名
name=kyo
#配置本地仓库
baseurl=file:///iso/cd1
		file:///iso/cd2
enabled=1
gpgcheck=0
#配置网络仓库
baseurl=http://3.3.3.1/centos1
        http://3.3.3.1/centos2
```
**公钥**
```
ssh-keygen
ssh-copy-id 192.168.1.1   把公钥发到对方
chmod 600 authorized_keys 设置权限
ssh -p xxx felix@server   端口变了
```
**umask** 
```
新建文件夹或文件的权限是由所谓基本码减去称之为umask的屏蔽位得到的。
按照规定：文件夹的基本码是rwxrwxrwx(777)，文件的基本码是rw-rw-rw-(666)
因此新建文件夹是777-022=755(rwxr-xr-x),新建文件是666-022=644(rw-r–r–)。
```
**导出mysql**

	1. mysqldump -u user -p passwd dbname > daname.sql

**导入mysql**

	1. mysql -u user -ppasswd dbname < dbname.sql


