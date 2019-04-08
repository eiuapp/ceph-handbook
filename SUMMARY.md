# Summary

## 序言

- [前言](README.md)

## env

* [环境](docs/env/env.md)
* [常用命令](docs/command/command.md)

## install

* [安装](docs/installation/README.md)
    * [vagrant install](docs/installation/vagrant/README.md)
        * [carmstrong/multinode-ceph-vagrant](docs/installation/vagrant/carmstrong/multinode-ceph-vagrant.md)
        * [oprypin/vagrant-ceph-quickstart](docs/installation/vagrant/oprypin/vagrant-ceph-quickstart.md)
    * [quick install](docs/installation/quick-install.md)
    * [添加osd](docs/installation/add-osds-with-quick-install.md)
    * [添加rgw实例](docs/installation/add-an-rgw-instance-with-quick-install.md)
    
    * [Installation (ceph-deploy)](ceph-deploy.md)

    * [Installation 常见报错](docs/installation/install-faq.md)

## Intro to Ceph

* [硬件推荐](docs/hardware-recommendations.md)



## configuration

* [configuration/common](docs/master/rados/configuration/common.md)

## RADOSGW

* [radosgw 与 swift]()
    * [使用RADOSGW提供ceph的Swift接口](docs/radosgw/swift/authentication.md)
    * [swift获取auth](docs/radosgw/swift/radosgw-swift-get-auth.md)
    * [swift上传文件](docs/radosgw/swift/radosgw-swift-upload.md)
    * [ceph集成openstack swift](docs/radosgw/swift/openstack-swift-with-ceph-backend-radosgw.md)
    * [ceph与openstack-keystone-v3和swift集成](docs/radosgw/swift/ceph-radosgw-with-openstack-keystone-v3.md)
* [tmp url]()
    * [(非keystone认证)swift生成并获取tmp url](docs/radosgw/swift/radosgw-swift-tmp-url-no-keystone.md)
    * [(keystone认证)swift生成并获取tmp url](docs/radosgw/swift/radosgw-swift-tmp-url-keystone.md)

## 常见问题

* [时间不一致](docs/faq/ntpdate.md)
* [修改rgw配置生效](docs/faq/rgw-conf-change.md)
* [查找服务名称](docs/faq/get-service-name.md)
* [查找用户信息](docs/radosgw/swift/get-user-info.md)
* [支持断点续传](docs/radosgw/basic.md)
* [ceph-deploy osd create时报错](docs/faq/ceph-deploy-osd-error.md)
* [swift-list临时出错](docs/faq/swift-list-error.md)
* [keystone更换mysql](docs/radosgw/swift/radosgw-swift-keystone-mysql-change.md)
* [常见问题-心远何方](docs/faq/faq-wangzhijian.md)


<!-- ## 未来

- [我的ceph探险之旅](https://b.qqbb.app/tags/ceph/)
- [Ceph Handbook](https://eiuapp/swift-handbook/)

## 相关资源

- [ceph技术工具与资源](docs/tech_resource.md) -->

