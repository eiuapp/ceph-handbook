
当用户通过keystone认证时，设置并生成tmp url

**注意：本文没有操作成功。也就是说，本文只是记录，没有给出解决方案。**

## step

tmp_url.py 内容如下

```python
print("拿 token ----------------------------------------------")
import requests
import json

URL = 'http://192.168.0.51:5000/v3/auth/tokens'
body =  {
            "auth": {
                "identity": {
                    "methods": [
                        "password"
                    ],
                    "password": {
                        "user": {
                            "name": "admin",
                            "domain": {
                                "name": "default"
                            },
                            "password": "openstack"
                        }
                    }
                },
                "scope": {
                    "project": {
                        "domain": {
                            "name": "default"
                        },
                        "name": "admin"
                    }
                }
            }
        }
body = json.dumps(body)
headers = {'Content-Type':'application/json'}
res = requests.post(URL,data=body,headers=headers)
token =res.headers['X-Subject-Token']
project_id = json.loads(res.content)["token"]["project"]["id"]
print(res.headers['X-Subject-Token'])

print("上传一个文件，先准备个文件哟。如果已经有文件，此步可忽略------------------------------------------------")
with open('abc.txt') as f:
    data = f.read()
URL = 'http://192.168.0.134:8080/swift/v1/123/abc.txt'
headers = {'X-Auth-Token':token}
res = requests.put(URL,headers=headers,data=data)
print(res.text)
print(res.headers)
print(res.status_code)


print("设置Temp URL------------------------------------------------")
import hmac
from hashlib import sha1
from time import time


URL = 'http://192.168.0.134:8080/swift/v1/'
headers = {'X-Auth-Token':token, 'X-Account-Meta-Temp-URL-Key':"hello"}
res = requests.post(URL,headers=headers)
print(res.text)
print(res.headers)
print(res.status_code)

print("查看有没有设置成功------------------------------------------------")

URL = 'http://192.168.0.134:8080/swift/v1/'
headers = {'X-Auth-Token':token, 'X-Account-Meta-Temp-URL-Key':"hello"}
res = requests.get(URL,headers=headers)
print(res.text)
print(res.headers)
print(res.status_code)


print("生成 tmp url------------------------------------------------")
import hmac
from hashlib import sha1
from time import time

method = 'GET'
host = 'http://192.168.0.134:8080/swift'
duration_in_seconds = 300  # Duration for which the url is valid
expires = int(time() + duration_in_seconds)
path = '/v1/123/abc.txt'
key = 'hello'
hmac_body = '%s\n%s\n%s' % (method, expires, path)
key = bytes(key , 'utf-8')
hmac_body = bytes(hmac_body, 'utf-8')
sig = hmac.new(key, hmac_body, sha1).hexdigest()
rest_uri = "{host}{path}?temp_url_sig={sig}&temp_url_expires={expires}".format(
             host=host, path=path, sig=sig, expires=expires)

print(rest_uri)
```

### 生成tmp url部分如下：

```python
method = 'GET'
host = 'http://192.168.0.134:8080/swift'
duration_in_seconds = 300  # Duration for which the url is valid
expires = int(time() + duration_in_seconds)
path = '/v1/123/abc.txt'
key = 'hello'
hmac_body = '%s\n%s\n%s' % (method, expires, path)
key = bytes(key , 'utf-8')
hmac_body = bytes(hmac_body, 'utf-8')
sig = hmac.new(key, hmac_body, sha1).hexdigest()
rest_uri = "{host}{path}?temp_url_sig={sig}&temp_url_expires={expires}".format(
             host=host, path=path, sig=sig, expires=expires)

print(rest_uri)
```

python tmp_url.py 

```
------------------------------------------------
Traceback (most recent call last):
  File "tmp_url.py", line 118, in <module>
    sig = hmac.new(key, hmac_body, sha1).hexdigest()
  File "/root/.pyenv/versions/3.7.2/lib/python3.7/hmac.py", line 153, in new
    return HMAC(key, msg, digestmod)
  File "/root/.pyenv/versions/3.7.2/lib/python3.7/hmac.py", line 49, in __init__
    raise TypeError("key: expected bytes or bytearray, but got %r" % type(key).__name__)
TypeError: key: expected bytes or bytearray, but got 'str'
```

参考 https://stackoverflow.com/questions/31848293/python3-and-hmac-how-to-handle-string-not-being-binary

知道，要将 key, hmac_body转成bytes

