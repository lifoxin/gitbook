* ansible
```
yum -y install ansible                               #安装包
ssh-keygen                                           #生成秘钥，设置可密钥远程
vim /etc/ansible/hosts                               # 设置定义组，域名，ip，等很多设置
ansible-doc -l                                       # 列出 Ansible 支持的模块
ansible-doc ping                                     # 查看该模块帮助信息
ansible Client -m command -a "free -m"               # 查看 Client 分组主机内存使用情况
ansible Client -m script -a "/home/test.sh 12 34"    # 远程执行本地脚本
ansible Client -m shell -a "/home/test.sh"           # 执行远程脚本
#向 Client 组中主机拷贝 test.sh 到 /tmp 下，属主、组为 root ，权限为 0755
ansible Client -m copy -a "src=/home/test.sh desc=/tmp/ owner=root group=root mode=0755"  
ansible Client -m stat -a "path=/etc/syctl.conf"     # 获取远程文件状态信息
ansible Client -m yum -a "name=curl state=latest"   # yum 模块，安装包
ansible Client -m cron -a "name='check dirs' hour='5,2' job='ls -alh > /dev/null'"  #任务
ansible Client -m mount -a "name=/mnt/data src=/dev/sd0 fstype=ext4 opts=ro state=present" 
ansible Client -m service -a "name=nginx state=stoped" #service模块
# 使用 Ansible-playbook 可以完成一组复杂的动作，例如部署环境、搭建服务、修改配置等。
```
