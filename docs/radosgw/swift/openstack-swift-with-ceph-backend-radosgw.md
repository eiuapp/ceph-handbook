
在 openstack 中用ceph作为 对象存储的后端, python-swiftclient作为客户端，集成openstack keystone.

正在尝试使用radosgw与Ceph（Minic）一起使用Openstack（liberty）swift。目的是将对象存储在ceph osds下。我有一个工作的Openstack和Ceph集群。

要使用Ceph作为对象存储后端，我在ceph集群中安装并配置了radosgw。在openstack节点中，我安装了“python-swiftclient”，创建了一个对象存储服务，并为该服务添加了一个URL为radosgw的端点。

我按照以下链接中的说明进行操作。 http://docs.ceph.com/docs/jewel/radosgw/keystone/


根据 http://docs.ceph.com/docs/master/radosgw/keystone/ 集成 keystone 

## env

swift client: 192.168.0.51
keystone: 192.168.0.51
ceph-rgw: 192.168.0.134
ceph-admin: 192.168.0.185

## step

### 确保在集成之前，用户信息有效

Connecting to 192.168.0.51 

```
root@controller:/home/ubuntu# cat ceph-swift-user1-openrc
username=swiftuser1
subusername=swiftsubuser1
displayname=swiftuser1displayname
password=8Us5blLxYeh77cqWQR9qhgWT3e8BMOu8zokeKB3k
root@controller:/home/ubuntu# 
root@controller:/home/ubuntu# . ceph-swift-user1-openrc
root@controller:/home/ubuntu# swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password} list 
swiftuser1-container1
root@controller:/home/ubuntu# swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password} list swiftuser1-container1
ceph-swiftuser1-container1-object-1.txt
dao.txt
guo.txt
root@controller:/home/ubuntu# 
```

### 把 swift 相关配置加入 openstack 


### 查看 openstack endpoint 