```
key = bytes(key , 'utf-8')
hmac_body = bytes(hmac_body, 'utf-8')
```

### 运行后，得到 

http://192.168.0.134:8080/swift/v1/123/abc.txt?temp_url_sig=8acedc3134554c69de5d3fb42cbd9981dc9a16e4&temp_url_expires=1554286239

但是，放到 chrome 中：

显示 

AccessDenied

什么情况？

### update your Keystone endpoint


#### 查一下

```
root@controller:/home/ubuntu# . admin-openrc 
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


#### 方向

根据 http://docs.ceph.com/docs/master/radosgw/swift/tempurl/ 的提示的   update your Keystone endpoint to the URL suffix /v1/AUTH_%(tenant_id)s (instead of just /v1). 操作

我们知道，在 keystone v3 中已经没有 tenant_id 了，替换成 project_id 。

也就是说，我们要把 swift 的 endpoint 由： http://ceph-rgw-client:8080/swift/v1 替换成 http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s


#### 更新endpoint有2种方式：

- 数据库方式

database: keystone
table: endpoint


- 命令行方式修改参考

https://docs.openstack.org/python-openstackclient/pike/cli/command-objects/endpoint.html

因为我们不知道keystone对应的mysql安装在哪里（也不需要知道），所以，我们选择命令行方式


#### 操作

<!-- hello
openstack endpoint set --url "http://ceph-rgw-client:8080/swift/v1" 4fa93f815d64428697bcc41e56f38d94 
openstack endpoint set --url "http://ceph-rgw-client:8080/swift/v1" 5be5403aecb9418aa73288b0f56e1315 
openstack endpoint set --url "http://ceph-rgw-client:8080/swift/v1" b7df0ccdceba42ce8272d26882627e33  -->


<!-- hello
openstack endpoint set --url "http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s" 4fa93f815d64428697bcc41e56f38d94 
openstack endpoint set --url "http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s" 5be5403aecb9418aa73288b0f56e1315 
openstack endpoint set --url "http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s" b7df0ccdceba42ce8272d26882627e33  -->

``` 
root@controller:/home/ubuntu# openstack endpoint set --url "http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s" 4fa93f815d64428697bcc41e56f38d94 
root@controller:/home/ubuntu# openstack endpoint set --url "http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s" 5be5403aecb9418aa73288b0f56e1315 
root@controller:/home/ubuntu# openstack endpoint set --url "http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s" b7df0ccdceba42ce8272d26882627e33 
root@controller:/home/ubuntu# openstack endpoint list
+----------------------------------+-----------+--------------+--------------+---------+-----------+----------------------------------------------------------+
| ID                               | Region    | Service Name | Service Type | Enabled | Interface | URL                                                      |
+----------------------------------+-----------+--------------+--------------+---------+-----------+----------------------------------------------------------+
| 1c1c7306ea024eb98fae578ed1383f99 | RegionOne | keystone     | identity     | True    | internal  | http://controller:5000/v3/                               |
| 4fa93f815d64428697bcc41e56f38d94 | RegionOne | swift        | object-store | True    | internal  | http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s |
| 5be5403aecb9418aa73288b0f56e1315 | RegionOne | swift        | object-store | True    | public    | http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s |
| b7df0ccdceba42ce8272d26882627e33 | RegionOne | swift        | object-store | True    | admin     | http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s |
| b9499be1f9ac4ad8ad9705f2e1a13847 | RegionOne | keystone     | identity     | True    | public    | http://controller:5000/v3/                               |
| badb42c672534b8290a37b828b758c9d | RegionOne | keystone     | identity     | True    | admin     | http://controller:5000/v3/                               |
+----------------------------------+-----------+--------------+--------------+---------+-----------+----------------------------------------------------------+
root@controller:/home/ubuntu# 
```

url 要加 双引号，否则报错如下：

```
root@controller:/home/ubuntu# openstack endpoint set --url http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s  4fa93f815d64428697bcc41e56f38d94 
bash: syntax error near unexpected token `('
root@controller:/home/ubuntu# 
```

查询，这时，会报错

```
root@controller:/home/ubuntu# . admin-openrc 
root@controller:/home/ubuntu# swift list
Account not found
root@controller:/home/ubuntu# 
```

回头再看文档，发现，是不是要同时修改 rgw节点的 /etc/ceph.conf 文件。

实际证明，是的。两个地方要同时修改。两处是相互配合的。如果只修改其中一处，则都会报错。

