


修改了 ceph.conf 中的关于 ceph rgw 部分，如何让配置生效

## env



## step 

### 配置文件更新

在 ceph-admin 节点

```
admin@ceph-admin:~/test-cluster$ ceph-deploy --overwrite-conf config push ceph-admin ceph-client 
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/admin/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /usr/bin/ceph-deploy --overwrite-conf config push ceph-admin ceph-client
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  overwrite_conf                : True
[ceph_deploy.cli][INFO  ]  subcommand                    : push
[ceph_deploy.cli][INFO  ]  quiet                         : False
[ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x7f7fdd703200>
[ceph_deploy.cli][INFO  ]  cluster                       : ceph
[ceph_deploy.cli][INFO  ]  client                        : ['ceph-admin', 'ceph-client']
[ceph_deploy.cli][INFO  ]  func                          : <function config at 0x7f7fddb51f50>
[ceph_deploy.cli][INFO  ]  ceph_conf                     : None
[ceph_deploy.cli][INFO  ]  default_release               : False
[ceph_deploy.config][DEBUG ] Pushing config to ceph-admin
[ceph-admin][DEBUG ] connection detected need for sudo
[ceph-admin][DEBUG ] connected to host: ceph-admin 
[ceph-admin][DEBUG ] detect platform information from remote host
[ceph-admin][DEBUG ] detect machine type
[ceph-admin][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
[ceph_deploy.config][DEBUG ] Pushing config to ceph-client
[ceph-client][DEBUG ] connection detected need for sudo
[ceph-client][DEBUG ] connected to host: ceph-client 
[ceph-client][DEBUG ] detect platform information from remote host
[ceph-client][DEBUG ] detect machine type
[ceph-client][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
admin@ceph-admin:~/test-cluster$ 
```

### 重启 ceph rgw 节点

在 ceph-client 节点

```
root@ceph-client:/home/admin# systemctl restart ceph-radosgw@rgw.ceph-client
root@ceph-client:/home/admin# systemctl status ceph-radosgw@rgw.ceph-client
● ceph-radosgw@rgw.ceph-client.service - Ceph rados gateway
   Loaded: loaded (/lib/systemd/system/ceph-radosgw@.service; enabled; vendor preset: enabled)
   Active: active (running) since 一 2019-03-11 17:55:58 CST; 1s ago
 Main PID: 82967 (radosgw)
   CGroup: /system.slice/system-ceph\x2dradosgw.slice/ceph-radosgw@rgw.ceph-client.service
           └─82967 /usr/bin/radosgw -f --cluster ceph --name client.rgw.ceph-client --setuser ceph --setgroup ceph

3月 11 17:55:58 ceph-client systemd[1]: Started Ceph rados gateway.
root@ceph-client:/home/admin# 
```

## ref
- https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/2/html-single/using_keystone_to_authenticate_ceph_object_gateway_users/index
