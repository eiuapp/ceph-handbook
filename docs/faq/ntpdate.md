
## env

```
admin@ceph-admin:~$ ceph -s
  cluster:
    id:     87aa8ebd-2782-415d-bf23-ad2eb89b61c6
    health: HEALTH_WARN
            application not enabled on 1 pool(s)
            clock skew detected on mon.ceph-server-2, mon.ceph-server-1
 
  services:
    mon: 3 daemons, quorum ceph-server-3,ceph-server-2,ceph-server-1
    mgr: ceph-server-1(active)
    osd: 2 osds: 2 up, 2 in
    rgw: 1 daemon active
 
  data:
    pools:   7 pools, 56 pgs
    objects: 227  objects, 2.0 KiB
    usage:   2.0 GiB used, 7.3 TiB / 7.3 TiB avail
    pgs:     56 active+clean
 
admin@ceph-admin:~$ 
```

根据提示`clock skew detected on mon.ceph-server-2, mon.ceph-server-1`, 说明是时间不对。观察后，知道，各组件之间的时间不一致。


## ntpdate 保证同一时间

安装与校对时间
```
sudo apt update && sudo apt install ntpdate -y
sudo ntpdate cn.pool.ntp.org && date
```

再观察

```
admin@ceph-server-1:~$ ceph -s
  cluster:
    id:     87aa8ebd-2782-415d-bf23-ad2eb89b61c6
    health: HEALTH_WARN
            application not enabled on 1 pool(s)
 
  services:
    mon: 3 daemons, quorum ceph-server-3,ceph-server-2,ceph-server-1
    mgr: ceph-server-1(active)
    osd: 2 osds: 2 up, 2 in
 
  data:
    pools:   7 pools, 56 pgs
    objects: 227  objects, 2.0 KiB
    usage:   2.0 GiB used, 7.3 TiB / 7.3 TiB avail
    pgs:     56 active+clean
 
admin@ceph-server-1:~$ 
```

时间问题解决。