具体如下：

- /etc/ceph.conf

```
root@ceph-client:/etc/ceph# cat ceph.conf 
[global]
fsid = 87aa8ebd-2782-415d-bf23-ad2eb89b61c6
mon_initial_members = ceph-server-1, ceph-server-2, ceph-server-3
mon_host = 192.168.0.130,192.168.0.111,192.168.0.105
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
osd pool default size = 2
mon_clock_drift_allowed = 1
rgw swift account in url = true

[client.rgw.ceph-client]
rgw_frontends = "civetweb port=8080"

rgw keystone api version = 3
rgw keystone url = http://192.168.0.51:5000 
rgw keystone admin user = admin
rgw keystone admin password = openstack
rgw keystone admin domain = default
rgw keystone admin project = admin
rgw keystone accepted roles = admin, user 
rgw keystone token cache size = 500
rgw s3 auth use keystone = true
rgw keystone revocation interval = 60
rgw keystone implicit tenants = true

root@ceph-client:/etc/ceph# 
root@ceph-client:/etc/ceph# sudo systemctl restart ceph-radosgw@rgw.ceph-client.service
```

- openstack endpoint list

```
root@controller:/home/ubuntu# . admin-openrc 
root@controller:/home/ubuntu# openstack endpoint list | grep object-store
| 4fa93f815d64428697bcc41e56f38d94 | RegionOne | swift        | object-store | True    | internal  | http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s |
| 5be5403aecb9418aa73288b0f56e1315 | RegionOne | swift        | object-store | True    | public    | http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s |
| b7df0ccdceba42ce8272d26882627e33 | RegionOne | swift        | object-store | True    | admin     | http://ceph-rgw-client:8080/swift/v1/AUTH_%(project_id)s |
root@controller:/home/ubuntu# 
```

这样就意味着：

上传下载URL 由 http://192.168.0.134:8080/swift/v1/ 更新为： http://192.168.0.134:8080/swift/v1/AUTH_%(project_id)s/  ( 就是多加   AUTH_(project_id)s/   )

因为修改了 ceph.conf ，那把配置分发一下。

ceph-admin节点：

```
admin@ceph-admin:~/test-cluster$ ceph-deploy --overwrite-conf config push ceph-admin ceph-client ceph-server-1 ceph-server-2 ceph-server-3
```

然后，再验证一下。（因为在ceph-admin节点无法运行`ssh ceph-server-1` 命令，发现 ceph-server-1 节点有问题，要修复这个ssh问题，所以暂停了。）


### 换思路，对比日志情况

- tail -f  /var/log/ceph/ceph-client.rgw.ceph-client.log

#### 原生用户 tmp url 日志

```
2019-04-09 10:46:18.303 7fa3d7a12700  1 ====== starting new request req=0x7fa3d7a09830 =====
2019-04-09 10:46:18.987 7fa3d7a12700  1 ====== req done req=0x7fa3d7a09830 op status=1902 http_status=204 ======
2019-04-09 10:46:18.987 7fa3d7a12700  1 civetweb: 0x205c000: 192.168.0.158 - - [09/Apr/2019:10:46:18 +0800] "POST /swift/v1/ HTTP/1.1" 204 265 - python-requests/2.21.0
2019-04-09 10:46:18.995 7fa3d7a12700  1 ====== starting new request req=0x7fa3d7a09830 =====
2019-04-09 10:46:19.003 7fa3d7a12700  1 ====== req done req=0x7fa3d7a09830 op status=0 http_status=200 ======
2019-04-09 10:46:19.003 7fa3d7a12700  1 civetweb: 0x205c000: 192.168.0.158 - - [09/Apr/2019:10:46:18 +0800] "GET /swift/v1/ HTTP/1.1" 200 755 - python-requests/2.21.0
```

chrome： http://192.168.0.134:8080/swift/v1/swiftuser1-container1/ceph-swiftuser1-container1-object-1.txt?temp_url_sig=b9a878f2293eb67ddf2f68ee044fd1445791d596&temp_url_expires=1554778278

