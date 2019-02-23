
根据 https://github.com/carmstrong/multinode-ceph-vagrant 安装 ceph 开发环境

## env

- os: windows 10
- 命令行： git-bash或者cmder或者cmd
- virtual: 6.0.4
- vagrant: 2.2.3
- vagrant box: ubuntu/bionic64
- ceph：V12.2.11 LUMINOUS
- ceph-deploy：1.5.38


架构部署

| 部件   |    OSD      |  Monitor | Managers | MDSs |
|:----------|:-------------|:-------------|:-------------|:------|
| ceph-admin |     |   |  |  |
| ceph-client |     |   |  |  |
| ceph-server-1 |     |  Y | Y |  |
| ceph-server-2 |   Y  |   |  |  |
| ceph-server-3 |   Y  |   |  |  |

## clone repo

```
git clone https://github.com/carmstrong/multinode-ceph-vagrant && cd multinode-ceph-vagrant
```
## vagrant plugin

```
vagrant plugin install vagrant-cachier
vagrant plugin install vagrant-hostmanager
```

## vagrant box add

提前下载好官方的 ubuntu/bionic64 

```
vagrant box list
vagrant box add ./ubuntu-18.04/bionic-server-cloudimg-amd64-vagrant.box  --name ubuntu/bionic64
vagrant box list
```

## Add your Vagrant key to the SSH agent

```
DELL@DESKTOP-MQ9CENU MINGW64 /f/tom/ceph/multinode-ceph-vagrant (master)
$ ssh-add -k ~/.vagrant.d/insecure_private_key
Could not open a connection to your authentication agent.

DELL@DESKTOP-MQ9CENU MINGW64 /f/tom/ceph/multinode-ceph-vagrant (master)
$ ssh-add -K ~/.vagrant.d/insecure_private_key
Could not open a connection to your authentication agent.

DELL@DESKTOP-MQ9CENU MINGW64 /f/tom/ceph/multinode-ceph-vagrant (master)
$
```

这里明显没有成功。

是不是因为我的环境中，之前有过 `~/.vagrant.d/insecure_private_key` ? 

我这里先跳过这一步

```
DELL@DESKTOP-MQ9CENU MINGW64 /f/tom/ceph/multinode-ceph-vagrant (master)
$ ls ~/.vagrant.d/insecure_private_key
/c/Users/DELL/.vagrant.d/insecure_private_key

DELL@DESKTOP-MQ9CENU MINGW64 /f/tom/ceph/multinode-ceph-vagrant (master)
$
```

其实这一步，就是为了让后面的 `vagrant ssh ceph-admin` 等生效。

而 `vagrant ssh ceph-admin` 的原理，就是使用 `~/.vagrant.d/insecure_private_key` 去登录 ceph-admin 中的 vagrant 用户。

你可以用这个原理，在 xshell 中导入 `~/.vagrant.d/insecure_private_key` ，然后登录 ceph-admin，ceph-client，ceph-server-1 等机器 中的 vagrant 用户。

## Start the VMs

This instructs Vagrant to start the VMs and install `ceph-deploy` on the admin machine.

```console
$ vagrant up
```

## Create the cluster

We'll create a simple cluster and make sure it's healthy. Then, we'll expand it.

First, we need to get an interactive shell on the admin machine:

```console
$ vagrant ssh ceph-admin
```

The `ceph-deploy` tool will write configuration files and logs to the current directory. So, let's create a directory for the new cluster:

```console
vagrant@ceph-admin:~$ mkdir test-cluster && cd test-cluster
```

```
vagrant@ceph-admin:~/test-cluster$ ceph-deploy --version
1.5.38
vagrant@ceph-admin:~/test-cluster$ 
```

Let's prepare the machines:

```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy new ceph-server-1 ceph-server-2 ceph-server-3
```

这个时候可能会报错：

