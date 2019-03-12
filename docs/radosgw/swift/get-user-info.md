
获取用户信息


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

## ref
- https://keithtenzer.com/2017/03/30/openstack-swift-integration-with-ceph/