```
2019-04-09 10:46:34.251 7fa3d7211700  1 ====== starting new request req=0x7fa3d7208830 =====
2019-04-09 10:46:34.259 7fa3d7211700  1 ====== req done req=0x7fa3d7208830 op status=0 http_status=200 ======
2019-04-09 10:46:34.259 7fa3d7211700  1 civetweb: 0x205c9d8: 192.168.0.158 - - [09/Apr/2019:10:46:34 +0800] "GET /swift/v1/swiftuser1-container1/ceph-swiftuser1-container1-object-1.txt?temp_url_sig=b9a878f2293eb67ddf2f68ee044fd1445791d596&temp_url_expires=1554778278 HTTP/1.1" 200 538 - Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
2019-04-09 10:46:42.351 7fa3d8a14700  0 ERROR: keystone revocation processing returned error r=-5
```

#### keystone 用户 日志

```
2019-04-09 10:44:46.925 7fa3d7a12700  1 ====== starting new request req=0x7fa3d7a09830 =====
2019-04-09 10:44:47.477 7fa3d7a12700  0 validated token: demo:demo expires: 1555987486
2019-04-09 10:44:47.825 7fa3d7a12700  1 ====== req done req=0x7fa3d7a09830 op status=1902 http_status=204 ======
2019-04-09 10:44:47.825 7fa3d7a12700  1 civetweb: 0x205c000: 192.168.0.158 - - [09/Apr/2019:10:44:46 +0800] "POST /swift/v1 HTTP/1.1" 204 265 - python-requests/2.21.0
2019-04-09 10:44:47.837 7fa3d7a12700  1 ====== starting new request req=0x7fa3d7a09830 =====
2019-04-09 10:44:47.849 7fa3d7a12700  1 ====== req done req=0x7fa3d7a09830 op status=0 http_status=200 ======
2019-04-09 10:44:47.849 7fa3d7a12700  1 civetweb: 0x205c000: 192.168.0.158 - - [09/Apr/2019:10:44:47 +0800] "GET /swift/v1 HTTP/1.1" 200 762 - python-requests/2.21.0
```

chrome： http://192.168.0.134:8080/swift/v1/abcd/hello.txt?temp_url_sig=d5b943ad55c2aa8ed3d391bd917d99be84f9fa9f&temp_url_expires=1554778187

```
2019-04-09 10:45:22.138 7fa3d7a12700  1 ====== starting new request req=0x7fa3d7a09830 =====
2019-04-09 10:45:22.142 7fa3d7a12700  1 ====== req done req=0x7fa3d7a09830 op status=0 http_status=403 ======
2019-04-09 10:45:22.142 7fa3d7a12700  1 civetweb: 0x205c000: 192.168.0.158 - - [09/Apr/2019:10:45:22 +0800] "GET /swift/v1/abcd/hello.txt?temp_url_sig=d5b943ad55c2aa8ed3d391bd917d99be84f9fa9f&temp_url_expires=1554778187 HTTP/1.1" 403 318 - Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36
```

返回的是 403 资源不可用. 嘛意思？

### 查看 rgw用户与 keystone用户的区别

下面我们以 rgw用户 swiftuser1 与 keystone用户 a9e83dfd0fa14ec4b2e47842f2d5797a（demo）为例子，进行展示

- a9e83dfd0fa14ec4b2e47842f2d5797a

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

- swiftuser1

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

通过观察，知道，我们的 keystone 用户，没有 swift_keys 配置。

这个值，可以通过 `sudo radosgw-admin key create --subuser="${username}:${subusername}" --key-type=swift --gen-secret` 生成并替换

但是，subuser 也没有设定，怎么办？

### 配置 subuser 和 swift_keys

我们知道 rgw 用户 是通过下面的方式创建的

```
root@ceph-admin:~# cat ceph-swift-user1-openrc 
username=swiftuser1
subusername=swiftsubuser1
displayname=swiftuser1displayname
password=swiftuser1password
root@ceph-admin:~# 
root@ceph-admin:~# . ceph-swift-user1-openrc 
root@ceph-admin:~# sudo radosgw-admin user create --subuser="${username}:${subusername}" --uid="${username}" --display-name="${displayname}" --key-type=swift --secret="${password}" --access=full
```

相同的，我们试一下


```
admin@ceph-admin:~/test-data$ cat ceph-keystone-demo 
username=a9e83dfd0fa14ec4b2e47842f2d5797a\$a9e83dfd0fa14ec4b2e47842f2d5797a
subusername=demo
displayname=demo
password=openstack
admin@ceph-admin:~/test-data$ . ceph-keystone-demo 
admin@ceph-admin:~/test-data$ sudo radosgw-admin user create --subuser="${username}:${subusername}" --uid="${username}" --display-name="${displayname}" --key-type=swift --secret="${password}" --access=full 
could not create subuser: unable to create subuser, unable to store user info
admin@ceph-admin:~/test-data$ 
```

