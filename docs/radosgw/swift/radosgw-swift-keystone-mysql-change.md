当keystone 相关联的数据库 出问题时，mysql作迁移后，ceph与swift集成所受影响


结论：

- 不受影响。
- 如果未迁移数据内容，则需要添加swift 与 ceph 集成的endpoint。（操作可参考 swift 与 ceph 集成操作）

## env

因其它原因，keystone关联的mysql出错，无法使用，用数据转至其它机器时

基于 之前 add-an-rgw-instance-with-quick-install ，因此有：

- ceph rgw 的 http 接口是： http://ceph-client:8080/
- ceph-client ip 是： 192.168.0.134 ，以下只用此IP表示 ceph rgw 接口。
- ceph用户一个。注意：此用户是直接通过，`radosgw-admin user create` 创建，并非 keystone 用户。

- endpoint 中 service 是： 

- rgw 节点配置文件/etc/ceph.conf 并没有配置：

```
swift account in url = true
```

## step

### 直接获取不到容器信息
```
root@controller:/home/ubuntu# cat admin-openrc 
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=openstack
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2

root@controller:/home/ubuntu# 
root@controller:/home/ubuntu# . admin-openrc 
root@controller:/home/ubuntu# swift list
Endpoint for object-store not found - have you specified a region?
root@controller:/home/ubuntu# 
```


### 确保在集成之前，用户信息有效

```
root@controller:/home/ubuntu# . ceph-swift-user1-openrc
root@controller:/home/ubuntu# swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password} list
Authorization Failure. Authorization failed: Method Not Allowed (HTTP 405)
root@controller:/home/ubuntu# 
root@controller:/home/ubuntu# swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password} list swiftuser1-container1
Authorization Failure. Authorization failed: Method Not Allowed (HTTP 405)
root@controller:/home/ubuntu#
```


这里报出来的是 405 错误，与之前的 401 不同。

那说明，ceph本身有问题

**注意：此时rgw接口依然是有效的，所以仅通过 http://192.168.0.134:8080/ 可访问来判断ceph集群是否正常，是无依据的。**

来到 ceph-admin 节点 0.185 

```
admin@ceph-admin:~$ ceph -s
  cluster:
    id:     87aa8ebd-2782-415d-bf23-ad2eb89b61c6
    health: HEALTH_WARN
            application not enabled on 1 pool(s)
            clock skew detected on mon.ceph-server-1
 
  services:
    mon: 3 daemons, quorum ceph-server-3,ceph-server-2,ceph-server-1
    mgr: ceph-server-1(active)
    osd: 2 osds: 2 up, 2 in
    rgw: 1 daemon active
 
  data:
    pools:   7 pools, 56 pgs
    objects: 2.28 k objects, 3.7 GiB
    usage:   9.5 GiB used, 7.3 TiB / 7.3 TiB avail
    pgs:     56 active+clean
 
admin@ceph-admin:~$ 
```

这个知道的，时间问题

http://eiuapp.github.io/ceph-handbook/docs/faq/ntpdate.html

解决后

```
admin@ceph-admin:~$ ceph -s
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
    pools:   7 pools, 56 pgs
    objects: 2.28 k objects, 3.7 GiB
    usage:   9.5 GiB used, 7.3 TiB / 7.3 TiB avail
    pgs:     56 active+clean
 
admin@ceph-admin:~$ 
```

这个是 application not enabled 问题，参考 http://eiuapp.github.io/ceph-handbook/docs/faq/ntpdate.html


