### 列出所有用户

- radosgw-admin user list
- radosgw-admin metadata list user

```
admin@ceph-admin:~$ radosgw-admin user list
[
    "1c0130df9066484791e2807ca65ce8c2$1c0130df9066484791e2807ca65ce8c2",
    "5ed7a713486547ae80f6c05d0da42a54$5ed7a713486547ae80f6c05d0da42a54",
    "03b11e496f45412f909fd5e58221d6c8$03b11e496f45412f909fd5e58221d6c8",
    "6a5bca70568e4b6fb7cb5f1447f160c2$6a5bca70568e4b6fb7cb5f1447f160c2",
    "a05c6b0541c44f49955ed1748b3fcfdb$a05c6b0541c44f49955ed1748b3fcfdb",
    "23c12f3f66fb40ccb565d58cf3402c54$23c12f3f66fb40ccb565d58cf3402c54",
    "41260b903ce44ab7b5c28e13132ce816$41260b903ce44ab7b5c28e13132ce816",
    "d1f1fb325ea84c58b4820dae96dcb721$d1f1fb325ea84c58b4820dae96dcb721",
    "a9e83dfd0fa14ec4b2e47842f2d5797a$a9e83dfd0fa14ec4b2e47842f2d5797a",
    "f04ec0abf3d1460dad82608bb03af589$f04ec0abf3d1460dad82608bb03af589",
    "swiftuser1",
    "4553832d3bf14c9a853b89c2992235b9$4553832d3bf14c9a853b89c2992235b9",
    "7d6eaa90d74a4f239963933c3a744df3$7d6eaa90d74a4f239963933c3a744df3",
    "446002de5b8c41e69685351a92d26a2d$446002de5b8c41e69685351a92d26a2d"
]
admin@ceph-admin:~$ 
```


### 获取用户信息


```
root@ceph-client:/etc/ceph# radosgw-admin metadata list user
[
    "swiftuser1"
]
root@ceph-client:/etc/ceph# radosgw-admin user info --uid="swiftuser1"
{
    "user_id": "swiftuser1",
    "display_name": "swiftuser1displayname",
    "email": "",
    "suspended": 0,
    "max_buckets": 1000,
    "auid": 0,
    "subusers": [
        {
            "id": "swiftuser1:swiftsubuser1",
            "permissions": "full-control"
        }
    ],
    "keys": [],
    "swift_keys": [
        {
            "user": "swiftuser1:swiftsubuser1",
            "secret_key": "8Us5blLxYeh77cqWQR9qhgWT3e8BMOu8zokeKB3k"
        }
    ],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "temp_url_keys": [],
    "type": "rgw",
    "mfa_ids": []
}

root@ceph-client:/etc/ceph# 
```

### 建个 openrc 文件

```
root@controller:/home/ubuntu# cat ceph-swift-user1-openrc
username=swiftuser1
subusername=swiftsubuser1
displayname=swiftuser1displayname
password=G8jTHyahPudXOQ0Dv3oDn3wq9fMKnbcc9e4IumZO
root@controller:/home/ubuntu# . ceph-swift-user1-openrc
```

### 获取上面的password

要获取上面的password等字段，则通过：

- `radosgw-admin user info --uid="swiftuser1"` 

### 生成并替换password

- `sudo radosgw-admin key create --subuser="${username}:${subusername}" --key-type=swift --gen-secret` 生成并替换



### rgw

```
admin@ceph-admin:~$ radosgw-admin user info --uid "swiftuser1"
{
    "user_id": "swiftuser1",
    "display_name": "swiftuser1displayname",
    "email": "",
    "suspended": 0,
    "max_buckets": 1000,
    "auid": 0,
    "subusers": [
        {
            "id": "swiftuser1:swiftsubuser1",
            "permissions": "full-control"
        }
    ],
    "keys": [],
    "swift_keys": [
        {
            "user": "swiftuser1:swiftsubuser1",
            "secret_key": "G8jTHyahPudXOQ0Dv3oDn3wq9fMKnbcc9e4IumZO"
        }
    ],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "temp_url_keys": [
        {
            "key": 0,
            "val": "hello"
        }
    ],
    "type": "rgw",
    "mfa_ids": []
}

admin@ceph-admin:~$ 
```

### keystone

keystone 用户，user_id 使用的是openstack project_id 名（不是openstack user_id）, 即：radosgw下的 user_id 为 `${project_id}$${project_id}`。

比如下面这个示例：当时是通过keystone生成的用户demo, 其 project_id 是 `a9e83dfd0fa14ec4b2e47842f2d5797a` ，则  所以下面 

- "type": "keystone", 
- "user_id": "a9e83dfd0fa14ec4b2e47842f2d5797a$a9e83dfd0fa14ec4b2e47842f2d5797a",
- temp_url_keys 也是明文展示在这里

**注意：在写参数的时候，要在 `$` 号前加 `\` 转义**

```
admin@ceph-admin:~$ radosgw-admin user info --uid "a9e83dfd0fa14ec4b2e47842f2d5797a\$a9e83dfd0fa14ec4b2e47842f2d5797a"
{
    "user_id": "a9e83dfd0fa14ec4b2e47842f2d5797a$a9e83dfd0fa14ec4b2e47842f2d5797a",
    "display_name": "demo",
    "email": "",
    "suspended": 0,
    "max_buckets": 1000,
    "auid": 0,
    "subusers": [],
    "keys": [],
    "swift_keys": [],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "temp_url_keys": [
        {
            "key": 0,
            "val": "demohelloworld"
        }
    ],
    "type": "keystone",
    "mfa_ids": []
}

admin@ceph-admin:~$ 
```

## ref
- https://keithtenzer.com/2017/03/30/openstack-swift-integration-with-ceph/
