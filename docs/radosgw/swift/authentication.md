
使用RADOSGW提供ceph的Swift接口

## env

基于 之前 add-an-rgw-instance-with-quick-install ，因此有：

- ceph rgw 的 http 接口是： http://ceph-client:8080/
- ceph-client ip 是： 192.168.0.134 ，以下只用此IP表示 ceph rgw 接口。

swift client : 192.168.0.51, 此上必须有 swift 命令

## step

### ceph-admin 创建 用户

最好此用户仅供 swift 使用

```
root@ceph-admin:~# cat ceph-swift-user1-openrc 
username=swiftuser1
subusername=swiftsubuser1
displayname=swiftuser1displayname
password=swiftuser1password
root@ceph-admin:~# 
root@ceph-admin:~# . ceph-swift-user1-openrc 
root@ceph-admin:~# sudo radosgw-admin user create --subuser="${username}:${subusername}" --uid="${username}" --display-name="${displayname}" --key-type=swift --secret="${password}" --access=full
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
            "secret_key": "swiftuser1password"
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

### swift-client


#### 是否与 ceph rgw 互通


```
ubuntu@controller:~$ telnet 192.168.0.134 8080
Trying 192.168.0.134...
Connected to 192.168.0.134.
Escape character is '^]'.
^]
telnet> quit
ubuntu@controller:~$ 
```

#### 有 swift 命令

```
ubuntu@controller:~$ swift --version
python-swiftclient 3.5.0
ubuntu@controller:~$ 
```

#### 加载变量值
```
ubuntu@controller:~$ cat ceph-swift-user1-openrc 
username=swiftuser1
subusername=swiftsubuser1
displayname=swiftuser1displayname
password=swiftuser1password
ubuntu@controller:~$ . ceph-swift-user1-openrc 
```

#### 查看swift里所有的容器

```
ubuntu@controller:~$ swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password}  list 
```

#### 创建容器

```
ubuntu@controller:~$ swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password} post swiftuser1-container1
ubuntu@controller:~$ swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password} list 
swiftuser1-container1
ubuntu@controller:~$ 
```

#### 上传文件

里面没有文件。现在可以上传文件

```
ubuntu@controller:~$ echo "Hello World" > ceph-swiftuser1-container1-object-1.txt
ubuntu@controller:~$ swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password}  upload swiftuser1-container1 ceph-swiftuser1-container1-object-1.txt
ceph-swiftuser1-container1-object-1.txt
ubuntu@controller:~$ swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password} list swiftuser1-container1
ceph-swiftuser1-container1-object-1.txt
```

#### 下载文件

```
ubuntu@controller:~$ mv ceph-swiftuser1-container1-object-1.txt ceph-swiftuser1-container1-object-1.txt.bak
ubuntu@controller:~$ ls
ceph-swiftuser1-container1-object-1.txt.bak  ceph-swift-user1-openrc  hello.txt
ubuntu@controller:~$ swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password}  download  swiftuser1-container1 ceph-swiftuser1-container1-object-1.txt
ceph-swiftuser1-container1-object-1.txt [auth 0.064s, headers 0.087s, total 0.111s, 0.000 MB/s]
ubuntu@controller:~$ ls
ceph-swiftuser1-container1-object-1.txt  ceph-swiftuser1-container1-object-1.txt.bak  ceph-swift-user1-openrc  hello.txt
ubuntu@controller:~$ cat ceph-swiftuser1-container1-object-1.txt
Hello World
ubuntu@controller:~$ 
```

## ref
- http://docs.ceph.com/docs/mimic/radosgw/swift/auth/
- http://qinghua.github.io/ceph-radosgw/