失败了。

所以，只能尝试 `sudo radosgw-admin user modify`

```
admin@ceph-admin:~/test-data$ radosgw-admin user modify --uid="${username}" --subuser="${username}:${subusername}" --display-name="${displayname}" --key-type=swift --secret="${password}" --access=full 
{
    "user_id": "a9e83dfd0fa14ec4b2e47842f2d5797a$a9e83dfd0fa14ec4b2e47842f2d5797a",
    "display_name": "demo",
    "email": "",
    "suspended": 0,
    "max_buckets": 1000,
    "auid": 0,
    "subusers": [],
    "keys": [],
    "swift_keys": [
        {
            "user": "a9e83dfd0fa14ec4b2e47842f2d5797a$a9e83dfd0fa14ec4b2e47842f2d5797a:demo",
            "secret_key": "openstack"
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
            "val": "demohelloworld"
        }
    ],
    "type": "keystone",
    "mfa_ids": []
}

admin@ceph-admin:~/test-data$ 
```

在 swift-client 节点尝试一下，获取 token

```
root@controller:/home/ubuntu# curl -X GET -H "X-Auth-User:a9e83dfd0fa14ec4b2e47842f2d5797a\$a9e83dfd0fa14ec4b2e47842f2d5797a:demo" -H "X-Auth-Key:openstack"  http://192.168.0.134:8080/auth -v
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying 192.168.0.134...
* Connected to 192.168.0.134 (192.168.0.134) port 8080 (#0)
> GET /auth HTTP/1.1
> Host: 192.168.0.134:8080
> User-Agent: curl/7.47.0
> Accept: */*
> X-Auth-User:a9e83dfd0fa14ec4b2e47842f2d5797a$a9e83dfd0fa14ec4b2e47842f2d5797a:demo
> X-Auth-Key:openstack
> 
< HTTP/1.1 204 No Content
< X-Storage-Url: http://192.168.0.134:8080/swift/v1
< X-Storage-Token: AUTH_rgwtk4600000061396538336466643066613134656334623265343738343266326435373937612461396538336466643066613134656334623265343738343266326435373937613a64656d6fbdfd3ea012e50d803494ad5c0458533bfeb782742194c86f788ad18a56ed51c3ac0523fb
< X-Auth-Token: AUTH_rgwtk4600000061396538336466643066613134656334623265343738343266326435373937612461396538336466643066613134656334623265343738343266326435373937613a64656d6fbdfd3ea012e50d803494ad5c0458533bfeb782742194c86f788ad18a56ed51c3ac0523fb
< X-Trans-Id: tx00000000000000000001f-005cac42b4-39385-default
< X-Openstack-Request-Id: tx00000000000000000001f-005cac42b4-39385-default
< Content-Type: application/json; charset=utf-8
< Date: Tue, 09 Apr 2019 06:59:00 GMT
< 
* Connection #0 to host 192.168.0.134 left intact
root@controller:/home/ubuntu# 
```

拿到了。

利用token进行设置 tmp url

发现会报 403 。

再对比，发现 `"subusers": []` 。也就是说给这个 subusers 设置了 swift_key, 但是，没有添加 subuser 用户。

```
admin@ceph-admin:~/test-data$ radosgw-admin subuser create --uid="${username}" --subuser="${username}:${subusername}" --display-name="${displayname}" --key-type=swift --secret="${password}" --access=full  
{
    "user_id": "a9e83dfd0fa14ec4b2e47842f2d5797a$a9e83dfd0fa14ec4b2e47842f2d5797a",
    "display_name": "demo",
    "email": "",
    "suspended": 0,
    "max_buckets": 1000,
    "auid": 0,
    "subusers": [
        {
            "id": "a9e83dfd0fa14ec4b2e47842f2d5797a$a9e83dfd0fa14ec4b2e47842f2d5797a:demo",
            "permissions": "full-control"
        }
    ],
    "keys": [],
    "swift_keys": [
        {
            "user": "a9e83dfd0fa14ec4b2e47842f2d5797a$a9e83dfd0fa14ec4b2e47842f2d5797a:demo",
            "secret_key": "openstack"
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
            "val": "demohelloworld2222222222222222222"
        }
    ],
    "type": "keystone",
    "mfa_ids": []
}

admin@ceph-admin:~/test-data$ 
```

