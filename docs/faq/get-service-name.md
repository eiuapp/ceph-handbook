通过 systemctl status 查找服务名称

比如，我现在把 rgw 安装在了一台机器上，但是，我不知道rgw的service全名是什么，这时，我修改了配置文件，就会不知道怎么重启服务。
这里可以按下面方式找到服务。



```
root@ceph-client:/etc/ceph# systemctl status | grep rados
           │     └─101359 grep --color=auto rados
             ├─system-ceph\x2dradosgw.slice
             │ └─ceph-radosgw@rgw.ceph-client.service
             │   └─101280 /usr/bin/radosgw -f --cluster ceph --name client.rgw.ceph-client --setuser ceph --setgroup ceph
root@ceph-client:/etc/ceph# systemctl status ceph-radosgw@rgw.ceph-client
● ceph-radosgw@rgw.ceph-client.service - Ceph rados gateway
   Loaded: loaded (/lib/systemd/system/ceph-radosgw@.service; enabled; vendor preset: enabled)
   Active: active (running) since 二 2019-03-12 15:44:27 CST; 21s ago
 Main PID: 101280 (radosgw)
   CGroup: /system.slice/system-ceph\x2dradosgw.slice/ceph-radosgw@rgw.ceph-client.service
           └─101280 /usr/bin/radosgw -f --cluster ceph --name client.rgw.ceph-client --setuser ceph --setgroup ceph

3月 12 15:44:27 ceph-client systemd[1]: Started Ceph rados gateway.
root@ceph-client:/etc/ceph# 
```