```shell
vagrant@ceph-admin:~/test-cluster$ ceph-deploy new ceph-server-1 ceph-server-2 ceph-server-3
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/vagrant/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (1.5.38): /usr/bin/ceph-deploy new ceph-server-1 ceph-server-2 ceph-server-3
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  overwrite_conf                : False
[ceph_deploy.cli][INFO  ]  quiet                         : False
[ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x7f3c7ba5afc8>
[ceph_deploy.cli][INFO  ]  cluster                       : ceph
[ceph_deploy.cli][INFO  ]  ssh_copykey                   : True
[ceph_deploy.cli][INFO  ]  mon                           : ['ceph-server-1', 'ceph-server-2', 'ceph-server-3']
[ceph_deploy.cli][INFO  ]  func                          : <function new at 0x7f3c7bed1578>
[ceph_deploy.cli][INFO  ]  public_network                : None
[ceph_deploy.cli][INFO  ]  ceph_conf                     : None
[ceph_deploy.cli][INFO  ]  cluster_network               : None
[ceph_deploy.cli][INFO  ]  default_release               : False
[ceph_deploy.cli][INFO  ]  fsid                          : None
[ceph_deploy.new][DEBUG ] Creating new cluster named ceph
[ceph_deploy.new][INFO  ] making sure passwordless SSH succeeds
[ceph-server-1][DEBUG ] connected to host: ceph-admin
[ceph-server-1][INFO  ] Running command: ssh -CT -o BatchMode=yes ceph-server-1
[ceph_deploy.new][WARNIN] could not connect via SSH
[ceph_deploy.new][INFO  ] creating a passwordless id_rsa.pub key file
[ceph_deploy.new][DEBUG ] connected to host: ceph-admin
[ceph_deploy.new][INFO  ] Running command: ssh-keygen -t rsa -N  -f /home/vagrant/.ssh/id_rsa
[ceph_deploy.new][DEBUG ] Generating public/private rsa key pair.
[ceph_deploy.new][DEBUG ] Your identification has been saved in /home/vagrant/.ssh/id_rsa.
[ceph_deploy.new][DEBUG ] Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub.
[ceph_deploy.new][DEBUG ] The key fingerprint is:
[ceph_deploy.new][DEBUG ] SHA256:JXlwZhhC4h1ous7ywvgUEZLfqt6fUFVcS09NcTB17iI vagrant@ceph-admin
[ceph_deploy.new][DEBUG ] The key's randomart image is:
[ceph_deploy.new][DEBUG ] +---[RSA 2048]----+
[ceph_deploy.new][DEBUG ] |... .o+ ++=o .===|
[ceph_deploy.new][DEBUG ] |.. ooo ooB. +  +o|
[ceph_deploy.new][DEBUG ] | ..+. ..o o. .  .|
[ceph_deploy.new][DEBUG ] |  o.. .  +     . |
[ceph_deploy.new][DEBUG ] |  .o .  S   E . .|
[ceph_deploy.new][DEBUG ] |  o..        . . |
[ceph_deploy.new][DEBUG ] |o+..             |
[ceph_deploy.new][DEBUG ] |=o+ . .          |
[ceph_deploy.new][DEBUG ] |.*o..o           |
[ceph_deploy.new][DEBUG ] +----[SHA256]-----+
[ceph_deploy.new][INFO  ] will connect again with password prompt
The authenticity of host 'ceph-server-1 (172.21.12.12)' can't be established.
ECDSA key fingerprint is SHA256:E9BwMCErur/NOG8wmkAUbl1j/q4gYtIKA2d1NBN0NQU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ceph-server-1,172.21.12.12' (ECDSA) to the list of known hosts.
vagrant@ceph-server-1: Permission denied (publickey).
[ceph_deploy][ERROR ] RuntimeError: connecting to host: ceph-server-1 resulted in errors: HostNotFound ceph-server-1

vagrant@ceph-admin:~/test-cluster$
```

在这种情况下，我先建立一个 snapshot

```
DELL@DESKTOP-MQ9CENU MINGW64 /f/tom/ceph/multinode-ceph-vagrant (master)
$ vagrant snapshot save ceph-admin init
==> ceph-admin: Snapshotting the machine as 'init'...
==> ceph-admin: Snapshot saved! You can restore the snapshot at any time by
==> ceph-admin: using `vagrant snapshot restore`. You can delete it using
==> ceph-admin: `vagrant snapshot delete`.

DELL@DESKTOP-MQ9CENU MINGW64 /f/tom/ceph/multinode-ceph-vagrant (master)
$ vagrant snapshot list ceph-admin
init

DELL@DESKTOP-MQ9CENU MINGW64 /f/tom/ceph/multinode-ceph-vagrant (master)
$ vagrant snapshot save ceph-client init
==> ceph-client: Snapshotting the machine as 'init'...
==> ceph-client: Snapshot saved! You can restore the snapshot at any time by
==> ceph-client: using `vagrant snapshot restore`. You can delete it using
==> ceph-client: `vagrant snapshot delete`.

DELL@DESKTOP-MQ9CENU MINGW64 /f/tom/ceph/multinode-ceph-vagrant (master)
$ vagrant snapshot save ceph-server-1 init
==> ceph-server-1: Snapshotting the machine as 'init'...
==> ceph-server-1: Snapshot saved! You can restore the snapshot at any time by
==> ceph-server-1: using `vagrant snapshot restore`. You can delete it using
==> ceph-server-1: `vagrant snapshot delete`.

DELL@DESKTOP-MQ9CENU MINGW64 /f/tom/ceph/multinode-ceph-vagrant (master)
$ vagrant snapshot save ceph-server-2 init
==> ceph-server-2: Snapshotting the machine as 'init'...
==> ceph-server-2: Snapshot saved! You can restore the snapshot at any time by
==> ceph-server-2: using `vagrant snapshot restore`. You can delete it using
==> ceph-server-2: `vagrant snapshot delete`.

DELL@DESKTOP-MQ9CENU MINGW64 /f/tom/ceph/multinode-ceph-vagrant (master)
$ vagrant snapshot save ceph-server-3 init
==> ceph-server-3: Snapshotting the machine as 'init'...
==> ceph-server-3: Snapshot saved! You can restore the snapshot at any time by
==> ceph-server-3: using `vagrant snapshot restore`. You can delete it using
==> ceph-server-3: `vagrant snapshot delete`.
```

