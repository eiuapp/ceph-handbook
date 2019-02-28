
这里是贯穿本书的环境

## env-1

软件

- os: windows 10
- 命令行： git-bash或者cmder或者cmd
- virtual: 6.0.4
- vagrant: 2.2.3
- vagrant box: ubuntu/bionic64
- ceph：V12.2.11 LUMINOUS
- ceph-deploy：1.5.38

硬件

- host: virtualbox 虚拟机
- cpu: 2
- RAM: 1GB

架构部署

| IP| 部件   |    OSD      |  Monitor | Managers | MDSs |
|:--------|:----------|:-------------|:-------------|:-------------|:------|
| 172.21.12.10| ceph-admin |     |   |  |  |
| 172.21.12.11| ceph-client |     |   |  |  |
| 172.21.12.12| ceph-server-1 |     |  Y | Y |  |
| 172.21.12.13| ceph-server-2 |   Y  |   |  |  |
| 172.21.12.14| ceph-server-3 |   Y  |   |  |  |


## env-2 

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