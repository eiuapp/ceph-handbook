
ceph-deploy osd create --data /dev/sda1 ceph-server-2 时报错如下：

![ceph-deploy-osd-create-error](https://res.cloudinary.com/dmtixvmgt/image/upload/v1553165879/ceph-deploy-osd-new-error_xfdphg.png)

## env

![fdisk-l](https://res.cloudinary.com/dmtixvmgt/image/upload/v1553165879/fdisk-l_bvn2wx.png)

## step

参考 http://docs.ceph.org.cn/rados/deployment/ceph-deploy-osd/


![ceph-deploy-disk-list](https://res.cloudinary.com/dmtixvmgt/image/upload/v1553165879/ceph-deploy-disk-list_db8kju.png)

能找到 磁盘，但是，没有显示 ceph 盘

![ceph-deploy-disk-zap](https://res.cloudinary.com/dmtixvmgt/image/upload/v1553165879/ceph-deploy-disk-zap_oxq3zu.png)

## ref

- http://docs.ceph.org.cn/rados/deployment/ceph-deploy-osd/