```
admin@ceph-admin:~$ ceph -s
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
    pools:   7 pools, 56 pgs
    objects: 2.28 k objects, 3.7 GiB
    usage:   9.5 GiB used, 7.3 TiB / 7.3 TiB avail
    pgs:     56 active+clean
 
admin@ceph-admin:~$ ceph health detail
HEALTH_WARN application not enabled on 1 pool(s)
POOL_APP_NOT_ENABLED application not enabled on 1 pool(s)
    application not enabled on pool 'mytest'
    use 'ceph osd pool application enable <pool-name> <app-name>', where <app-name> is 'cephfs', 'rbd', 'rgw', or freeform for custom applications.
admin@ceph-admin:~$ ceph osd pool application enable mytest rbd
enabled application 'rbd' on pool 'mytest'
admin@ceph-admin:~$ ceph -s
  cluster:
    id:     87aa8ebd-2782-415d-bf23-ad2eb89b61c6
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-server-3,ceph-server-2,ceph-server-1
    mgr: ceph-server-1(active)
    osd: 2 osds: 2 up, 2 in
    rgw: 1 daemon active
 
  data:
    pools:   7 pools, 56 pgs
    objects: 2.28 k objects, 3.7 GiB
    usage:   9.5 GiB used, 7.3 TiB / 7.3 TiB avail
    pgs:     56 active+clean
 
admin@ceph-admin:~$ 
```

好了，现在 ceph 健康了。


**注意，这个时候，不应该使用原来的 窗口 查询 swift 信息，应该另开一个窗口来查询。不然，还是会得到错误。具体原因，不知道。**

```
root@controller:/home/ubuntu# . ceph-swift-user1-openrc
root@controller:/home/ubuntu# swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password}  list  
Authorization Failure. Authorization failed: Method Not Allowed (HTTP 405)
root@controller:/home/ubuntu# swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password}  list  
Authorization Failure. Authorization failed: Unable to establish connection to http://192.168.0.134:8080/auth/v1.0/auth/tokens
root@controller:/home/ubuntu# 
```

### swift用户信息

```
root@controller:/home/ubuntu# . admin-openrc 
root@controller:/home/ubuntu# swift stat -v
Authorization Failure. Authorization failed: An unexpected error prevented the server from fulfilling your request. (HTTP 500) (Request-ID: req-71675d8c-3e88-42b7-85ad-b285b26ac1d9)
root@controller:/home/ubuntu# 
```

因为之前，我们知道，这个应该是 endpoint list 没有加入swift环境导致。这样我们就要把之前的 swift 与 ceph 集成的操作再做一次。

### swift 与 ceph 集成

