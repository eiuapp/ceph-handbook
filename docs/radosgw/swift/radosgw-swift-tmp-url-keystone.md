
当用户通过keystone认证时，设置并生成tmp url

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


## ref
- http://docs.ceph.com/docs/master/radosgw/swift/tempurl/
- https://blog.csdn.net/u014104588/article/details/86248675?tdsourcetag=s_pcqq_aiomsg
- http://docs.ceph.org.cn/radosgw/swift/tempurl/