```
root@controller:~# ls
account.ring.gz  admin-openrc  backups  cirros-0.4.0-x86_64-disk.img  container.ring.gz  demo-openrc  object.ring.gz  swift-conf  tmp  tmp2  tmp3
root@controller:~# cat admin-openrc 
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=openstack
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
root@controller:~# . admin-openrc 
root@controller:~# openstack endpoint list
+----------------------------------|-----------|--------------|--------------|---------|-----------|-----------------------------------------------+
| ID                               | Region    | Service Name | Service Type | Enabled | Interface | URL                                           |
+----------------------------------|-----------|--------------|--------------|---------|-----------|-----------------------------------------------+
| 014c4ed5c42042c394d62a1194bf07ce | RegionOne | keystone     | identity     | True    | internal  | http://controller:5000/v3/                    |
| 057cd51c8ace4daa8834d25ae15998f4 | RegionOne | swift        | object-store | True    | admin     | http://controller:8080/v1                     |
| 0b3dc5788cac4f2cb66edc3efacf10c0 | RegionOne | keystone     | identity     | True    | public    | http://controller:5000/v3/                    |
| 19e727c1668c4e8aae4315e13057fbbd | RegionOne | cinderv2     | volumev2     | True    | public    | http://controller:8776/v2/%(project_id)s      |
| 2548787f06524287b6a27d0a562da375 | RegionOne | glance       | image        | True    | internal  | http://controller:9292                        |
| 295cfee33ff04ab7890a84b47e95bc3f | RegionOne | keystone     | identity     | True    | admin     | http://controller:5000/v3/                    |
| 50d3b61b60654a65bda26584b4fe0896 | RegionOne | neutron      | network      | True    | admin     | http://controller:9696                        |
| 51952812b675436899ba118585b2dae2 | RegionOne | swift        | object-store | True    | public    | http://swiftproxy:8080/v1/AUTH_%(project_id)s |
| 55367bdc8db6438da3a4ed82b1a1b04c | RegionOne | glance       | image        | True    | public    | http://controller:9292                        |
| 5b60dcd8e374410f83e675201d76f06f | RegionOne | placement    | placement    | True    | admin     | http://controller:8778                        |
| 76d61f360fc140a58b4258c344ea9ae5 | RegionOne | nova         | compute      | True    | internal  | http://controller:8774/v2.1                   |
| 7eece3971f9f4105b031214666940d54 | RegionOne | placement    | placement    | True    | public    | http://controller:8778                        |
| 8237211a85314e0c914ea34851e324b7 | RegionOne | cinderv2     | volumev2     | True    | admin     | http://controller:8776/v2/%(project_id)s      |
| 87de9f6afbd74792bc80c7d092eb7da6 | RegionOne | swift        | object-store | True    | internal  | http://controller:8080/v1/AUTH_%(project_id)s |
| 99b6673fda7f48bb8b6ae11a87ba0e1b | RegionOne | glance       | image        | True    | admin     | http://controller:9292                        |
| a61cbe1448284dd482725fb3067fa25c | RegionOne | nova         | compute      | True    | public    | http://controller:8774/v2.1                   |
| beccdad7270643cabf8a271258c102c0 | RegionOne | swift        | object-store | True    | admin     | http://swiftproxy:8080/v1                     |
| cfb1df03d1bd42719dd506e0fbee7e5a | RegionOne | cinderv3     | volumev3     | True    | internal  | http://controller:8776/v3/%(project_id)s      |
| cfe6b374e86048c98951d079a3cdaa42 | RegionOne | placement    | placement    | True    | internal  | http://controller:8778                        |
| d3f5808482cb42f386c3741b8e1c1d82 | RegionOne | swift        | object-store | True    | public    | http://controller:8080/v1/AUTH_%(project_id)s |
| d9bb0785c23143ed855d0bb74dfe53bb | RegionOne | cinderv3     | volumev3     | True    | public    | http://controller:8776/v3/%(project_id)s      |
| e1f98d0e79f549a5bc46bcf05705c4b1 | RegionOne | neutron      | network      | True    | public    | http://controller:9696                        |
| e577371ea7e349ffa2a86f7aef2ef35e | RegionOne | swift        | object-store | True    | internal  | http://swiftproxy:8080/v1/AUTH_%(project_id)s |
| f089ebe1cb5040319e942dbc607e9930 | RegionOne | cinderv3     | volumev3     | True    | admin     | http://controller:8776/v3/%(project_id)s      |
| f3350a1d66fa4813a091beef3782b6fd | RegionOne | neutron      | network      | True    | internal  | http://controller:9696                        |
| f5460d92ef8b428b9dc354628acdb289 | RegionOne | cinderv2     | volumev2     | True    | internal  | http://controller:8776/v2/%(project_id)s      |
| fb5c30679ed14e078646041e75e77294 | RegionOne | nova         | compute      | True    | admin     | http://controller:8774/v2.1                   |
+----------------------------------|-----------|--------------|--------------|---------|-----------|-----------------------------------------------+
root@controller:~# openstack endpoint list | grep object-store
| 057cd51c8ace4daa8834d25ae15998f4 | RegionOne | swift        | object-store | True    | admin     | http://controller:8080/v1                     |
| 51952812b675436899ba118585b2dae2 | RegionOne | swift        | object-store | True    | public    | http://swiftproxy:8080/v1/AUTH_%(project_id)s |
| 87de9f6afbd74792bc80c7d092eb7da6 | RegionOne | swift        | object-store | True    | internal  | http://controller:8080/v1/AUTH_%(project_id)s |
| beccdad7270643cabf8a271258c102c0 | RegionOne | swift        | object-store | True    | admin     | http://swiftproxy:8080/v1                     |
| d3f5808482cb42f386c3741b8e1c1d82 | RegionOne | swift        | object-store | True    | public    | http://controller:8080/v1/AUTH_%(project_id)s |
| e577371ea7e349ffa2a86f7aef2ef35e | RegionOne | swift        | object-store | True    | internal  | http://swiftproxy:8080/v1/AUTH_%(project_id)s |
root@controller:~# openstack endpoint show object-store 
More than one endpoint exists with the name 'object-store'.
root@controller:~# 
```

已经有了 object-store , 删除相应的 endpoint

