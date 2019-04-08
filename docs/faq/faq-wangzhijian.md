转自： https://blog.51cto.com/wangzhijian/2159701?source=dra


关于ceph的一些问题及解决
==============


1.问题:
```
# ceph health
HEALTH\_WARN application not enabled on 1 pool(s)
```
  

解决:
```
# ceph health detail
HEALTH\_WARN application not enabled on 1 pool(s)
POOL\_APP\_NOT\_ENABLED application not enabled on 1 pool(s)
    application not enabled on pool 'kube'
    use 'ceph osd pool application enable <pool-name> <app-name>', where <app-name> is 'cephfs', 'rbd', 'rgw', or freeform for custom applications.
# ceph osd pool application enable kube rbd
enabled application 'rbd' on pool 'kube'
# ceph health
HEALTH\_OK
```
  

2.问题:
```
# ceph -s
  cluster:
    id:     e781a2e4-097d-4867-858d-bdbd3a264435
    health: HEALTH\_WARN
            clock skew detected on mon.ceph02, mon.ceph03
```
  

解决:  
```
####确认NTP服务是否正常工作
# systemctl status ntpd
####修改ceph配置中的时间偏差阈值
# vim /etc/ceph/ceph.conf
###在global字段下添加:
mon clock drift allowed = 2
mon clock drift warn backoff = 30   
####向需要同步的mon节点推送配置文件
# cd /etc/ceph/
# ceph-deploy --overwrite-conf config  push ceph{01..03}
####重启mon服务并验证
# systemctl restart ceph-mon.target
# ceph -s
  cluster:
    id:     e781a2e4-097d-4867-858d-bdbd3a264435
    health: HEALTH\_OK
```
  

3.问题:
```
# rbd map abc/zhijian --id admin
rbd: sysfs write failed
RBD image feature set mismatch. Try disabling features unsupported by the kernel with "rbd feature disable".
In some cases useful info is found in syslog - try "dmesg | tail".
rbd: map failed: (6) No such device or address
```
  

解决:

由于kernel不支持块设备镜像的一些特性，所以映射失败
```
# rbd feature disable abc/zhijian exclusive-lock, object-map, fast-diff, deep-flatten
# rbd info abc/zhijian
rbd image 'zhijian':
size 1024 MB in 256 objects
order 22 (4096 kB objects)
block\_name\_prefix: rbd\_data.1011074b0dc51
format: 2
features: layering
flags: 
create\_timestamp: Sun May  6 13:35:21 2018
# rbd map abc/zhijian --id admin
/dev/rbd0
```
  

4.问题:
```
# ceph osd pool delete cephfs\_data
Error EPERM: WARNING: this will \*PERMANENTLY DESTROY\* all data stored in pool cephfs\_data.  If you are \*ABSOLUTELY CERTAIN\* that is what you want, pass the pool name \*twice\*, followed by --yes-i-really-really-mean-it.
# ceph osd pool delete cephfs\_data cephfs\_data --yes-i-really-really-mean-it
Error EPERM: pool deletion is disabled; you must first set the mon\_allow\_pool\_delete config option to true before you can destroy a pool
```
  

解决:
```
# tail -n 2 /etc/ceph/ceph.conf 
\[mon\]
mon allow pool delete = true
```
  

向需要同步的mon节点推送配置文件:
```
# cd /etc/ceph/
# ceph-deploy --overwrite-conf config  push ceph{01..03}
```
  

重启mon服务并验证:
```
# systemctl restart ceph-mon.target
# ceph osd pool delete cephfs\_data cephfs\_data --yes-i-really-really-mean-it
pool 'cephfs\_data' removed
```
  

5.问题:
```
# ceph osd pool rm cephfs\_data cephfs\_data --yes-i-really-really-mean-it
Error EBUSY: pool 'cephfs\_data' is in use by CephFS
```
  

解决:
```
# ceph fs ls
name: cephfs, metadata pool: cephfs\_metadata, data pools: \[cephfs\_data \]
#  ceph fs rm cephfs --yes-i-really-mean-it
Error EINVAL: all MDS daemons must be inactive before removing filesystem
# systemctl stop ceph-mds.target
# ceph fs rm cephfs
Error EPERM: this is a DESTRUCTIVE operation and will make data in your filesystem permanently inaccessible.  Add --yes-i-really-mean-it if you are sure you wish to continue.
# ceph fs rm cephfs --yes-i-really-mean-it
# ceph fs ls
No filesystems enabled
```
  

6.问题:

使用静态PV创建pod，pod一直处于ContainerCreating状态:
```
# kubectl get pod ceph-pod1
NAME        READY     STATUS              RESTARTS   AGE
ceph-pod1   0/1       ContainerCreating   0          10s
......
# kubectl describe pod ceph-pod1
Warning  FailedMount             41s (x8 over 1m)  kubelet, node01            MountVolume.WaitForAttach failed for volume "ceph-pv" : fail to check rbd image status with: (executable file not found in $PATH), rbd output: ()
Warning  FailedMount             0s                kubelet, node01            Unable to mount volumes for pod "ceph-pod1\_default(14e3a07d-93a8-11e8-95f6-000c29b1ec26)": timeout expired waiting for volumes to attach or mount for pod "default"/"ceph-pod1". list of unmounted volumes=\[ceph-vol1\]. list of unattached volumes=\[ceph-vol1 default-token-v9flt\]
```
解决:node节点安装最新版的ceph-common解决该问题，ceph集群使用的是最新的mimic版本，而base源的版本太陈旧，故出现该问题

  

7.问题:

创建动态PV，PVC一直处于pending状态:
```
# kubectl get pvc -n ceph
NAME       STATUS   VOLUME  CAPACITY  ACCESS MODES  STORAGECLASS  AGE
ceph-pvc   Pending                                     ceph-rbd    2m
# kubectl describe pvc -n ceph
......
Warning  ProvisioningFailed  27s   persistentvolume-controller  Failed to provision volume with StorageClass "ceph-rbd": failed to create rbd image: exit status 1, command output: 2018-07-31 11:10:33.395991 7faa3558b7c0 -1 did not load config file, using default settings.
rbd: extraneous parameter --image-feature
```
解决:  

persistentvolume-controller 服务运行在master节点，受kube-controller-manager 控制，故master节点也需要安装ceph-common包

  
