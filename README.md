# system-init

## 系统环境
```
系统版本：CentOS Linux release 7.6.1810 (Core)
系统内核：3.10.0-957.el7.x86_64
网络环境：要求服务器可以连通网络
规格大小：建议使用不小于1C1G
网卡数量：建议不小于1块网卡
系统盘：  建议使用不小于100GB的空间
数据盘：  建议使用不小于100GB的空间（可选）
```

## 挂载数据盘(可选)
如果服务器需要挂载数据盘，可使用以下快捷命令
```
mkfs.xfs -f /dev/sdb
mkdir -p /data
mount /dev/sdb /data/
 echo "/dev/sdb                                  /data                   xfs     defaults        0 0" >>/etc/fstab ; cat /etc/fstab |grep data
```

## 配置YUM仓库
```
rm -f /etc/yum.repos.d/*.repo
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo
yum repolist
```

## 软件包安装
```
yum -y install ansible vim git
```

## 拉取远程代码
```
cd /root/
git clone https://gitee.com/chriscentos/system-init.git
```

## 复制模版信息
```
cd /root/system-init/
\cp group_vars/all-template group_vars/all
\cp hosts-example hosts
```

## 配置主机列表
vim hosts
```
[nodes01]
192.168.1.10 

[all:vars]
ansible_ssh_port=22            ## 设置远程服务器端口
ansible_ssh_user=root          ## 设置远程服务器用户
ansible_ssh_pass="bkce123"     ## 设置远程服务器密码
```

## 生成密钥对
检查是否存在密钥对
```
# ls -l /root/.ssh/id_rsa*
-rw------- 1 root root 1679 Dec  3 22:51 /root/.ssh/id_rsa
-rw-r--r-- 1 root root  403 Dec  3 22:51 /root/.ssh/id_rsa.pub
```
若已生成密钥对，则可以跳过此操作
```
# ssh-keygen    ## 执行此命令，然后一直回车即可
```

## 初始化变量
vim group_vars/all
```
# system init size percentage
ntp_server_host: 'ntp1.aliyun.com' #配置NTP地址
dns_server_host: '114.114.114.114' #配置DNS地址

# Safety reinforcement related configuration
auth_keys_file: '/root/.ssh/authorized_keys' #设置key文件地址
password_auth: 'yes' #是否开启密码登录，yes表示开启 no表示不开启
root_public_key: ''  #设置免密公钥
root_passwd: 'bkce123'
```
公钥信息获取方式：cat /root/.ssh/id_rsa.pub 

## 服务器检查
检查远程服务器的SSH连接状态
```
ansible -m ping nodes01
192.168.1.10 | SUCCESS => {
```

## 服务器优化
```
ansible-playbook playbooks/system_init.yml -e "nodes=nodes01"
```

## 检查服务器
验证远程服务器
```
1.检查是否免密
# ssh 192.168.1.1
[root@linux-bkce-node1 ~]#

2.检查dns设置
# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 

3.检查ntp时间同步状态
# ntpstat 
synchronised to NTP server (120.25.115.20) at stratum 3
   time correct to within 25 ms
   polling server every 64 s
如果出现以下问题：
# ntpstat 
Unable to talk to NTP daemon. Is it running?
建议重启一下chronyd服务即可： systemctl restart chronyd

4.检查数据盘是否挂载(可选)
# df -h|grep data
/dev/sdb        500G   61G  440G  13% /data
```