这下都有了。

#### 生成 token 

```
root@controller:/home/ubuntu# curl -X GET -H "X-Auth-User:a9e83dfd0fa14ec4b2e47842f2d5797a\$a9e83dfd0fa14ec4b2e47842f2d5797a:demo" -H "X-Auth-Key:openstack"  http://192.168.0.134:8080/auth -v 
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying 192.168.0.134...
* Connected to 192.168.0.134 (192.168.0.134) port 8080 (#0)
> GET /auth HTTP/1.1
> Host: 192.168.0.134:8080
> User-Agent: curl/7.47.0
> Accept: */*
> X-Auth-User:a9e83dfd0fa14ec4b2e47842f2d5797a$a9e83dfd0fa14ec4b2e47842f2d5797a:demo
> X-Auth-Key:openstack
> 
< HTTP/1.1 204 No Content
< X-Storage-Url: http://192.168.0.134:8080/swift/v1
< X-Storage-Token: AUTH_rgwtk4600000061396538336466643066613134656334623265343738343266326435373937612461396538336466643066613134656334623265343738343266326435373937613a64656d6f61190960e628bb4a7f9aad5ce3f24b168054f7204cba88919a36fd3eac665283c179bbcc
< X-Auth-Token: AUTH_rgwtk4600000061396538336466643066613134656334623265343738343266326435373937612461396538336466643066613134656334623265343738343266326435373937613a64656d6f61190960e628bb4a7f9aad5ce3f24b168054f7204cba88919a36fd3eac665283c179bbcc
< X-Trans-Id: tx0000000000000000000a2-005cac48ff-39385-default
< X-Openstack-Request-Id: tx0000000000000000000a2-005cac48ff-39385-default
< Content-Type: application/json; charset=utf-8
< Date: Tue, 09 Apr 2019 07:25:51 GMT
< 
* Connection #0 to host 192.168.0.134 left intact
root@controller:/home/ubuntu# 
```

#### 生成 tmp url key



```python
import requests
import json 

token = "AUTH_rgwtk4600000061396538336466643066613134656334623265343738343266326435373937612461396538336466643066613134656334623265343738343266326435373937613a64656d6f61190960e628bb4a7f9aad5ce3f24b168054f7204cba88919a36fd3eac665283c179bbcc"
import hmac
from hashlib import sha1
from time import time

key = 'demohelloworld444444444444444'

URL = 'http://192.168.0.134:8080/swift/v1/'
URL = 'http://192.168.0.134:8080/swift/v1'
headers = {'X-Auth-Token':token, 'X-Account-Meta-Temp-URL-Key':key}
res = requests.post(URL,headers=headers)
print(res.text)
print(res.headers)
print(res.status_code)

URL = 'http://192.168.0.134:8080/swift/v1/'
URL = 'http://192.168.0.134:8080/swift/v1'
headers = {'X-Auth-Token':token, 'X-Account-Meta-Temp-URL-Key':key}
res = requests.get(URL,headers=headers)
print(res.text)
print(res.headers)
print(res.status_code)

print("------------------------------------------------") 
```

#### 生成 tmp url

```
import hmac
from hashlib import sha1
from time import time 
method = 'GET'
host = 'http://192.168.0.134:8080/swift'
# host = 'http://192.168.0.51:5000/v3/auth/tokens'
duration_in_seconds = 300  # Duration for which the url is valid
expires = int(time() + duration_in_seconds)
path = '/v1/abcd/hello.txt' 
hmac_body = '%s\n%s\n%s' % (method, expires, path)
key = bytes(key , 'utf-8')
hmac_body = bytes(hmac_body, 'utf-8')
sig = hmac.new(key, hmac_body, sha1).hexdigest()
rest_uri = "{host}{path}?temp_url_sig={sig}&temp_url_expires={expires}".format(
             host=host, path=path, sig=sig, expires=expires)

print(rest_uri) 
```

chrome: http://192.168.0.134:8080/swift/v1/abcd/hello.txt?temp_url_sig=cfca82e1b725919ef684b2d14a36c664cdb20bf4&temp_url_expires=1554795078

失败。

## ref
- http://docs.ceph.com/docs/master/radosgw/swift/tempurl/
- https://blog.csdn.net/u014104588/article/details/86248675?tdsourcetag=s_pcqq_aiomsg
- http://docs.ceph.org.cn/radosgw/swift/tempurl/