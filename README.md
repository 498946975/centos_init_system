# centos_init_system

## 系统环境
```yaml
系统版本：CentOS Linux release 7.6.1810 (Core)
系统内核：3.10.0-957.el7.x86_64
网络环境：要求服务器可以连通网络
规格大小：建议使用不小于1C1G
网卡数量：建议不小于1块网卡
系统盘：  建议使用不小于100GB的空间
数据盘：  建议使用不小于100GB的空间（可选）
```


## ansible主机，配置YUM仓库
```shell script
rm -f /etc/yum.repos.d/*.repo
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.cloud.tencent.com/repo/centos7_base.repo
curl -o /etc/yum.repos.d/epel.repo http://mirrors.cloud.tencent.com/repo/epel-7.repo
yum repolist
```

## ansible主机，软件包安装
```shell script
yum -y install ansible vim git
```

## ansible主机，拉取远程代码
```shell script
cd /root/
git clone https://github.com/498946975/centos_init_system.git
```

## ansible主机，复制模版信息
```shell script
cd /root/system-init/
\cp group_vars/all-template group_vars/all
\cp hosts-example hosts
```

## ansible主机，配置主机列表
vim hosts
```shell script
[test]
172.16.120.159

[all:vars]
ansible_ssh_port=22            ## 设置远程服务器端口
ansible_ssh_user=root          ## 设置远程服务器用户
ansible_ssh_pass="123"     ## 设置远程服务器密码
```

## ansible主机，生成密钥对
检查是否存在密钥对
```shell script
# ls -l /root/.ssh/id_rsa*
-rw------- 1 root root 1679 Dec  3 22:51 /root/.ssh/id_rsa
-rw-r--r-- 1 root root  403 Dec  3 22:51 /root/.ssh/id_rsa.pub
```
ansible主机，若已生成密钥对，则可以跳过此操作
```
# ssh-keygen    ## 执行此命令，然后一直回车即可
```

## ansible主机，初始化变量
vim group_vars/all
```shell script
# system init size percentage
ntp_server_host: 'ntp.aliyun.com' #配置NTP地址
dns_server_host: '114.114.114.114' #配置DNS地址

# Safety reinforcement related configuration
auth_keys_file: '/root/.ssh/authorized_keys' #设置key文件地址
password_auth: 'yes' #是否开启密码登录，yes表示开启 no表示不开启
root_public_key: ''  #设置免密公钥
root_passwd: '123'
```
ansible主机，公钥信息获取方式：cat /root/.ssh/id_rsa.pub 

## ansible主机，服务器检查
检查远程服务器的SSH连接状态
```shell script
ansible -m ping test

172.16.120.159 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
```

## ansible主机，对test服务器优化
```shell script
ansible-playbook playbooks/system_init.yml -e "nodes=test"
```

## 检查test服务器
验证远程服务器
```
1.检查是否免密
# ssh 192.168.1.1
[root@test ~]# exit

2.检查dns设置
# cat /etc/resolv.conf 
nameserver 114.114.114.114

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
