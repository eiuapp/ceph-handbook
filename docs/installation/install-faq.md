
## ceph-deploy 报 `sudo: no tty present and no askpass program specified`

说明，ceph用户没有能力在 `ssh ceph-client "sudo ls"` 时不输入密码。

解决：各节点运行

```
echo "admin ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/admin
sudo chmod 0440 /etc/sudoers.d/admin
```

报错日志：

```
admin@ceph-admin:~$ ceph-deploy -v new ceph-server-1 ceph-server-2 ceph-server-3 
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/admin/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /usr/bin/ceph-deploy -v new ceph-server-1 ceph-server-2 ceph-server-3
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  verbose                       : True
[ceph_deploy.cli][INFO  ]  overwrite_conf                : False
[ceph_deploy.cli][INFO  ]  quiet                         : False
[ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x7fa0449ad950>
[ceph_deploy.cli][INFO  ]  cluster                       : ceph
[ceph_deploy.cli][INFO  ]  ssh_copykey                   : True
[ceph_deploy.cli][INFO  ]  mon                           : ['ceph-server-1', 'ceph-server-2', 'ceph-server-3']
[ceph_deploy.cli][INFO  ]  func                          : <function new at 0x7fa044e0bf50>
[ceph_deploy.cli][INFO  ]  public_network                : None
[ceph_deploy.cli][INFO  ]  ceph_conf                     : None
[ceph_deploy.cli][INFO  ]  cluster_network               : None
[ceph_deploy.cli][INFO  ]  default_release               : False
[ceph_deploy.cli][INFO  ]  fsid                          : None
[ceph_deploy.new][DEBUG ] Creating new cluster named ceph
[ceph_deploy.new][INFO  ] making sure passwordless SSH succeeds
[ceph-server-1][DEBUG ] connected to host: ceph-admin 
[ceph-server-1][INFO  ] Running command: ssh -CT -o BatchMode=yes ceph-server-1
[ceph-server-1][DEBUG ] connection detected need for sudo
sudo: no tty present and no askpass program specified
[ceph_deploy][ERROR ] RuntimeError: connecting to host: ceph-server-1 resulted in errors: IOError cannot send (already closed?)
admin@ceph-admin:~$ 
admin@ceph-admin:~$ echo "admin ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/admin
[sudo] password for admin: 
admin ALL = (root) NOPASSWD:ALL
admin@ceph-admin:~$ sudo chmod 0440 /etc/sudoers.d/admin
admin@ceph-admin:~$ ceph-deploy  new ceph-server-1 ceph-server-2 ceph-server-3 
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/admin/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (2.0.1): /usr/bin/ceph-deploy new ceph-server-1 ceph-server-2 ceph-server-3
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  overwrite_conf                : False
[ceph_deploy.cli][INFO  ]  quiet                         : False
[ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x7fb781db3950>
[ceph_deploy.cli][INFO  ]  cluster                       : ceph
[ceph_deploy.cli][INFO  ]  ssh_copykey                   : True
[ceph_deploy.cli][INFO  ]  mon                           : ['ceph-server-1', 'ceph-server-2', 'ceph-server-3']
[ceph_deploy.cli][INFO  ]  func                          : <function new at 0x7fb782211f50>
[ceph_deploy.cli][INFO  ]  public_network                : None
[ceph_deploy.cli][INFO  ]  ceph_conf                     : None
[ceph_deploy.cli][INFO  ]  cluster_network               : None
[ceph_deploy.cli][INFO  ]  default_release               : False
[ceph_deploy.cli][INFO  ]  fsid                          : None
[ceph_deploy.new][DEBUG ] Creating new cluster named ceph
[ceph_deploy.new][INFO  ] making sure passwordless SSH succeeds
[ceph-server-1][DEBUG ] connected to host: ceph-admin 
[ceph-server-1][INFO  ] Running command: ssh -CT -o BatchMode=yes ceph-server-1
[ceph-server-1][DEBUG ] connection detected need for sudo
[ceph-server-1][DEBUG ] connected to host: ceph-server-1 
[ceph-server-1][DEBUG ] detect platform information from remote host
[ceph-server-1][DEBUG ] detect machine type
[ceph-server-1][DEBUG ] find the location of an executable
[ceph-server-1][INFO  ] Running command: sudo /bin/ip link show
[ceph-server-1][INFO  ] Running command: sudo /bin/ip addr show
[ceph-server-1][DEBUG ] IP addresses found: [u'192.168.0.130']
[ceph_deploy.new][DEBUG ] Resolving host ceph-server-1
[ceph_deploy.new][DEBUG ] Monitor ceph-server-1 at 192.168.0.130
[ceph_deploy.new][INFO  ] making sure passwordless SSH succeeds
[ceph-server-2][DEBUG ] connected to host: ceph-admin 
[ceph-server-2][INFO  ] Running command: ssh -CT -o BatchMode=yes ceph-server-2
[ceph-server-2][DEBUG ] connection detected need for sudo
[ceph-server-2][DEBUG ] connected to host: ceph-server-2 
[ceph-server-2][DEBUG ] detect platform information from remote host
[ceph-server-2][DEBUG ] detect machine type
[ceph-server-2][DEBUG ] find the location of an executable
[ceph-server-2][INFO  ] Running command: sudo /bin/ip link show
[ceph-server-2][INFO  ] Running command: sudo /bin/ip addr show
[ceph-server-2][DEBUG ] IP addresses found: [u'192.168.0.111']
[ceph_deploy.new][DEBUG ] Resolving host ceph-server-2
[ceph_deploy.new][DEBUG ] Monitor ceph-server-2 at 192.168.0.111
[ceph_deploy.new][INFO  ] making sure passwordless SSH succeeds
[ceph-server-3][DEBUG ] connected to host: ceph-admin 
[ceph-server-3][INFO  ] Running command: ssh -CT -o BatchMode=yes ceph-server-3
[ceph-server-3][DEBUG ] connection detected need for sudo
[ceph-server-3][DEBUG ] connected to host: ceph-server-3 
[ceph-server-3][DEBUG ] detect platform information from remote host
[ceph-server-3][DEBUG ] detect machine type
[ceph-server-3][DEBUG ] find the location of an executable
[ceph-server-3][INFO  ] Running command: sudo /bin/ip link show
[ceph-server-3][INFO  ] Running command: sudo /bin/ip addr show
[ceph-server-3][DEBUG ] IP addresses found: [u'192.168.0.105']
[ceph_deploy.new][DEBUG ] Resolving host ceph-server-3
[ceph_deploy.new][DEBUG ] Monitor ceph-server-3 at 192.168.0.105
[ceph_deploy.new][DEBUG ] Monitor initial members are ['ceph-server-1', 'ceph-server-2', 'ceph-server-3']
[ceph_deploy.new][DEBUG ] Monitor addrs are ['192.168.0.130', '192.168.0.111', '192.168.0.105']
[ceph_deploy.new][DEBUG ] Creating a random mon key...
[ceph_deploy.new][DEBUG ] Writing monitor keyring to ceph.mon.keyring...
[ceph_deploy.new][DEBUG ] Writing initial config to ceph.conf...
admin@ceph-admin:~$ 
admin@ceph-admin:~$ ssh ceph-client "sudo ls"
client
Desktop
Documents
Downloads
examples.desktop
Music
Pictures
Public
Templates
Videos
xtclient
admin@ceph-admin:~$ 
```