看一下之前的那个报错。应该就是从 ceph-admin 连接 ceph-server-1 的时候 出错了。

看过程，应该是 ceph-admin 自己建立了一个 id_rsa 然后，希望通过自己建立的这个 id_rsa 去登录 ceph-server-1 。
```
vagrant@ceph-admin:~/test-cluster$ ll ~/.ssh/
total 24
drwx------ 2 vagrant vagrant 4096 Feb 23 02:11 ./
drwxr-xr-x 6 vagrant vagrant 4096 Feb 23 02:11 ../
-rw-r--r-- 1 vagrant vagrant  409 Feb 19 16:00 authorized_keys
-rw------- 1 vagrant vagrant 1679 Feb 23 02:11 id_rsa
-rw-r--r-- 1 vagrant vagrant  400 Feb 23 02:11 id_rsa.pub
-rw-r--r-- 1 vagrant vagrant  666 Feb 23 02:21 known_hosts
vagrant@ceph-admin:~/test-cluster$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCtVkEjX/fomoDRPyBgGlmLX24yjXz83zxeB5Zb7Hmk2bpJNyEKnHmexuoCfxGkulQEtVGzsFxO4fdx1pLWZsJAkGIvAq7KZ41LLAeOCPSxB8CaGLGXdtpnDUgyGdGk88qOfWo+vCHqY2M7Vt3HKbtypb6lXsLBOsNiQStWavSPN1afipF9weuxn8nNvJZNKkHVbfzI6C0dVnFxzBTsEoyS1GBi1pTGuZyVmY8pYj7i47A3qVidUQuFIkQ9VZXIZ5RGuw/AaAvM4KaQ+qilu1NN0yCihHlQFLf8QtRvJou2JqYPDMHvNqQ1qDb056an1KOkoAvkAToHEnykyTNsb5Cn vagrant@ceph-admin
vagrant@ceph-admin:~/test-cluster$
```

那我们去 ceph-server-1 中去看一下，是否在 其下的 .ssh/authorized_keys 文件内容

```
vagrant@ceph-server-1:~$ ll .ssh/
total 12
drwx------ 2 vagrant vagrant 4096 Feb 19 16:00 ./
drwxr-xr-x 5 vagrant vagrant 4096 Feb 22 10:26 ../
-rw-r--r-- 1 vagrant vagrant  409 Feb 19 16:00 authorized_keys
vagrant@ceph-server-1:~$ cat .ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key
vagrant@ceph-server-1:~$
```
很明显，ceph-server-1 的 /home/vagrant/.ssh/authorized_keys 文件内容中并没有 ceph-admin 的 /home/vagrant/.ssh/id_rsa.pub 的内容。这自然就无法登录。

那么问题出在哪里了？

