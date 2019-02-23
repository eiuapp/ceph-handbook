
## MONITORS
- 典型的 Ceph 生产集群至少部署 3 个监视器来确保高可靠性。最好奇数个。

在 ceph-server-[1,2,3] 中会有类型下面的文件夹（ceph-admin, ceph-client）没有：

```
vagrant@ceph-server-3:~$ sudo ls /var/lib/ceph/mon/ceph-ceph-server-3/
done  keyring  kv_backend  store.db  systemd
vagrant@ceph-server-3:~$ 
```

## OSDS

在 ceph-server-2 和 ceph-server-3 中 (以 ceph-server-2 为例)

```
vagrant@ceph-server-2:~$ sudo ls /var/lib/ceph/osd/ceph-0 -l
lrwxrwxrwx 1 root root 15 Feb 23 03:37 /var/lib/ceph/osd/ceph-0 -> /var/local/osd0
vagrant@ceph-server-2:~$ sudo ls /var/lib/ceph/osd/ceph-0 
activate.monmap  active  block	bluefs	ceph_fsid  fsid  keyring  kv_backend  magic  mkfs_done	ready  systemd	type  whoami
vagrant@ceph-server-2:~$ 
```

We recommend using the xfs file system when running mkfs. (btrfs and ext4 are not recommended and no longer tested.)

See the OSD Config Reference for additional configuration details.

但是，说好得 [bluestore](http://docs.ceph.com/docs/master/rados/configuration/storage-devices/#bluestore) 怎么搞？