

### 配置

#### openrc 方式

```
root@controller:/home/ubuntu# cat admin-openrc 
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=openstack
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2

root@controller:/home/ubuntu# . admin-openrc 
root@controller:/home/ubuntu# 
```


#### 参数 方式

```
root@controller:/home/ubuntu# swift --os-auth-url http://controller:5000/v3 --auth-version 3        --os-project-name admin --os-project-domain-name Default         --os-username admin --os-user-domain-name Default         --os-password openstack list abcd
App.ico
hello.txt
world.txt
root@controller:/home/ubuntu#
```

### 设置并生成tmp url

#### 命令行方式

```
root@controller:/home/ubuntu# swift post -m "Temp-URL-Key:admintmpurlkey"

```

#### py方式

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

### 验证

#### 命令行方式

`swift stat -v` 输出的 `Meta Temp-Url-Key` 就是。 

```
root@controller:/home/ubuntu# swift stat -v
                                 StorageURL: http://ceph-rgw-client:8080/swift/v1
                                 Auth Token: gAAAAABcrBT-mEGVWB-gx706oHzO87Fl8c9XR5F8DmhC_rvSaefWISL9Y_hmYw_Ubf5HlY0K_UISlp14U2ZubTayDhTMtqS7UF4r2xxWP9BMczLCOWJHeu80Z00HVVH-GSWfbyO8n-dNpCiCNmEPF8aYzt6M9ZcWwv_D4a9M4pWfxu4GwGQk9qw
                                    Account: v1
                                 Containers: 1
                                    Objects: 3
                                      Bytes: 25529
Objects in policy "default-placement-bytes": 0
  Bytes in policy "default-placement-bytes": 0
   Containers in policy "default-placement": 1
      Objects in policy "default-placement": 3
        Bytes in policy "default-placement": 25529
                          Meta Temp-Url-Key: admintmpurlkey
                              Accept-Ranges: bytes
                                X-Timestamp: 1554781438.70248
                X-Account-Bytes-Used-Actual: 36864
                                 X-Trans-Id: tx000000000000000000463-005cac14fe-392d5-default
                               Content-Type: text/plain; charset=utf-8
                     X-Openstack-Request-Id: tx000000000000000000463-005cac14fe-392d5-default
root@controller:/home/ubuntu# 
```

#### py方式

```python
URL = 'http://192.168.0.134:8080/swift/v1/'
# headers = {'X-Auth-Token':token, 'X-Account-Meta-Temp-URL-Key':"hello"}
headers = {'X-Auth-Token':token}
res = requests.get(URL,headers=headers)
print(res.text)
print(res.headers)
print(res.status_code)
```

运行与输出 中的 `'X-Account-Meta-Temp-Url-Key': 'admintmpurlkey'` 就是。

```
(swift) ➜  utils git:(master) ✗ python u.py
abcd
{'X-Timestamp': '1554781444.03614', 'X-Account-Container-Count': '1', 'X-Account-Object-Count': '3', 'X-Account-Bytes-Used': '25529', 'X-Account-Bytes-Used-Actual': '36864', 'X-Account-Storage-Policy-Default-Placement-Container-Count': '1', 'X-Account-Storage-Policy-Default-Placement-Object-Count': '3', 'X-Account-Storage-Policy-Default-Placement-Bytes-Used': '25529', 'X-Account-Storage-Policy-Default-Placement-Bytes-Used-Actual': '36864', 'X-Account-Meta-Temp-Url-Key': 'admintmpurlkey', 'Accept-Ranges': 'bytes', 'X-Trans-Id': 'tx000000000000000000464-005cac1503-392d5-default', 'X-Openstack-Request-Id': 'tx000000000000000000464-005cac1503-392d5-default', 'Content-Type': 'text/plain; charset=utf-8', 'Content-Length': '4', 'Date': 'Tue, 09 Apr 2019 03:44:04 GMT', 'Connection': 'Keep-Alive'}
200
```
(swift) ➜  utils git:(master) ✗