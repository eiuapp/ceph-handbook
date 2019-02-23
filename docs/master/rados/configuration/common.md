
- 典型的 Ceph 生产集群至少部署 3 个监视器来确保高可靠性。最好奇数个。

在 ceph-server-[1,2,3] 中会有类型下面的文件夹（ceph-admin, ceph-client）没有：

```
vagrant@ceph-server-3:~$ sudo ls /var/lib/ceph/mon/ceph-ceph-server-3/
done  keyring  kv_backend  store.db  systemd
vagrant@ceph-server-3:~$ 
```