回头再看一下[预检/CEPH 节点安装/允许无密码 SSH 登录](http://docs.ceph.com/docs/master/start/quick-start-preflight/#enable-password-less-ssh) ，这需要 ceph-admin 的 允许无密码 SSH 登录，需要设置 /home/vagrant/.ssh/config 文件。

而 ceph-server-1 中没有这个 config 文件。
```
vagrant@ceph-server-1:~$ ls .ssh/
authorized_keys
vagrant@ceph-server-1:~$ 
```

那么我们手工完成一下吧。我这里通过写入各 ceph 节点 的 /home/vagrant/.ssh/authorized_keys 文件完成。下以 ceph-server-2 为例：
```
vagrant@ceph-server-2:~$ echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCtVkEjX/fomoDRPyBgGlmLX24yjXz83zxeB5Zb7Hmk2bpJNyEKnHmexuoCfxGkulQEtVGzsFxO4fdx1pLWZsJAkGIvAq7KZ41LLAeOCPSxB8CaGLGXdtpnDUgyGdGk88qOfWo+vCHqY2M7Vt3HKbtypb6lXsLBOsNiQStWavSPN1afipF9weuxn8nNvJZNKkHVbfzI6C0dVnFxzBTsEoyS1GBi1pTGuZyVmY8pYj7i47A3qVidUQuFIkQ9VZXIZ5RGuw/AaAvM4KaQ+qilu1NN0yCihHlQFLf8QtRvJou2JqYPDMHvNqQ1qDb056an1KOkoAvkAToHEnykyTNsb5Cn vagrant@ceph-admin" >> .ssh/authorized_keys
```

再`ceph-deploy new ceph-server-1 ceph-server-2 ceph-server-3`生成了 `ceph.conf` 和 `ceph.mon.keyring`

```
vagrant@ceph-admin:~/test-cluster$ ceph-deploy new ceph-server-1 ceph-server-2 ceph-server-3
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/vagrant/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (1.5.38): /usr/bin/ceph-deploy new ceph-server-1 ceph-server-2 ceph-server-3
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  overwrite_conf                : False
[ceph_deploy.cli][INFO  ]  quiet                         : False
[ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x7f8dd8634fc8>
[ceph_deploy.cli][INFO  ]  cluster                       : ceph
[ceph_deploy.cli][INFO  ]  ssh_copykey                   : True
[ceph_deploy.cli][INFO  ]  mon                           : ['ceph-server-1', 'ceph-server-2', 'ceph-server-3']
[ceph_deploy.cli][INFO  ]  func                          : <function new at 0x7f8dd8aab578>
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
[ceph-server-1][DEBUG ] IP addresses found: [u'172.21.12.12', u'10.0.2.15']
[ceph_deploy.new][DEBUG ] Resolving host ceph-server-1
[ceph_deploy.new][DEBUG ] Monitor ceph-server-1 at 172.21.12.12
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
[ceph-server-2][DEBUG ] IP addresses found: [u'172.21.12.13', u'10.0.2.15']
[ceph_deploy.new][DEBUG ] Resolving host ceph-server-2
[ceph_deploy.new][DEBUG ] Monitor ceph-server-2 at 172.21.12.13
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
[ceph-server-3][DEBUG ] IP addresses found: [u'10.0.2.15', u'172.21.12.14']
[ceph_deploy.new][DEBUG ] Resolving host ceph-server-3
[ceph_deploy.new][DEBUG ] Monitor ceph-server-3 at 172.21.12.14
[ceph_deploy.new][DEBUG ] Monitor initial members are ['ceph-server-1', 'ceph-server-2', 'ceph-server-3']
[ceph_deploy.new][DEBUG ] Monitor addrs are ['172.21.12.12', '172.21.12.13', '172.21.12.14']
[ceph_deploy.new][DEBUG ] Creating a random mon key...
[ceph_deploy.new][DEBUG ] Writing monitor keyring to ceph.mon.keyring...
[ceph_deploy.new][DEBUG ] Writing initial config to ceph.conf...
vagrant@ceph-admin:~/test-cluster$ ls
ceph-deploy-ceph.log  ceph.conf  ceph.mon.keyring
vagrant@ceph-admin:~/test-cluster$ 
```


Now, we have to change a default setting. For our initial cluster, we are only going to have two [object storage daemons](http://docs.ceph.com/docs/master/architecture/#the-ceph-storage-cluster). We need to tell Ceph to allow us to achieve an `active + clean` state with just two Ceph OSDs. Add `osd pool default size = 2` to `./ceph.conf`.

Because we're dealing with multiple VMs sharing the same host, we can expect to see more clock skew. We can tell Ceph that we'd like to tolerate slightly more clock skew by adding the following section to `ceph.conf`:
```
mon_clock_drift_allowed = 1
```

After these few changes, the file should look similar to:

```
[global]
fsid = 7acac25d-2bd8-4911-807e-e35377e741bf
mon_initial_members = ceph-server-1, ceph-server-2, ceph-server-3
mon_host = 172.21.12.12,172.21.12.13,172.21.12.14
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
osd pool default size = 2
mon_clock_drift_allowed = 1
```

实操作：

```
vagrant@ceph-admin:~/test-cluster$ echo "osd pool default size = 2" >> ceph.conf 
vagrant@ceph-admin:~/test-cluster$ echo "mon_clock_drift_allowed = 1" >> ceph.conf 
vagrant@ceph-admin:~/test-cluster$ cat ceph.conf 
[global]
fsid = c49c3fb1-cfb9-4f16-adfd-ba93b44bd3b5
mon_initial_members = ceph-server-1, ceph-server-2, ceph-server-3
mon_host = 172.21.12.12,172.21.12.13,172.21.12.14
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx

osd pool default size = 2
mon_clock_drift_allowed = 1
vagrant@ceph-admin:~/test-cluster$ 
```

## Install Ceph

We're finally ready to install!

Note here that we specify the Ceph release we'd like to install, which is [luminous](http://docs.ceph.com/docs/master/releases/luminous/).

```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy install --release=luminous ceph-admin ceph-server-1 ceph-server-2 ceph-server-3 ceph-client
```

运行命令吧。日志放在 `./multinode-ceph-vagrant-install-log-part-2.log` 和 `./multinode-ceph-vagrant-install-log-part-3.log` 中

最后的输出如下：

```
[ceph-client][DEBUG ] Created symlink /etc/systemd/system/multi-user.target.wants/ceph-mds.target → /lib/systemd/system/ceph-mds.target.
[ceph-client][DEBUG ] Created symlink /etc/systemd/system/ceph.target.wants/ceph-mds.target → /lib/systemd/system/ceph-mds.target.
[ceph-client][DEBUG ] Processing triggers for libc-bin (2.27-3ubuntu1) ...
[ceph-client][DEBUG ] Processing triggers for ureadahead (0.100.0-20) ...
[ceph-client][DEBUG ] Processing triggers for systemd (237-3ubuntu10.13) ...
[ceph-client][INFO  ] Running command: sudo ceph --version
[ceph-client][DEBUG ] ceph version 12.2.8 (ae699615bac534ea496ee965ac6192cb7e0e07c0) luminous (stable)
vagrant@ceph-admin:~/test-cluster$ 
```

`ceph -v` 一下

```
vagrant@ceph-admin:~/test-cluster$ ceph -v
ceph version 12.2.8 (ae699615bac534ea496ee965ac6192cb7e0e07c0) luminous (stable)
vagrant@ceph-admin:~/test-cluster$ 
```

## Configure monitor and OSD services

Next, we add a monitor node:

```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy mon create-initial
```

生成了几个新文件

```
vagrant@ceph-admin:~/test-cluster$ ls
ceph-deploy-ceph.log  ceph.bootstrap-mds.keyring  ceph.bootstrap-mgr.keyring  ceph.bootstrap-osd.keyring  ceph.bootstrap-rgw.keyring  ceph.client.admin.keyring  ceph.conf  ceph.mon.keyring
vagrant@ceph-admin:~/test-cluster$ 
```

具体日志在 `./multinode-ceph-vagrant-install-log-part-4.log` 中

And our two OSDs. For these, we need to log into the server machines directly:

```console
vagrant@ceph-admin:~/test-cluster$ ssh ceph-server-2 "sudo mkdir /var/local/osd0 && sudo chown ceph:ceph /var/local/osd0"
```

```console
vagrant@ceph-admin:~/test-cluster$ ssh ceph-server-3 "sudo mkdir /var/local/osd1 && sudo chown ceph:ceph /var/local/osd1"
```

Now we can prepare and activate the OSDs:

```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy osd prepare ceph-server-2:/var/local/osd0 ceph-server-3:/var/local/osd1
vagrant@ceph-admin:~/test-cluster$ ceph-deploy osd activate ceph-server-2:/var/local/osd0 ceph-server-3:/var/local/osd1
```

具体日志在 ``./multinode-ceph-vagrant-install-log-part-5.log` 中

## Configuration and status

We can copy our config file and admin key to all the nodes, so each one can use the `ceph` CLI.

```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy admin ceph-admin ceph-server-1 ceph-server-2 ceph-server-3 ceph-client
```

具体如下：

```
vagrant@ceph-admin:~/test-cluster$ ceph-deploy admin ceph-admin ceph-server-1 ceph-server-2 ceph-server-3 ceph-client
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/vagrant/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (1.5.38): /usr/bin/ceph-deploy admin ceph-admin ceph-server-1 ceph-server-2 ceph-server-3 ceph-client
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  overwrite_conf                : False
[ceph_deploy.cli][INFO  ]  quiet                         : False
[ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x7fb642da7a28>
[ceph_deploy.cli][INFO  ]  cluster                       : ceph
[ceph_deploy.cli][INFO  ]  client                        : ['ceph-admin', 'ceph-server-1', 'ceph-server-2', 'ceph-server-3', 'ceph-client']
[ceph_deploy.cli][INFO  ]  func                          : <function admin at 0x7fb6436ac0c8>
[ceph_deploy.cli][INFO  ]  ceph_conf                     : None
[ceph_deploy.cli][INFO  ]  default_release               : False
[ceph_deploy.admin][DEBUG ] Pushing admin keys and conf to ceph-admin
[ceph-admin][DEBUG ] connection detected need for sudo
[ceph-admin][DEBUG ] connected to host: ceph-admin 
[ceph-admin][DEBUG ] detect platform information from remote host
[ceph-admin][DEBUG ] detect machine type
[ceph-admin][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
[ceph_deploy.admin][DEBUG ] Pushing admin keys and conf to ceph-server-1
[ceph-server-1][DEBUG ] connection detected need for sudo
[ceph-server-1][DEBUG ] connected to host: ceph-server-1 
[ceph-server-1][DEBUG ] detect platform information from remote host
[ceph-server-1][DEBUG ] detect machine type
[ceph-server-1][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
[ceph_deploy.admin][DEBUG ] Pushing admin keys and conf to ceph-server-2
[ceph-server-2][DEBUG ] connection detected need for sudo
[ceph-server-2][DEBUG ] connected to host: ceph-server-2 
[ceph-server-2][DEBUG ] detect platform information from remote host
[ceph-server-2][DEBUG ] detect machine type
[ceph-server-2][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
[ceph_deploy.admin][DEBUG ] Pushing admin keys and conf to ceph-server-3
[ceph-server-3][DEBUG ] connection detected need for sudo
[ceph-server-3][DEBUG ] connected to host: ceph-server-3 
[ceph-server-3][DEBUG ] detect platform information from remote host
[ceph-server-3][DEBUG ] detect machine type
[ceph-server-3][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
[ceph_deploy.admin][DEBUG ] Pushing admin keys and conf to ceph-client
[ceph-client][DEBUG ] connection detected need for sudo
[ceph-client][DEBUG ] connected to host: ceph-client 
[ceph-client][DEBUG ] detect platform information from remote host
[ceph-client][DEBUG ] detect machine type
[ceph-client][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
vagrant@ceph-admin:~/test-cluster$ ls
ceph-deploy-ceph.log  ceph.bootstrap-mds.keyring  ceph.bootstrap-mgr.keyring  ceph.bootstrap-osd.keyring  ceph.bootstrap-rgw.keyring  ceph.client.admin.keyring  ceph.conf  ceph.mon.keyring
vagrant@ceph-admin:~/test-cluster$ ls /etc/ceph/ -l
total 12
-rw------- 1 root root  63 Feb 23 03:43 ceph.client.admin.keyring
-rw-r--r-- 1 root root 313 Feb 23 03:43 ceph.conf
-rw-r--r-- 1 root root  92 Jan 29 09:32 rbdmap
-rw------- 1 root root   0 Feb 23 03:43 tmpXyXDGy
vagrant@ceph-admin:~/test-cluster$ date
Sat Feb 23 03:43:31 UTC 2019
vagrant@ceph-admin:~/test-cluster$ ls
ceph-deploy-ceph.log  ceph.bootstrap-mds.keyring  ceph.bootstrap-mgr.keyring  ceph.bootstrap-osd.keyring  ceph.bootstrap-rgw.keyring  ceph.client.admin.keyring  ceph.conf  ceph.mon.keyring
vagrant@ceph-admin:~/test-cluster$
```

看 /etc/ceph/ 文件时间，知道，是刚刚写入的。

然后去 ceph-server-3 中检查一下。

```
vagrant@ceph-server-3:~$ ceph -v
ceph version 12.2.8 (ae699615bac534ea496ee965ac6192cb7e0e07c0) luminous (stable)
vagrant@ceph-server-3:~$ ls /etc/ceph/ -l
total 12
-rw------- 1 root root  63 Feb 23 03:43 ceph.client.admin.keyring
-rw-r--r-- 1 root root 313 Feb 23 03:43 ceph.conf
-rw-r--r-- 1 root root  92 Jan 29 09:32 rbdmap
-rw------- 1 root root   0 Feb 23 03:34 tmpADlUqE
vagrant@ceph-server-3:~$ date
Sat Feb 23 03:43:48 UTC 2019
vagrant@ceph-server-3:~$ 
```

没毛病。

We also should make sure the keyring is readable:

```console
vagrant@ceph-admin:~/test-cluster$ sudo chmod +r /etc/ceph/ceph.client.admin.keyring
vagrant@ceph-admin:~/test-cluster$ ssh ceph-server-1 sudo chmod +r /etc/ceph/ceph.client.admin.keyring
vagrant@ceph-admin:~/test-cluster$ ssh ceph-server-2 sudo chmod +r /etc/ceph/ceph.client.admin.keyring
vagrant@ceph-admin:~/test-cluster$ ssh ceph-server-3 sudo chmod +r /etc/ceph/ceph.client.admin.keyring
```

Finally, check on the health of the cluster:

我的与文档的不同。

* 文档的：

```console
vagrant@ceph-admin:~/test-cluster$ ceph health
```

You should see something similar to this once it's healthy:

```console
vagrant@ceph-admin:~/test-cluster$ ceph health
HEALTH_OK
vagrant@ceph-admin:~/test-cluster$ ceph -s
    cluster 18197927-3d77-4064-b9be-bba972b00750
     health HEALTH_OK
     monmap e2: 3 mons at {ceph-server-1=172.21.12.12:6789/0,ceph-server-2=172.21.12.13:6789/0,ceph-server-3=172.21.12.14:6789/0}, election epoch 6, quorum 0,1,2 ceph-server-1,ceph-server-2,ceph-server-3
     osdmap e9: 2 osds: 2 up, 2 in
      pgmap v13: 192 pgs, 3 pools, 0 bytes data, 0 objects
            12485 MB used, 64692 MB / 80568 MB avail
                 192 active+clean
```

Notice that we have two OSDs (`osdmap e9: 2 osds: 2 up, 2 in`) and all of the [placement groups](http://docs.ceph.com/docs/master/rados/operations/placement-groups/) (pgs) are reporting as `active+clean`.

* 我的：

```
vagrant@ceph-admin:~/test-cluster$ ceph health
HEALTH_WARN no active mgr
vagrant@ceph-admin:~/test-cluster$ ceph -s
  cluster:
    id:     c49c3fb1-cfb9-4f16-adfd-ba93b44bd3b5
    health: HEALTH_WARN
            no active mgr
 
  services:
    mon: 3 daemons, quorum ceph-server-1,ceph-server-2,ceph-server-3
    mgr: no daemons active
    osd: 2 osds: 2 up, 2 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0B
    usage:   0B used, 0B / 0B avail
    pgs:     
 
vagrant@ceph-admin:~/test-cluster$ 
```

说明我们没有安装 mgr , 那安装一下：

安装在 ceph-server-1 .

```
vagrant@ceph-admin:~/test-cluster$ ceph-deploy mgr create ceph-server-1
[ceph_deploy.conf][DEBUG ] found configuration file at: /home/vagrant/.cephdeploy.conf
[ceph_deploy.cli][INFO  ] Invoked (1.5.38): /usr/bin/ceph-deploy mgr create ceph-server-1
[ceph_deploy.cli][INFO  ] ceph-deploy options:
[ceph_deploy.cli][INFO  ]  username                      : None
[ceph_deploy.cli][INFO  ]  verbose                       : False
[ceph_deploy.cli][INFO  ]  mgr                           : [('ceph-server-1', 'ceph-server-1')]
[ceph_deploy.cli][INFO  ]  overwrite_conf                : False
[ceph_deploy.cli][INFO  ]  subcommand                    : create
[ceph_deploy.cli][INFO  ]  quiet                         : False
[ceph_deploy.cli][INFO  ]  cd_conf                       : <ceph_deploy.conf.cephdeploy.Conf instance at 0x7f79d921e2d8>
[ceph_deploy.cli][INFO  ]  cluster                       : ceph
[ceph_deploy.cli][INFO  ]  func                          : <function mgr at 0x7f79d91e69b0>
[ceph_deploy.cli][INFO  ]  ceph_conf                     : None
[ceph_deploy.cli][INFO  ]  default_release               : False
[ceph_deploy.mgr][DEBUG ] Deploying mgr, cluster ceph hosts ceph-server-1:ceph-server-1
[ceph-server-1][DEBUG ] connection detected need for sudo
[ceph-server-1][DEBUG ] connected to host: ceph-server-1 
[ceph-server-1][DEBUG ] detect platform information from remote host
[ceph-server-1][DEBUG ] detect machine type
[ceph_deploy.mgr][INFO  ] Distro info: Ubuntu 18.04 bionic
[ceph_deploy.mgr][DEBUG ] remote host will use systemd
[ceph_deploy.mgr][DEBUG ] deploying mgr bootstrap to ceph-server-1
[ceph-server-1][DEBUG ] write cluster configuration to /etc/ceph/{cluster}.conf
[ceph-server-1][WARNIN] mgr keyring does not exist yet, creating one
[ceph-server-1][DEBUG ] create a keyring file
[ceph-server-1][DEBUG ] create path if it doesn't exist
[ceph-server-1][INFO  ] Running command: sudo ceph --cluster ceph --name client.bootstrap-mgr --keyring /var/lib/ceph/bootstrap-mgr/ceph.keyring auth get-or-create mgr.ceph-server-1 mon allow profile mgr osd allow * mds allow * -o /var/lib/ceph/mgr/ceph-ceph-server-1/keyring
[ceph-server-1][INFO  ] Running command: sudo systemctl enable ceph-mgr@ceph-server-1
[ceph-server-1][WARNIN] Created symlink /etc/systemd/system/ceph-mgr.target.wants/ceph-mgr@ceph-server-1.service → /lib/systemd/system/ceph-mgr@.service.
[ceph-server-1][INFO  ] Running command: sudo systemctl start ceph-mgr@ceph-server-1
[ceph-server-1][INFO  ] Running command: sudo systemctl enable ceph.target
vagrant@ceph-admin:~/test-cluster$ ceph -s
  cluster:
    id:     c49c3fb1-cfb9-4f16-adfd-ba93b44bd3b5
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph-server-1,ceph-server-2,ceph-server-3
    mgr: ceph-server-1(active)
    osd: 2 osds: 2 up, 2 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0B
    usage:   2.00GiB used, 18.0GiB / 20GiB avail
    pgs:     
 
vagrant@ceph-admin:~/test-cluster$ 
```

Congratulations!

好了，到这里，我们一个基本的ceph环境的已经搭建好了！

## Expanding the cluster （没有实践）

To more closely model a production cluster, we're going to add one more OSD daemon and a [Ceph Metadata Server](http://docs.ceph.com/docs/master/man/8/ceph-mds/). We'll also add monitors to all hosts instead of just one.

### Add an OSD
```console
vagrant@ceph-admin:~/test-cluster$ ssh ceph-server-1 "sudo mkdir /var/local/osd2 && sudo chown ceph:ceph /var/local/osd2"
```

Now, from the admin node, we prepare and activate the OSD:
```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy osd prepare ceph-server-1:/var/local/osd2
vagrant@ceph-admin:~/test-cluster$ ceph-deploy osd activate ceph-server-1:/var/local/osd2
```

Watch the rebalancing:

```console
vagrant@ceph-admin:~/test-cluster$ ceph -w
```

You should eventually see it return to an `active+clean` state, but this time with 3 OSDs:

```console
vagrant@ceph-admin:~/test-cluster$ ceph -w
    cluster 18197927-3d77-4064-b9be-bba972b00750
     health HEALTH_OK
     monmap e2: 3 mons at {ceph-server-1=172.21.12.12:6789/0,ceph-server-2=172.21.12.13:6789/0,ceph-server-3=172.21.12.14:6789/0}, election epoch 30, quorum 0,1,2 ceph-server-1,ceph-server-2,ceph-server-3
     osdmap e38: 3 osds: 3 up, 3 in
      pgmap v415: 192 pgs, 3 pools, 0 bytes data, 0 objects
            18752 MB used, 97014 MB / 118 GB avail
                 192 active+clean
```

### Add metadata server

Let's add a metadata server to server1:

```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy mds create ceph-server-1
```

## Add more monitors（没有实践）

We add monitors to servers 2 and 3.

```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy mon create ceph-server-2 ceph-server-3
```

Watch the quorum status, and ensure it's happy:

```console
vagrant@ceph-admin:~/test-cluster$ ceph quorum_status --format json-pretty
```

## Install Ceph Object Gateway（没有实践）

TODO

## Play around!

Now that we have everything set up, let's actually use the cluster. We'll use the ceph-client machine for this.

### Create a block device

```console
$ vagrant ssh ceph-client
vagrant@ceph-client:~$ sudo rbd create foo --size 4096 -m ceph-server-1
vagrant@ceph-client:~$ sudo rbd map foo --pool rbd --name client.admin -m ceph-server-1
vagrant@ceph-client:~$ sudo mkfs.ext4 -m0 /dev/rbd/rbd/foo
vagrant@ceph-client:~$ sudo mkdir /mnt/ceph-block-device
vagrant@ceph-client:~$ sudo mount /dev/rbd/rbd/foo /mnt/ceph-block-device
```

### Create a mount with Ceph FS

TODO

### Store a blob object

TODO

## Cleanup（没有实践）

When you're all done, tell Vagrant to destroy the VMs.

```console
$ vagrant destroy -f
```