```
root@controller:~# openstack endpoint delete 057cd51c8ace4daa8834d25ae15998f4 
root@controller:~# openstack endpoint delete 51952812b675436899ba118585b2dae2
root@controller:~# openstack endpoint delete 87de9f6afbd74792bc80c7d092eb7da6
root@controller:~# openstack endpoint delete beccdad7270643cabf8a271258c102c0
root@controller:~# openstack endpoint delete d3f5808482cb42f386c3741b8e1c1d82
root@controller:~# openstack endpoint delete e577371ea7e349ffa2a86f7aef2ef35e
```

### 添加 openstack service 

```
root@controller:~# openstack service create --name=swift \
>                          --description="Swift Service" \
>                          object-store
+-------------|----------------------------------+
| Field       | Value                            |
+-------------|----------------------------------+
| description | Swift Service                    |
| enabled     | True                             |
| id          | c428acff9bb746e8947e99f24e8c5dc8 |
| name        | swift                            |
| type        | object-store                     |
+-------------|----------------------------------+
```

### (可跳过)添加 openstack endpoint 出错

```
root@controller:~# vi /etc/hosts 
root@controller:~# openstack endpoint create --region RegionOne \
>      --publicurl   "http://ceph-rgw-client:8080/swift/v1" \
>      --adminurl    "http://ceph-rgw-client:8080/swift/v1" \
>      --internalurl "http://ceph-rgw-client:8080/swift/v1" \
>      swift
usage: openstack endpoint create [-h] [-f {json,shell,table,value,yaml}]
                                 [-c COLUMN] [--max-width <integer>]
                                 [--fit-width] [--print-empty] [--noindent]
                                 [--prefix PREFIX] [--region <region-id>]
                                 [--enable | --disable]
                                 <service> <interface> <url>
openstack endpoint create: error: argument <interface>: invalid choice: u'http://ceph-rgw-client:8080/swift/v1' (choose from 'admin', 'public', 'internal')
root@controller:~# 
```

参数不对，那可能使用方式变了。参考 https://docs.openstack.org/swift/rocky/install/controller-install-ubuntu.html 

```
root@controller:~# openstack endpoint create --region RegionOne \
>   object-store public http://ceph-rgw-client:8080/swift/v1
Multiple service matches found for 'object-store', use an ID to be more specific.
root@controller:~# 
```

加不进，有多个 service 。那应该就是刚刚删除的endpoint的service没删除。

为了不要搞错了，这里再新建立一次。

```
root@controller:~# openstack service create --name=swift --description="Swift Service" object-store
+-------------|----------------------------------+
| Field       | Value                            |
+-------------|----------------------------------+
| description | Swift Service                    |
| enabled     | True                             |
| id          | 356603934f624475b863560b95394ad3 |
| name        | swift                            |
| type        | object-store                     |
+-------------|----------------------------------+
root@controller:~# 
```

查看 service 并删除

```
root@controller:~# openstack service | grep swift
root@controller:~# openstack service list | grep swift
| 356603934f624475b863560b95394ad3 | swift     | object-store |
| a1f85844507d4135b9e77a4c36503552 | swift     | object-store |
| c428acff9bb746e8947e99f24e8c5dc8 | swift     | object-store |
root@controller:~# openstack service delete a1f85844507d4135b9e77a4c36503552
root@controller:~# openstack service delete c428acff9bb746e8947e99f24e8c5dc8
root@controller:~# openstack service list | grep swift
| 356603934f624475b863560b95394ad3 | swift     | object-store |
```

### 添加 openstack endpoint

