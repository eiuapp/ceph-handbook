
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


## ref
- https://keithtenzer.com/2017/03/30/openstack-swift-integration-with-ceph/
