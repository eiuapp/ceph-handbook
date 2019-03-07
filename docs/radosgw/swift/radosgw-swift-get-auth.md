
radosgw swift auth 之 get auth

## env

基于 之前 add-an-rgw-instance-with-quick-install ，因此有：

- ceph rgw 的 http 接口是： http://ceph-client:8080/
- ceph-client ip 是： 192.168.0.134 ，以下只用此IP表示 ceph rgw 接口。

swift client : 192.168.0.51, 此上必须有 swift 命令

本文基于 authentication.md 之上。

## step

### ceph-admin 节点

上文设置了 专用于 swift 的帐户密码
现在生成 swift secret key 

```
root@ceph-admin:~# cat ceph-swift-user1-openrc 
username=swiftuser1
subusername=swiftsubuser1
displayname=swiftuser1displayname
password=swiftuser1password
root@ceph-admin:~# . ceph-swift-user1-openrc 
root@ceph-admin:~# sudo radosgw-admin key create --subuser="${username}:${subusername}" --key-type=swift --gen-secret
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

root@ceph-admin:~# 
```


主要看输出的 swift_keys 部分。


### swift client 节点

#### 原来的 password 已失效


**注意** 因为这里已经生成了 新的secret_key, 则原来的 password 已经失效了。

```
ubuntu@controller:~$ cat ceph-swift-user1-openrc 
username=swiftuser1
subusername=swiftsubuser1
displayname=swiftuser1displayname
password=swiftuser1password
ubuntu@controller:~$ . ceph-swift-user1-openrc 
ubuntu@controller:~$ swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password} list swiftuser1-container1
Auth GET failed: http://192.168.0.134:8080/auth/v1.0 401 Unauthorized  [first 60 chars of response] {"Code":"AccessDenied","RequestId":"tx000000000000000000016-
Failed Transaction ID: tx000000000000000000016-005c80e8e9-5e2c-default
ubuntu@controller:~$ cp ceph-swift-user1-openrc ceph-swift-user1-openrc.password
ubuntu@controller:~$ vi ceph-swift-user1-openrc
ubuntu@controller:~$ cat ceph-swift-user1-openrc
username=swiftuser1
subusername=swiftsubuser1
displayname=swiftuser1displayname
password=8Us5blLxYeh77cqWQR9qhgWT3e8BMOu8zokeKB3k
ubuntu@controller:~$ . ceph-swift-user1-openrc
ubuntu@controller:~$ swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password} list swiftuser1-container1
ceph-swiftuser1-container1-object-1.txt
dao.txt
guo.txt
ubuntu@controller:~$ 
```

#### 正确输出X-Auth-Token

```
ubuntu@controller:~$ curl -X GET -H "X-Auth-User:swiftuser1:swiftsubuser1" -H "X-Auth-Key:8Us5blLxYeh77cqWQR9qhgWT3e8BMOu8zokeKB3k"  http://192.168.0.134:8080/auth 
ubuntu@controller:~$ curl -X GET -H "X-Auth-User:swiftuser1:swiftsubuser1" -H "X-Auth-Key:8Us5blLxYeh77cqWQR9qhgWT3e8BMOu8zokeKB3k"  http://192.168.0.134:8080/auth -v
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying 192.168.0.134...
* Connected to 192.168.0.134 (192.168.0.134) port 8080 (#0)
> GET /auth HTTP/1.1
> Host: 192.168.0.134:8080
> User-Agent: curl/7.47.0
> Accept: */*
> X-Auth-User:swiftuser1:swiftsubuser1
> X-Auth-Key:8Us5blLxYeh77cqWQR9qhgWT3e8BMOu8zokeKB3k
> 
< HTTP/1.1 204 No Content
< X-Storage-Url: http://192.168.0.134:8080/swift/v1
< X-Storage-Token: AUTH_rgwtk18000000737769667475736572313a73776966747375627573657231e5dfabacea7bf6ba822c825c99cb4f193b8d565ada00ebc8cad494ef2f7d956f902be17b
< X-Auth-Token: AUTH_rgwtk18000000737769667475736572313a73776966747375627573657231e5dfabacea7bf6ba822c825c99cb4f193b8d565ada00ebc8cad494ef2f7d956f902be17b
< X-Trans-Id: tx00000000000000000000d-005c80db02-5e2c-default
< X-Openstack-Request-Id: tx00000000000000000000d-005c80db02-5e2c-default
< Content-Type: application/json; charset=utf-8
< Date: Thu, 07 Mar 2019 08:49:06 GMT
< 
* Connection #0 to host 192.168.0.134 left intact
ubuntu@controller:~$ 
```

#### （可忽略）参数错误的效果

用户密码均不对

```
ubuntu@controller:~$ curl -X GET -H "X-Auth-User:swiftuser1" -H "X-Auth-Key:swiftuser1password"  http://192.168.0.134:8080/auth -v
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying 192.168.0.134...
* Connected to 192.168.0.134 (192.168.0.134) port 8080 (#0)
> GET /auth HTTP/1.1
> Host: 192.168.0.134:8080
> User-Agent: curl/7.47.0
> Accept: */*
> X-Auth-User:swiftuser1
> X-Auth-Key:swiftuser1password
> 
< HTTP/1.1 403 Forbidden
< Content-Length: 117
< X-Trans-Id: tx000000000000000000008-005c80d941-5e2c-default
< X-Openstack-Request-Id: tx000000000000000000008-005c80d941-5e2c-default
< Accept-Ranges: bytes
< Content-Type: application/json; charset=utf-8
< Date: Thu, 07 Mar 2019 08:41:37 GMT
< 
* Connection #0 to host 192.168.0.134 left intact
{"Code":"AccessDenied","RequestId":"tx000000000000000000008-005c80d941-5e2c-default","HostId":"5e2c-default-default"}ubuntu@controller:~$ ^C
ubuntu@controller:~$ 
```

用户不对，密码对

```
ubuntu@controller:~$ curl -X GET -H "X-Auth-User:swiftuser1" -H "X-Auth-Key:8Us5blLxYeh77cqWQR9qhgWT3e8BMOu8zokeKB3k"  http://192.168.0.134:8080/auth -v
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying 192.168.0.134...
* Connected to 192.168.0.134 (192.168.0.134) port 8080 (#0)
> GET /auth HTTP/1.1
> Host: 192.168.0.134:8080
> User-Agent: curl/7.47.0
> Accept: */*
> X-Auth-User:swiftuser1
> X-Auth-Key:8Us5blLxYeh77cqWQR9qhgWT3e8BMOu8zokeKB3k
> 
< HTTP/1.1 403 Forbidden
< Content-Length: 117
< X-Trans-Id: tx000000000000000000009-005c80daca-5e2c-default
< X-Openstack-Request-Id: tx000000000000000000009-005c80daca-5e2c-default
< Accept-Ranges: bytes
< Content-Type: application/json; charset=utf-8
< Date: Thu, 07 Mar 2019 08:48:10 GMT
< 
* Connection #0 to host 192.168.0.134 left intact
{"Code":"AccessDenied","RequestId":"tx000000000000000000009-005c80daca-5e2c-default","HostId":"5e2c-default-default"}ubuntu@controller:~$ 
ubuntu@controller:~$ 
```
## ref
- https://blog.csdn.net/u012359453/article/details/79978574