```
root@controller:~# ping ceph-rgw-client
root@controller:~# openstack endpoint create --region RegionOne   object-store public http://ceph-rgw-client:8080/swift/v1
+--------------|--------------------------------------+
| Field        | Value                                |
+--------------|--------------------------------------+
| enabled      | True                                 |
| id           | d2cb83a1aab9467fa9d6d2ede12f9eb7     |
| interface    | public                               |
| region       | RegionOne                            |
| region_id    | RegionOne                            |
| service_id   | 356603934f624475b863560b95394ad3     |
| service_name | swift                                |
| service_type | object-store                         |
| url          | http://ceph-rgw-client:8080/swift/v1 |
+--------------|--------------------------------------+
root@controller:~# openstack endpoint create --region RegionOne \
>   object-store internal http://ceph-rgw-client:8080/swift/v1
+--------------|--------------------------------------+
| Field        | Value                                |
+--------------|--------------------------------------+
| enabled      | True                                 |
| id           | 47fa3d7baab34bf4b333b4fbaf269f08     |
| interface    | internal                             |
| region       | RegionOne                            |
| region_id    | RegionOne                            |
| service_id   | 356603934f624475b863560b95394ad3     |
| service_name | swift                                |
| service_type | object-store                         |
| url          | http://ceph-rgw-client:8080/swift/v1 |
+--------------|--------------------------------------+
root@controller:~# openstack endpoint create --region RegionOne \
>   object-store admin http://ceph-rgw-client:8080/swift/v1
+--------------|--------------------------------------+
| Field        | Value                                |
+--------------|--------------------------------------+
| enabled      | True                                 |
| id           | 96b6fe4b7e404fa4868d06c3d402ba8f     |
| interface    | admin                                |
| region       | RegionOne                            |
| region_id    | RegionOne                            |
| service_id   | 356603934f624475b863560b95394ad3     |
| service_name | swift                                |
| service_type | object-store                         |
| url          | http://ceph-rgw-client:8080/swift/v1 |
+--------------|--------------------------------------+
root@controller:~# openstack endpoint list
+----------------------------------|-----------|--------------|--------------|---------|-----------|------------------------------------------+
| ID                               | Region    | Service Name | Service Type | Enabled | Interface | URL                                      |
+----------------------------------|-----------|--------------|--------------|---------|-----------|------------------------------------------+
| 014c4ed5c42042c394d62a1194bf07ce | RegionOne | keystone     | identity     | True    | internal  | http://controller:5000/v3/               |
| 0b3dc5788cac4f2cb66edc3efacf10c0 | RegionOne | keystone     | identity     | True    | public    | http://controller:5000/v3/               |
| 19e727c1668c4e8aae4315e13057fbbd | RegionOne | cinderv2     | volumev2     | True    | public    | http://controller:8776/v2/%(project_id)s |
| 2548787f06524287b6a27d0a562da375 | RegionOne | glance       | image        | True    | internal  | http://controller:9292                   |
| 295cfee33ff04ab7890a84b47e95bc3f | RegionOne | keystone     | identity     | True    | admin     | http://controller:5000/v3/               |
| 47fa3d7baab34bf4b333b4fbaf269f08 | RegionOne | swift        | object-store | True    | internal  | http://ceph-rgw-client:8080/swift/v1     |
| 50d3b61b60654a65bda26584b4fe0896 | RegionOne | neutron      | network      | True    | admin     | http://controller:9696                   |
| 55367bdc8db6438da3a4ed82b1a1b04c | RegionOne | glance       | image        | True    | public    | http://controller:9292                   |
| 5b60dcd8e374410f83e675201d76f06f | RegionOne | placement    | placement    | True    | admin     | http://controller:8778                   |
| 76d61f360fc140a58b4258c344ea9ae5 | RegionOne | nova         | compute      | True    | internal  | http://controller:8774/v2.1              |
| 7eece3971f9f4105b031214666940d54 | RegionOne | placement    | placement    | True    | public    | http://controller:8778                   |
| 8237211a85314e0c914ea34851e324b7 | RegionOne | cinderv2     | volumev2     | True    | admin     | http://controller:8776/v2/%(project_id)s |
| 96b6fe4b7e404fa4868d06c3d402ba8f | RegionOne | swift        | object-store | True    | admin     | http://ceph-rgw-client:8080/swift/v1     |
| 99b6673fda7f48bb8b6ae11a87ba0e1b | RegionOne | glance       | image        | True    | admin     | http://controller:9292                   |
| a61cbe1448284dd482725fb3067fa25c | RegionOne | nova         | compute      | True    | public    | http://controller:8774/v2.1              |
| cfb1df03d1bd42719dd506e0fbee7e5a | RegionOne | cinderv3     | volumev3     | True    | internal  | http://controller:8776/v3/%(project_id)s |
| cfe6b374e86048c98951d079a3cdaa42 | RegionOne | placement    | placement    | True    | internal  | http://controller:8778                   |
| d2cb83a1aab9467fa9d6d2ede12f9eb7 | RegionOne | swift        | object-store | True    | public    | http://ceph-rgw-client:8080/swift/v1     |
| d9bb0785c23143ed855d0bb74dfe53bb | RegionOne | cinderv3     | volumev3     | True    | public    | http://controller:8776/v3/%(project_id)s |
| e1f98d0e79f549a5bc46bcf05705c4b1 | RegionOne | neutron      | network      | True    | public    | http://controller:9696                   |
| f089ebe1cb5040319e942dbc607e9930 | RegionOne | cinderv3     | volumev3     | True    | admin     | http://controller:8776/v3/%(project_id)s |
| f3350a1d66fa4813a091beef3782b6fd | RegionOne | neutron      | network      | True    | internal  | http://controller:9696                   |
| f5460d92ef8b428b9dc354628acdb289 | RegionOne | cinderv2     | volumev2     | True    | internal  | http://controller:8776/v2/%(project_id)s |
| fb5c30679ed14e078646041e75e77294 | RegionOne | nova         | compute      | True    | admin     | http://controller:8774/v2.1              |
+----------------------------------|-----------|--------------|--------------|---------|-----------|------------------------------------------+
root@controller:~# openstack endpoint show object-store 
More than one endpoint exists with the name 'object-store'.
root@controller:~# openstack endpoint list |grep -i object 
| 47fa3d7baab34bf4b333b4fbaf269f08 | RegionOne | swift        | object-store | True    | internal  | http://ceph-rgw-client:8080/swift/v1     |
| 96b6fe4b7e404fa4868d06c3d402ba8f | RegionOne | swift        | object-store | True    | admin     | http://ceph-rgw-client:8080/swift/v1     |
| d2cb83a1aab9467fa9d6d2ede12f9eb7 | RegionOne | swift        | object-store | True    | public    | http://ceph-rgw-client:8080/swift/v1     |
root@controller:~# 
```

