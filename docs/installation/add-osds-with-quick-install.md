

本文基于 ceph deploy quick install 而 Add OSDs

## env

同上

## step

### Add OSDs

```
admin@lattepanda:~$ ceph health
HEALTH_OK
admin@lattepanda:~$ ceph -s
  cluster:
    id:     87aa8ebd-2782-415d-bf23-ad2eb89b61c6
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-server-3,ceph-server-2,ceph-server-1
    mgr: ceph-server-1(active)
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:     
 
admin@lattepanda:~$ 
```

大家看到，现在的 osd , pools, pgs, objects 等数据均为0。

我们要把 硬盘加上去，才算是把 osd 分配完成。

```
ceph-deploy osd create --data /dev/sda1 ceph-server-2
ceph-deploy osd create --data /dev/sda1 ceph-server-3
```

### 报错`GenericError: Failed to create 1 OSDs`

如果 `Failed to execute command: /usr/sbin/ceph-volume --cluster ceph lvm create --bluestore --data /dev/sda1`，具体如下：

```
admin@ceph-admin:~/test-cluster$ ceph-deploy osd create --data /dev/sda1 ceph-server-3
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/admin/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /usr/bin/ceph-deploy osd create --data /dev/sda1 ceph-server-3
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  bluestore                     : None
[ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x7f7bd4f96908>
[ceph_deploy.cli][INFO  ]  cluster                       : ceph
[ceph_deploy.cli][INFO  ]  fs_type                       : xfs
[ceph_deploy.cli][INFO  ]  block_wal                     : None
[ceph_deploy.cli][INFO  ]  default_release               : False
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  journal                       : None
[ceph_deploy.cli][INFO  ]  subcommand                    : create
[ceph_deploy.cli][INFO  ]  host                          : ceph-server-3
[ceph_deploy.cli][INFO  ]  filestore                     : None
[ceph_deploy.cli][INFO  ]  func                          : <function osd at 0x7f7bd53f1c08>
[ceph_deploy.cli][INFO  ]  ceph_conf                     : None
[ceph_deploy.cli][INFO  ]  zap_disk                      : False
[ceph_deploy.cli][INFO  ]  data                          : /dev/sda1
[ceph_deploy.cli][INFO  ]  block_db                      : None
[ceph_deploy.cli][INFO  ]  dmcrypt                       : False
[ceph_deploy.cli][INFO  ]  overwrite_conf                : False
[ceph_deploy.cli][INFO  ]  dmcrypt_key_dir               : /etc/ceph/dmcrypt-keys
[ceph_deploy.cli][INFO  ]  quiet                         : False
[ceph_deploy.cli][INFO  ]  debug                         : False
[ceph_deploy.osd][DEBUG ] Creating OSD on cluster ceph with data device /dev/sda1
[ceph-server-3][DEBUG ] connection detected need for sudo
[ceph-server-3][DEBUG ] connected to host: ceph-server-3 
[ceph-server-3][DEBUG ] detect platform information from remote host
[ceph-server-3][DEBUG ] detect machine type
[ceph-server-3][DEBUG ] find the location of an executable
[ceph_deploy.osd][INFO  ] Distro info: Ubuntu 16.04 xenial
[ceph_deploy.osd][DEBUG ] Deploying osd to ceph-server-3
[ceph-server-3][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
[ceph-server-3][DEBUG ] find the location of an executable
[ceph-server-3][INFO  ] Running command: sudo /usr/sbin/ceph-volume --cluster ceph lvm create --bluestore --data /dev/sda1
[ceph-server-3][WARNIN] -->  RuntimeError: command returned non-zero exit status: 5
[ceph-server-3][DEBUG ] Running command: /usr/bin/ceph-authtool --gen-print-key
[ceph-server-3][DEBUG ] Running command: /usr/bin/ceph --cluster ceph --name client.bootstrap-osd --keyring /var/lib/ceph/bootstrap-osd/ceph.keyring -i - osd new f2d1197f-301b-419a-a83b-e5cb8634e594
[ceph-server-3][DEBUG ] Running command: /sbin/vgcreate --force --yes ceph-d702b141-e07b-4067-ac55-f12f2ac7fd90 /dev/sda1
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: /run/lvm/lvmetad.socket: connect failed: No such file or directory
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: WARNING: Failed to connect to lvmetad. Falling back to internal scanning.
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: /dev/mmcblk0rpmb: read failed after 0 of 4096 at 4128768: Input/output error
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: /dev/mmcblk0rpmb: read failed after 0 of 4096 at 4186112: Input/output error
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: /dev/mmcblk0rpmb: read failed after 0 of 4096 at 0: Input/output error
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: /dev/mmcblk0rpmb: read failed after 0 of 4096 at 4096: Input/output error
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: Can't open /dev/sda1 exclusively.  Mounted filesystem?
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ]  stderr: Unable to add physical volume '/dev/sda1' to volume group 'ceph-d702b141-e07b-4067-ac55-f12f2ac7fd90'.
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][DEBUG ] --> Was unable to complete a new OSD, will rollback changes
[ceph-server-3][DEBUG ] Running command: /usr/bin/ceph --cluster ceph --name client.bootstrap-osd --keyring /var/lib/ceph/bootstrap-osd/ceph.keyring osd purge-new osd.1 --yes-i-really-mean-it
[ceph-server-3][DEBUG ]  stderr: purged osd.1
[ceph-server-3][DEBUG ]  stderr: 
[ceph-server-3][ERROR ] RuntimeError: command returned non-zero exit status: 1
[ceph_deploy.osd][ERROR ] Failed to execute command: /usr/sbin/ceph-volume --cluster ceph lvm create --bluestore --data /dev/sda1
[ceph_deploy][ERROR ] GenericError: Failed to create 1 OSDs

admin@ceph-admin:~/test-cluster$ 
```

可能原因：

- 硬盘未处于 umount 状态。比如已经mount了。

```
admin@ceph-server-3:~$ mount |grep "sda1"
/dev/sda1 on /media/admin/f7d6f660-f0cf-4c92-8cef-abc8db2d0d0a type xfs (rw,nosuid,nodev,relatime,attr2,inode64,noquota,uhelper=udisks2)
admin@ceph-server-3:~$ umount /media/admin/f7d6f660-f0cf-4c92-8cef-abc8db2d0d0a 
```

### ceph -s

```
admin@ceph-admin:~/test-cluster$ ceph -s
  cluster:
    id:     87aa8ebd-2782-415d-bf23-ad2eb89b61c6
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-server-3,ceph-server-2,ceph-server-1
    mgr: ceph-server-1(active)
    osd: 2 osds: 2 up, 2 in
    rgw: 1 daemon active
 
  data:
    pools:   5 pools, 40 pgs
    objects: 187  objects, 1.1 KiB
    usage:   2.0 GiB used, 7.3 TiB / 7.3 TiB avail
    pgs:     40 active+clean
 
admin@ceph-admin:~/test-cluster$ 
```

这里，会显示出 osd, pools, objects 的数量，且 pgs 为 `active+clean`

- **objects** 在未存放数据的情况下不为0


