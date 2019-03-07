
python 通过 python-swiftclient 库实现上传文件至ceph

## env

swift 与 ceph 接口已打通

具体一些环境，可以看下面代码中的参数

- ceph rgw: http://192.168.0.134:8080
- swift user: 'swiftuser1:swiftsubuser1'
- swift key : '7Us5blLxYeh77cqWQR9qhgWT3e8BMOu8zokeKB3k'
- swift container : 'swiftuser1-container1'

## step 

安装包

```
pip install python-swiftclient
```

代码

```python
# -*- coding: utf-8 -*-
# 测试上传视频2个， 运行格式: `python uploadvideo.py 2` 
import swiftclient
import os
import sys
import datetime

# swift info
user = 'swiftuser1:swiftsubuser1'
key = '8Us5blLxYeh77cqWQR9qhgWT3e8BMOu8zokeKB3k'
rootdir = "./upload/"
count = sys.argv[1]
# container name
container_name = 'swiftuser1-container1'

# connect swift
conn = swiftclient.Connection(
        user=user,
        key=key,
        authurl='http://192.168.0.134:8080/auth',
)

# uploadfile
def uploadFile(pathName,num):
    if not num.isdigit():
        print('please input number')
        return
    num = int(num)
    print('start uploading')
    start = datetime.datetime.now()
    # read all file
    f = open(rootdir + '/video.txt', 'r')
    file_names = f.readlines()
    # Guarantee not to cross the borde
    temp = len(file_names) if num > len(file_names) else num
    if temp != num:
        print('The test may be unreasonable Insufficient number of documents')
    num = temp
    for i in range(num):
        filepath = pathName + '/' + file_names[i].strip('\n')
        conn.put_object(container_name, file_names[i].strip('\n'), open(filepath, 'rb'))
    end = datetime.datetime.now()
    print('test upload location ' + pathName +  ' number: ' + num.__str__() + ' spend time:' + (end-start).total_seconds().__str__() + 'microseconds')

#  upload file 500
uploadFile(rootdir,num=count)

```

准备上传文件

这里准备几个小文件

```
$ ls upload
dao.txt  dong.txt  guo.txt  video.txt  zuo.txt
```

运行效果

```
(cephswift) ➜  CephSwiftTest git:(master) ✗ python uploadvideo.py 2
start uploading
test upload location /mnt/f/tmp/upload/ number: 2 spend time:1.272459microseconds
(cephswift) ➜  CephSwiftTest git:(master) ✗
```

去 swift client 检查

```
ubuntu@controller:~$ . ceph-swift-user1-openrc
ubuntu@controller:~$ swift -A http://192.168.0.134:8080/auth/v1.0 -U ${username}:${subusername} -K ${password} list swiftuser1-container1
ceph-swiftuser1-container1-object-1.txt
dao.txt
guo.txt
ubuntu@controller:~$ 
```

文件在。耶耶耶。成功。

## ref
- https://github.com/JakeRed/CephSwiftTest