好了。这里已经加好了！~

### 修改 ceph.conf 配置

因为这里我们不确定写的配置能一次性成功，所以，先在 ceph rgw client 节点直接修改 /etc/ceph/ceph.conf 文件，并重启。

可参考[这里](/docs/faq/rgw-conf-change.html)

#### 修改配置

```
root@ceph-client:/home/admin# mkdir /var/ceph/nss -p
root@ceph-client:/home/admin# vi /etc/ceph/ceph.conf 
root@ceph-client:/home/admin# cat /etc/ceph/ceph.conf 
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

rgw_keystone_api_version = v3
rgw_keystone_url = http://controller:5000
rgw_keystone_admin_token = openstack
#rgw_keystone_admin_token_path = {path to keystone admin token} #preferred
rgw_keystone_accepted_roles =  _member_,admin
rgw_keystone_token_cache_size = 200
rgw_keystone_revocation_interval = 60
rgw_keystone_admin_domain = default
rgw_keystone_admin_project = default
rgw_keystone_implicit_tenants = true
rgw_keystone_verify_ssl = false
nss_db_path = /var/ceph/nss
root@ceph-client:/home/admin# 
```

#### 重启 rgw 服务

```
root@ceph-client:/home/admin# systemctl restart ceph-radosgw@rgw.ceph-client
root@ceph-client:/home/admin# systemctl status ceph-radosgw@rgw.ceph-client
● ceph-radosgw@rgw.ceph-client.service - Ceph rados gateway
   Loaded: loaded (/lib/systemd/system/ceph-radosgw@.service; enabled; vendor preset: enabled)
   Active: active (running) since 一 2019-03-11 18:02:41 CST; 4ms ago
 Main PID: 83757 ((radosgw))
   CGroup: /system.slice/system-ceph\x2dradosgw.slice/ceph-radosgw@rgw.ceph-client.service
           └─83757 (radosgw)        

3月 11 18:02:41 ceph-client systemd[1]: Started Ceph rados gateway.
root@ceph-client:/home/admin# systemctl status ceph-radosgw@rgw.ceph-client
● ceph-radosgw@rgw.ceph-client.service - Ceph rados gateway
   Loaded: loaded (/lib/systemd/system/ceph-radosgw@.service; enabled; vendor preset: enabled)
   Active: failed (Result: start-limit-hit) since 一 2019-03-11 18:46:10 CST; 2s ago
  Process: 84958 ExecStart=/usr/bin/radosgw -f --cluster ${CLUSTER} --name client.%i --setuser ceph --setgroup ceph (code=dumped, signal=ABRT)
 Main PID: 84958 (code=dumped, signal=ABRT)

3月 11 18:46:09 ceph-client systemd[1]: ceph-radosgw@rgw.ceph-client.service: Main process exited, code=dumped, status=6/ABRT
3月 11 18:46:09 ceph-client systemd[1]: ceph-radosgw@rgw.ceph-client.service: Unit entered failed state.
3月 11 18:46:09 ceph-client systemd[1]: ceph-radosgw@rgw.ceph-client.service: Failed with result 'core-dump'.
3月 11 18:46:10 ceph-client systemd[1]: ceph-radosgw@rgw.ceph-client.service: Service hold-off time over, scheduling restart.
3月 11 18:46:10 ceph-client systemd[1]: Stopped Ceph rados gateway.
3月 11 18:46:10 ceph-client systemd[1]: ceph-radosgw@rgw.ceph-client.service: Start request repeated too quickly.
3月 11 18:46:10 ceph-client systemd[1]: Failed to start Ceph rados gateway.
3月 11 18:46:10 ceph-client systemd[1]: ceph-radosgw@rgw.ceph-client.service: Unit entered failed state.
3月 11 18:46:10 ceph-client systemd[1]: ceph-radosgw@rgw.ceph-client.service: Failed with result 'start-limit-hit'.
root@ceph-client:/home/admin# 
```

