Ceph M 快速安装 - 好脑袋和烂笔头 

参考： https://my.oschina.net/guol/blog/1928348  有修改

ceph角色分配
========

172.31.68.241
admin-node/deph-deploy/mon/mgr/mds/rgw

172.31.68.242
osd.0/mon

172.31.68.243
osd.1/mon

配置ssh无密码登录
==========

admin-node要可以无密码ssh登录osd机器，如果是普通用户，则要分配sudo权限，如下：

    useradd -d /home/cephadmin -m cephadmin
    echo "cephadmin ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephadmin
    chmod 0440 /etc/sudoers.d/cephadmin

快速安装
====

    ceph-deploy new ceph1
    
    ceph-deploy install ceph1 ceph2 ceph3
    
    ceph-deploy --overwrite-conf mon create-initial
    
    ceph-deploy admin ceph1 ceph2 ceph3
    
    
    ceph-deploy mgr create ceph1
    
    ceph-deploy osd create --data /dev/vdb1 ceph2
    ceph-deploy osd create --data /dev/vdb1 ceph3
    ceph health
    ceph -s

扩展
==

    ceph-deploy mds create ceph1
    
    ceph-deploy mon add ceph2
    ceph-deploy mon add ceph3
    
    ceph quorum_status --format json-pretty


本地配置上传
--------

    ceph-deploy --overwrite-conf config push ceph-admin ceph-server-1 ceph-server-2

安装rgw
--------

    ceph-deploy rgw create ceph1
    
    调整配置
    [client.rgw.ceph1]
    rgw_frontends = "civetweb port=8080"
    
    ceph-deploy --overwrite-conf admin ceph1 ceph2 ceph3 ceph4
    systemctl restart ceph-radosgw@rgw.ceph1.service
    curl http://ceph1:8080 -I

模拟client
========

    apt-get install ceph
    ceph-deploy admin ceph4

  
对象存储
-------

    echo 'hello ceph oject storage' > testfile.txt
    
    创建pool
    ceph osd pool create mytest 8
    
    上传文件
    rados put test-object-1 testfile.txt --pool=mytest
    
    
    rados -p mytest ls
    
    获取文件
    rados get test-object-1 testfile.txt.1 --pool=mytest
    
    查看映射位置
    ceph osd map mytest test-object-1
    
    删除文件
    rados rm test-object-1 --pool=mytest
    删除pool
    ceph osd pool rm mytest mytest --yes-i-really-really-mean-it

块存储
---

    admin上执行：ceph osd pool create rdb 8
    admin上执行：rbd pool init rdb
    
    admin上执行：
    
    rbd create foo --size 512 --image-feature layering -p rdb
    rbd map foo --name client.admin -p rdb

cephfs
------

    admin：ceph osd pool create cephfs_data 4
    admin：ceph osd pool create cephfs_metadata 4
    admin：ceph osd lspools
    admin：ceph fs new cephfs cephfs_metadata cephfs_data
    
    admin.secret内容为ceph.client.admin.keyring内容的一部分
    AQDhRX1baLeFFxAAskNapEuyipJ7SqS7Q1mh/Q==
    
    内核级别挂载方法：
    mkdir /mnt/mycephfs
    mount -t ceph 172.31.68.241:6789,172.31.68.242:6789,172.31.68.243:6789:/ /mnt/mycephfs -o name=admin,secretfile=admin.secret
    
    实验
    cd /mnt/mycephfs
    echo 'hello ceph CephFS' > hello.txt
    cd ~
    
    卸载
    umount -lf /mnt/mycephfs
    rm -rf /mnt/mycephfs
    
    用户级别挂载：
    mkdir /mnt/mycephfs
    ceph-fuse -m 172.31.68.241:6789 /mnt/mycephfs

S3 存储
-----

    ceph osd pool create .rgw 8 8
    ceph osd pool create .rgw.root 8 8
    ceph osd pool create .rgw.control 8 8
    ceph osd pool create .rgw.gc 8 8
    ceph osd pool create .rgw.buckets 8 8
    ceph osd pool create .rgw.buckets.index 8 8
    ceph osd pool create .rgw.buckets.extra 8 8
    ceph osd pool create .log 8 8
    ceph osd pool create .intent-log 8 8
    ceph osd pool create .usage 8 8
    ceph osd pool create .users 8 8
    ceph osd pool create .users.email 8 8
    ceph osd pool create .users.swift 8 8
    ceph osd pool create .users.uid 8 8

vm外挂磁盘
======

    cd /opt/vm/data_image
    qemu-img create -f qcow2 ubuntu16.04-2-data.img 2G
    qemu-img create -f qcow2 ubuntu16.04-3-data.img 2G
    virsh attach-disk [--domin] $DOMIN  [--source] $SOURCEFILE [--target] $TARGET --subdriver qcow2 --config --live
    virsh attach-disk Ubuntu16.04-2 /opt/vm/data_image/ubuntu16.04-2-data.img vdb  --subdriver qcow2
    virsh attach-disk Ubuntu16.04-3 /opt/vm/data_image/ubuntu16.04-3-data.img vdb  --subdriver qcow2

清除安装包
=====

    ceph-deploy purge ceph1 ceph2 ceph3
    
    清除配置信息
    ceph-deploy purgedata ceph1 ceph2 ceph3
    ceph-deploy forgetkeys
    
    每个节点删除残留的配置文件
    rm -rf /var/lib/ceph/osd/*
    rm -rf /var/lib/ceph/mon/*
    rm -rf /var/lib/ceph/mds/*
    rm -rf /var/lib/ceph/bootstrap-mds/*
    rm -rf /var/lib/ceph/bootstrap-osd/*
    rm -rf /var/lib/ceph/bootstrap-mon/*
    rm -rf /var/lib/ceph/tmp/*
    rm -rf /etc/ceph/*
    rm -rf /var/run/ceph/*

清除lvm配置
=======

    vgscan 
    vgdisplay -v
    lvremove
    vgremove
    pvremove

ceph-deplop
====

    ceph-deploy disk list ceph-server-2 # 列出ceph-server-2节点磁盘
    
## ref
- https://my.oschina.net/guol/blog/1928348
