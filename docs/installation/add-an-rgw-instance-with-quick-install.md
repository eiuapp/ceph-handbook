
在 ceph deploy quick install 基础上安装一个 RGW INSTANCE

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

RGW 安装 在 ceph-client 上。

## step

### 查看ceph 状态

```shell
admin@ceph-admin:~/test-cluster$ ceph health
HEALTH_OK
admin@ceph-admin:~/test-cluster$ 
admin@ceph-admin:~/test-cluster$ ceph -s
  cluster:
    id:     87aa8ebd-2782-415d-bf23-ad2eb89b61c6
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-server-3,ceph-server-2,ceph-server-1
    mgr: ceph-server-1(active)
    osd: 2 osds: 2 up, 2 in
    rgw: 1 daemon active
 
  data:
    pools:   5 pools, 40 pgs
    objects: 187  objects, 1.1 KiB
    usage:   2.0 GiB used, 7.3 TiB / 7.3 TiB avail
    pgs:     40 active+clean
 
admin@ceph-admin:~/test-cluster$ 
```

### （此步看需求，可忽略）修改RGW访问端口7480为8080

RGW开始运，我们就可以通过端口7480（如果没有修改的话）来访问。如果你希望修改成8080的话，往下看。

修改配置
```
admin@ceph-admin:~/test-cluster$ vi ceph.conf 
```
添加如下内容, 其中`ceph-client`是你希望添加RGW的节点名，8080为将来访问port。

```
[client.rgw.ceph-client]
rgw_frontends = "civetweb port=8080"
```

### 分发配置文件

```
ceph-deploy --overwrite-conf config push ceph-admin ceph-client ceph-server-1 ceph-server-2 ceph-server-3
```

### 安装CEPH OBJECT GATEWAY

```
ceph-deploy install --rgw ceph-client
```

### 管理RGW节点

Ceph CLI工具需要在管理员模式下运行，因此需要执行以下命令：

```
ceph-deploy admin ceph-client
```



### 安装RGW实例
执行命令：

```
ceph-deploy rgw create ceph-client 
```

一旦RGW开始运行，我们就可以通过端口7480（如果没有修改的话）来访问。如：

```
http://ceph-client:7480
```

如果RGW运行正常，它应该返回类似的信息：

```
<ListAllMyBucketsResult>
  <Owner>
    <ID>anonymous</ID>
    <DisplayName/>
  </Owner>
  <Buckets/>
</ListAllMyBucketsResult>
```

我的实操：

```
admin@ceph-admin:~/test-cluster$ curl http://ceph-client:8080
<?xml version="1.0" encoding="UTF-8"?><ListAllMyBucketsResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/"><Owner><ID>anonymous</ID><DisplayName></DisplayName></Owner><Buckets></Buckets></ListAllMyBucketsResult>admin@ceph-admin:~/test-cluster$ 
admin@ceph-admin:~/test-cluster$ curl http://ceph-client:8080 -I
HTTP/1.1 200 OK
x-amz-request-id: tx000000000000000000003-005c77a1a5-10c0-default
Content-Type: application/xml
Content-Length: 0
Date: Thu, 28 Feb 2019 08:53:57 GMT

admin@ceph-admin:~/test-cluster$ 
```

注意：剩下的创建用户的步骤都应该在RGW节点上运行。

### 对象存储 操作

```
echo 'hello ceph oject storage' > testfile.txt

创建pool
ceph osd pool create mytest 8

上传文件
rados put test-object-1 testfile.txt --pool=mytest


rados -p mytest ls

获取文件
rados get test-object-1 testfile.txt.1 --pool=mytest

查看映射位置
ceph osd map mytest test-object-1

删除文件
rados rm test-object-1 --pool=mytest
删除pool
ceph osd pool rm mytest mytest --yes-i-really-really-mean-it
```

实操：

```
admin@ceph-admin:~/test-cluster$ mkdir ~/test-data/ && cd ~/test-data/ 
admin@ceph-admin:~/test-cluster$ echo "this is my first test data" > testfile.txt
admin@ceph-admin:~/test-data$ ceph osd pool create mytest 8
admin@ceph-admin:~/test-data$ rados put test-object-1 ./testfile.txt --pool=mytest
admin@ceph-admin:~/test-cluster$ rados -p mytest ls
test-object-1
admin@ceph-admin:~/test-cluster$ ceph osd map mytest test-object-1
osdmap e26 pool 'mytest' (2) object 'test-object-1' -> pg 2.74dc35e2 (2.2) -> up ([1,0], p1) acting ([1,0], p1)
admin@ceph-admin:~/test-cluster$ 
```

这样，就完成了 RGW 的基本操作。

#### (忽略)`ceph osd pool create mytest 8` 后

此时，ceph -s 会发现 在 data/gps 中会多出 

```
    pgs:     100.000% pgs unknown
             8 unknown
```
这样的内容。

### 查看状态

```
admin@ceph-admin:~/test-data$ ceph -s
  cluster:
    id:     87aa8ebd-2782-415d-bf23-ad2eb89b61c6
    health: HEALTH_WARN
            application not enabled on 1 pool(s)
 
  services:
    mon: 3 daemons, quorum ceph-server-3,ceph-server-2,ceph-server-1
    mgr: ceph-server-1(active)
    osd: 2 osds: 2 up, 2 in
    rgw: 1 daemon active
 
  data:
    pools:   5 pools, 40 pgs
    objects: 188  objects, 1.2 KiB
    usage:   2.0 GiB used, 7.3 TiB / 7.3 TiB avail
    pgs:     40 active+clean
 
  io:
    client:   85 B/s wr, 0 op/s rd, 0 op/s wr
             
admin@ceph-admin:~/test-cluster$ 
```

### 记录一下配置文件

```
admin@ceph-admin:~/test-cluster$ ls
ceph.bootstrap-mds.keyring  ceph.bootstrap-mgr.keyring  ceph.bootstrap-osd.keyring  ceph.bootstrap-rgw.keyring  ceph.client.admin.keyring  ceph.conf  ceph-deploy-ceph.log  ceph.mon.keyring  release.asc
admin@ceph-admin:~/test-cluster$ cat ceph.conf
[global]
fsid = 87aa8ebd-2782-415d-bf23-ad2eb89b61c6
mon_initial_members = ceph-server-1, ceph-server-2, ceph-server-3
mon_host = 192.168.0.130,192.168.0.111,192.168.0.105
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx

osd pool default size = 2
mon_clock_drift_allowed = 1

[client.rgw.ceph-client]
rgw_frontends = "civetweb port=8080"
admin@ceph-admin:~/test-cluster$ ls /etc/ceph/
ceph.client.admin.keyring  ceph.conf  rbdmap  tmp4FMEG8  tmpEO0ZBg
admin@ceph-admin:~/test-cluster$ 
```

### 如果 RGW 安装过程中出了问题

- 通过netstat查看 RGW 配置的端口，判断rgw服务是否启动

```
admin@swift0134:~$ netstat -tnlp | grep 8080
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      -               
admin@swift0134:~$ sudo systemctl restart ceph-radosgw@rgw.ceph-client.service 
```
- 查看rgw服务状态。`sudo systemctl status ceph-radosgw@rgw.ceph-client.service` 

```
admin@swift0134:~$ sudo systemctl status ceph-radosgw@rgw.ceph-client.service 
admin@swift0134:~$ sudo systemctl restart ceph-radosgw@rgw.ceph-client.service  
```

## ref
- http://docs.ceph.com/docs/master/start/quick-ceph-deploy/
- https://blog.csdn.net/styshoo/article/details/58572816
- https://my.oschina.net/guol/blog/1928348