报错了，啥问题



### swift client 验证

```
root@controller:/home/ubuntu# ls
ceph-swiftuser1-container1-object-1.txt  ceph-swiftuser1-container1-object-1.txt.bak  ceph-swift-user1-openrc  ceph-swift-user1-openrc.password  hello.txt
root@controller:/home/ubuntu# . ceph-swift-user1-openrc
root@controller:/home/ubuntu# swift list -v 
Account GET failed: http://ceph-rgw-client:8080/swift/v1?format=json 401 Unauthorized  [first 60 chars of response] {"Code":"AccessDenied","RequestId":"tx00000000000000000001f-
Failed Transaction ID: tx00000000000000000001f-005c862b5a-5e2c-default
root@controller:/home/ubuntu# cat ceph-swift-user1-openrc
username=swiftuser1
subusername=swiftsubuser1
displayname=swiftuser1displayname
password=8Us5blLxYeh77cqWQR9qhgWT3e8BMOu8zokeKB3k
root@controller:/home/ubuntu# . ceph-swift-user1-openrc
root@controller:/home/ubuntu# swift list -v 
Account GET failed: http://ceph-rgw-client:8080/swift/v1?format=json 401 Unauthorized  [first 60 chars of response] {"Code":"AccessDenied","RequestId":"tx000000000000000000021-
Failed Transaction ID: tx000000000000000000021-005c862e4a-5e2c-default
root@controller:/home/ubuntu# ls
ceph-swiftuser1-container1-object-1.txt  ceph-swiftuser1-container1-object-1.txt.bak  ceph-swift-user1-openrc  ceph-swift-user1-openrc.password  hello.txt
root@controller:/home/ubuntu# . ceph-swift-user1-openrc
root@controller:/home/ubuntu# . ceph-swift-user1-openrc
root@controller:/home/ubuntu# swift list -v 
HTTPConnectionPool(host='ceph-rgw-client', port=8080): Max retries exceeded with url: /swift/v1?format=json (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7fb1ca95f490>: Failed to establish a new connection: [Errno 111] Connection refused',))
root@controller:/home/ubuntu# telnet 192.168.0.134 8080
Trying 192.168.0.134...
telnet: Unable to connect to remote host: Connection refused
root@controller:/home/ubuntu# 
```



## ref
- http://docs.ceph.com/docs/master/radosgw/keystone/
- http://docs.ceph.com/docs/jewel/radosgw/keystone/
- https://stackoverflow.com/questions/39461803/openstack-swift-with-ceph-backend-radosgw
- https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/2/html-single/using_keystone_to_authenticate_ceph_object_gateway_users/index
