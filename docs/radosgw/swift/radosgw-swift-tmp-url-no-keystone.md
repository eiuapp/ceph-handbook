
当用户不通过keystone认证时，设置并生成tmp url

## env

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

### 查看帐户，容器，对象信息

```
root@controller:/home/ubuntu# cat ceph-swift-user1-openrc
username=swiftuser1
subusername=swiftsubuser1
displayname=swiftuser1displayname
password=G8jTHyahPudXOQ0Dv3oDn3wq9fMKnbcc9e4IumZO
root@controller:/home/ubuntu# . ceph-swift-user1-openrc
```

要获取上面的password等字段，则通过：

- `radosgw-admin user info --uid="swiftuser1"` 得到
- `sudo radosgw-admin key create --subuser="${username}:${subusername}" --key-type=swift --gen-secret` 生成并替换


```
root@controller:/home/ubuntu# swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password}  list swiftuser1-container1
ceph-swiftuser1-container1-object-1.txt
dao.txt
guo.txt
root@controller:/home/ubuntu# 
```

这里有一个 ceph-swiftuser1-container1-object-1.txt 对象，等会我们就把这个对象生成tmp url 来访问。

### 得到token

```
root@controller:~# curl -X GET -H "X-Auth-User:swiftuser1:swiftsubuser1" -H "X-Auth-Key:G8jTHyahPudXOQ0Dv3oDn3wq9fMKnbcc9e4IumZO"  http://192.168.0.134:8080/auth -v
Note: Unnecessary use of -X or --request, GET is already inferred.
*   Trying 192.168.0.134...
* Connected to 192.168.0.134 (192.168.0.134) port 8080 (#0)
> GET /auth HTTP/1.1
> Host: 192.168.0.134:8080
> User-Agent: curl/7.47.0
> Accept: */*
> X-Auth-User:swiftuser1:swiftsubuser1
> X-Auth-Key:G8jTHyahPudXOQ0Dv3oDn3wq9fMKnbcc9e4IumZO
> 
< HTTP/1.1 204 No Content
< X-Storage-Url: http://192.168.0.134:8080/swift/v1
< X-Storage-Token: AUTH_rgwtk18000000737769667475736572313a737769667473756275736572313568fcfafc4ff2bf40e3cd567292dd2786f6771b677628913b927020413a5debf2db55b4
< X-Auth-Token: AUTH_rgwtk18000000737769667475736572313a737769667473756275736572313568fcfafc4ff2bf40e3cd567292dd2786f6771b677628913b927020413a5debf2db55b4
< X-Trans-Id: tx00000000000000000004a-0056cc91c0-603b-default
< X-Openstack-Request-Id: tx00000000000000000004a-0056cc91c0-603b-default
< Content-Type: application/json; charset=utf-8
< Date: Tue, 23 Feb 2016 17:07:12 GMT
< 
* Connection #0 to host 192.168.0.134 left intact
root@controller:~# 
```

好了，我们要的token 就是： `AUTH_rgwtk18000000737769667475736572313a737769667473756275736572313568fcfafc4ff2bf40e3cd567292dd2786f6771b677628913b927020413a5debf2db55b4`

### 设置并生成tmp url

tmp_url.py 内容如下

```python
import requests
import json

print("设置Temp URL------------------------------------------------")
token = "AUTH_rgwtk18000000737769667475736572313a737769667473756275736572313568fcfafc4ff2bf40e3cd567292dd2786f6771b677628913b927020413a5debf2db55b4"
URL = 'http://192.168.0.134:8080/swift/v1/'
headers = {'X-Auth-Token':token, 'X-Account-Meta-Temp-URL-Key':"hello"}
res = requests.post(URL,headers=headers)
print(res.text)
print(res.headers)
print(res.status_code)

print("查看有没有设置成功------------------------------------------------")
URL = 'http://192.168.0.134:8080/swift/v1/'
# headers = {'X-Auth-Token':token, 'X-Account-Meta-Temp-URL-Key':"hello"}
headers = {'X-Auth-Token':token}
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
path = '/v1/swiftuser1-container1/ceph-swiftuser1-container1-object-1.txt'
key = 'hello'
hmac_body = '%s\n%s\n%s' % (method, expires, path)
key = bytes(key , 'utf-8')
hmac_body = bytes(hmac_body, 'utf-8')
sig = hmac.new(key, hmac_body, sha1).hexdigest()
rest_uri = "{host}{path}?temp_url_sig={sig}&temp_url_expires={expires}".format(
             host=host, path=path, sig=sig, expires=expires)
print(rest_uri)
```

#### （可忽略）生成tmp url部分如下：

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

### 运行`python tmp_url.py`后，得到 

http://192.168.0.134:8080/swift/v1/swiftuser1-container1/ceph-swiftuser1-container1-object-1.txt?temp_url_sig=f223797922ab7e4371c93d619f4bceac98069a3c&temp_url_expires=1554287683

但是，放到 chrome 中：

直接下载了一个 ceph-swiftuser1-container1-object-1.txt 

查看一下 ceph-swiftuser1-container1-object-1.txt 文件内容，没毛病，内容正确。


## ref
- http://docs.ceph.com/docs/master/radosgw/swift/tempurl/
- https://blog.csdn.net/u014104588/article/details/86248675?tdsourcetag=s_pcqq_aiomsg
- http://docs.ceph.org.cn/radosgw/swift/tempurl/