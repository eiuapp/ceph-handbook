
## env


基于 之前 add-an-rgw-instance-with-quick-install ，因此有：

- ceph rgw 的 http 接口是： http://ceph-client:8080/
- ceph-client ip 是： 192.168.0.134 ，以下只用此IP表示 ceph rgw 接口。
- ceph-admin: 192.168.0.185
- keystone: 192.168.0.51 

swift client : 192.168.0.51, 此上必须有 swift 命令

### 现象

之前正常的。突然某个时间点 swift list 报错了

```
root@controller:~# . admin-openrc 
root@controller:~# swift list
Account HEAD failed: http://ceph-rgw-client:8080/swift/v1 401 Unauthorized
Failed Transaction ID: tx000000000000000000104-0056bcd49c-5ec1-default
root@controller:~# 
```

或者

在 访问 horizon ： http://192.168.0.51/horizon/project/containers/ 时，会自动退出，并跳到登录页面。


## step 

原理：因为 ceph-rgw-client 重启了一次。

解决：重启ceph rgw 节点

```
root@ceph-client:/etc/ceph# systemctl restart ceph-radosgw@rgw.ceph-client
root@ceph-client:/etc/ceph# systemctl status ceph-radosgw@rgw.ceph-client
● ceph-radosgw@rgw.ceph-client.service - Ceph rados gateway
   Loaded: loaded (/lib/systemd/system/ceph-radosgw@.service; enabled; vendor preset: enabled)
   Active: active (running) since 五 2016-02-12 02:42:28 CST; 11s ago
 Main PID: 4109 (radosgw)
   CGroup: /system.slice/system-ceph\x2dradosgw.slice/ceph-radosgw@rgw.ceph-client.service
           └─4109 /usr/bin/radosgw -f --cluster ceph --name client.rgw.ceph-client --setuser ceph --setgroup ceph

2月 12 02:42:28 ceph-client systemd[1]: Started Ceph rados gateway.
root@ceph-client:/etc/ceph# 
```

再次访问就正常了

```
root@controller:~# . admin-openrc 
root@controller:~# swift list 
123
test_1
root@controller:~# 
```