```
root@controller:/home/ubuntu# ping ceph-rgw-client 
PING ceph-rgw-client (192.168.0.134) 56(84) bytes of data.
64 bytes from ceph-rgw-client (192.168.0.134): icmp_seq=1 ttl=64 time=1.66 ms
^C
--- ceph-rgw-client ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.666/1.666/1.666/0.000 ms
root@controller:/home/ubuntu# openstack endpoint list
+----------------------------------+-----------+--------------+--------------+---------+-----------+----------------------------+
| ID                               | Region    | Service Name | Service Type | Enabled | Interface | URL                        |
+----------------------------------+-----------+--------------+--------------+---------+-----------+----------------------------+
| 1c1c7306ea024eb98fae578ed1383f99 | RegionOne | keystone     | identity     | True    | internal  | http://controller:5000/v3/ |
| b9499be1f9ac4ad8ad9705f2e1a13847 | RegionOne | keystone     | identity     | True    | public    | http://controller:5000/v3/ |
| badb42c672534b8290a37b828b758c9d | RegionOne | keystone     | identity     | True    | admin     | http://controller:5000/v3/ |
+----------------------------------+-----------+--------------+--------------+---------+-----------+----------------------------+
root@controller:/home/ubuntu# openstack service create --name=swift \
>                          --description="Swift Service" \
>                          object-store
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Swift Service                    |
| enabled     | True                             |
| id          | 632c97c278784f5caae69cc9fe9df549 |
| name        | swift                            |
| type        | object-store                     |
+-------------+----------------------------------+
root@controller:/home/ubuntu# openstack endpoint create --region RegionOne   object-store public http://ceph-rgw-client:8080/swift/v1
+--------------+--------------------------------------+
| Field        | Value                                |
+--------------+--------------------------------------+
| enabled      | True                                 |
| id           | 5be5403aecb9418aa73288b0f56e1315     |
| interface    | public                               |
| region       | RegionOne                            |
| region_id    | RegionOne                            |
| service_id   | 632c97c278784f5caae69cc9fe9df549     |
| service_name | swift                                |
| service_type | object-store                         |
| url          | http://ceph-rgw-client:8080/swift/v1 |
+--------------+--------------------------------------+
root@controller:/home/ubuntu# openstack endpoint create --region RegionOne   object-store internal http://ceph-rgw-client:8080/swift/v1
+--------------+--------------------------------------+
| Field        | Value                                |
+--------------+--------------------------------------+
| enabled      | True                                 |
| id           | 4fa93f815d64428697bcc41e56f38d94     |
| interface    | internal                             |
| region       | RegionOne                            |
| region_id    | RegionOne                            |
| service_id   | 632c97c278784f5caae69cc9fe9df549     |
| service_name | swift                                |
| service_type | object-store                         |
| url          | http://ceph-rgw-client:8080/swift/v1 |
+--------------+--------------------------------------+
root@controller:/home/ubuntu# openstack endpoint create --region RegionOne   object-store admin http://ceph-rgw-client:8080/swift/v1
+--------------+--------------------------------------+
| Field        | Value                                |
+--------------+--------------------------------------+
| enabled      | True                                 |
| id           | b7df0ccdceba42ce8272d26882627e33     |
| interface    | admin                                |
| region       | RegionOne                            |
| region_id    | RegionOne                            |
| service_id   | 632c97c278784f5caae69cc9fe9df549     |
| service_name | swift                                |
| service_type | object-store                         |
| url          | http://ceph-rgw-client:8080/swift/v1 |
+--------------+--------------------------------------+
root@controller:/home/ubuntu# openstack endpoint list
+----------------------------------+-----------+--------------+--------------+---------+-----------+--------------------------------------+
| ID                               | Region    | Service Name | Service Type | Enabled | Interface | URL                                  |
+----------------------------------+-----------+--------------+--------------+---------+-----------+--------------------------------------+
| 1c1c7306ea024eb98fae578ed1383f99 | RegionOne | keystone     | identity     | True    | internal  | http://controller:5000/v3/           |
| 4fa93f815d64428697bcc41e56f38d94 | RegionOne | swift        | object-store | True    | internal  | http://ceph-rgw-client:8080/swift/v1 |
| 5be5403aecb9418aa73288b0f56e1315 | RegionOne | swift        | object-store | True    | public    | http://ceph-rgw-client:8080/swift/v1 |
| b7df0ccdceba42ce8272d26882627e33 | RegionOne | swift        | object-store | True    | admin     | http://ceph-rgw-client:8080/swift/v1 |
| b9499be1f9ac4ad8ad9705f2e1a13847 | RegionOne | keystone     | identity     | True    | public    | http://controller:5000/v3/           |
| badb42c672534b8290a37b828b758c9d | RegionOne | keystone     | identity     | True    | admin     | http://controller:5000/v3/           |
+----------------------------------+-----------+--------------+--------------+---------+-----------+--------------------------------------+
root@controller:/home/ubuntu# 
```

### 再验证

```
root@controller:~# . admin-openrc 
root@controller:~# swift list
root@controller:~# . demo-openrc 
root@controller:~# swift list
root@controller:~# 
```

可以了。

**注意：这里看到，原有的数据内容已经不存在了。但是，实际上，原admin用户是有存放数据在 ceph 中的。**

这是个巨大风险点呀。

**注意：同时，我们之前的原生用户 swiftuser1 不能访问了。**

```
root@controller:/home/ubuntu# . ceph-swift-user1-openrc
root@controller:/home/ubuntu# swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password}  list  
Authorization Failure. Authorization failed: Method Not Allowed (HTTP 405)
root@controller:/home/ubuntu# . admin-openrc 
root@controller:/home/ubuntu# swift list
abcd
root@controller:/home/ubuntu# 
```


