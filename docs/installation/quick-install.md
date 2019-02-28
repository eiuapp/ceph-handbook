
本文是快速安装

根据 http://docs.ceph.com/docs/master/start/ 安装 ceph 开发环境

## env

软件

- os：Ubuntu Server 16.04×64 
- Python: 2.7.12
- ceph：v13.2.4 Mimic
- ceph-deploy：2.0.1


硬件

每一台机器都是 派+硬盘 模式，派上放os，硬盘存储

- 存储设置：4T 与 操作系统分离


架构部署

| IP| 部件   |    OSD      |  Monitor | Managers | MDSs |
|:--------|:--------|:----------|:-----------|:----------|:------|
| 192.168.0.185| ceph-admin |     |   |  |  |
| 192.168.0.134| ceph-client |     |   |  |  |
| 192.168.0.130| ceph-server-1 |     |  Y | Y |  |
| 192.168.0.111| ceph-server-2 |   Y  |   |  |  |
| 192.168.0.105| ceph-server-3 |   Y  |   |  |  |
## step

除非说明，则均为所有节点

### etc-hosts

配置 /etc/hosts 文件

```
192.168.0.185 ceph-admin   
192.168.0.134 ceph-client  
192.168.0.130 ceph-server-1 
192.168.0.111 ceph-server-2 
192.168.0.105 ceph-server-3
```

`hostnamectl --static set-hostname ***` 设置 hostname 
```
ceph-admin
ceph-client
ceph-server-1 
ceph-server-2 
ceph-server-3
```
### ntp
```
sudo apt-get update && sudo apt-get install -yq ntp openssh-server
```

### 创建 ceph 用户
```
ssh user@ceph-server
sudo useradd -d /home/{username} -m {username}
sudo passwd {username}
echo "{username} ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/{username}
sudo chmod 0440 /etc/sudoers.d/{username}
```
因为我这里的环境已经有 admin 用户，则把 admin 作为 ceph 用户

```
echo "admin ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/admin
sudo chmod 0440 /etc/sudoers.d/admin
```
### ceph-deploy （ceph-admin节点）

```
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
echo deb https://download.ceph.com/debian-mimic/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list
sudo apt-get update && sudo apt-get install -yq ceph-deploy 
ceph-deploy --version
```

### ENABLE PASSWORD-LESS SSH（ceph-admin节点）
```
ssh-keygen
```
### Copy the key to each Ceph Node （ceph-admin节点）
```
ssh-copy-id {username}@node1
ssh-copy-id {username}@node2
ssh-copy-id {username}@node3
```
检查一下，是否能免密码登录

### OPEN REQUIRED PORTS

如果是实验环境，这一步可暂时忽略。

如果是由 firewall-cmd 管理的话：

For example, on monitors:
```
sudo firewall-cmd --zone=public --add-service=ceph-mon --permanent
```
and on OSDs and MDSs:
```
sudo firewall-cmd --zone=public --add-service=ceph --permanent
```
Once you have finished configuring firewalld with the --permanent flag, you can make the changes live immediately without rebooting:
```
sudo firewall-cmd --reload
```
### CREATE A CLUSTER （ceph-admin节点）

#### ceph-deploy install （ceph-admin节点）
```
ceph-deploy new ceph-server-1 ceph-server-2 ceph-server-3
echo "osd pool default size = 2" >> ceph.conf 
echo "mon_clock_drift_allowed = 1" >> ceph.conf 
cat ceph.conf 
ceph-deploy install ceph-admin ceph-server-1 ceph-server-2 ceph-server-3 ceph-client
ceph -v
```

#### Configure monitor and OSD services（ceph-admin节点）
```
ceph-deploy mon create-initial
ls
```
#### Configuration and status（ceph-admin节点）
```
ceph-deploy admin ceph-admin ceph-server-1 ceph-server-2 ceph-server-3 ceph-client
ceph -v
ls /etc/ceph/ -l
sudo chmod +r /etc/ceph/ceph.client.admin.keyring
ssh ceph-server-1 sudo chmod +r /etc/ceph/ceph.client.admin.keyring
ssh ceph-server-2 sudo chmod +r /etc/ceph/ceph.client.admin.keyring
ssh ceph-server-3 sudo chmod +r /etc/ceph/ceph.client.admin.keyring
```
#### Configure mgr （ceph-admin节点）

```
ceph-deploy mgr create ceph-server-1
```
### ceph health （除 ceph-client 节点）

```
ceph health
ceph -s
```

## ref
- http://docs.ceph.com/docs/master/start/