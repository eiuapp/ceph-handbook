
根据 https://github.com/carmstrong/multinode-ceph-vagrant 安装 ceph 开发环境

## env

- os: windows 10
- virtual: 6.0.4
- vagrant: 2.2.3
- vagrant box: ubuntu/bionic64
- ceph：V12.2.11 LUMINOUS
- 命令行： git-bash或者cmder或者cmd

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

Let's prepare the machines:

```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy new ceph-server-1 ceph-server-2 ceph-server-3
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

## Install Ceph

We're finally ready to install!

Note here that we specify the Ceph release we'd like to install, which is [luminous](http://docs.ceph.com/docs/master/releases/luminous/).

```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy install --release=luminous ceph-admin ceph-server-1 ceph-server-2 ceph-server-3 ceph-client
```

## Configure monitor and OSD services

Next, we add a monitor node:

```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy mon create-initial
```

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

## Configuration and status

We can copy our config file and admin key to all the nodes, so each one can use the `ceph` CLI.

```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy admin ceph-admin ceph-server-1 ceph-server-2 ceph-server-3 ceph-client
```

We also should make sure the keyring is readable:

```console
vagrant@ceph-admin:~/test-cluster$ sudo chmod +r /etc/ceph/ceph.client.admin.keyring
vagrant@ceph-admin:~/test-cluster$ ssh ceph-server-1 sudo chmod +r /etc/ceph/ceph.client.admin.keyring
vagrant@ceph-admin:~/test-cluster$ ssh ceph-server-2 sudo chmod +r /etc/ceph/ceph.client.admin.keyring
vagrant@ceph-admin:~/test-cluster$ ssh ceph-server-3 sudo chmod +r /etc/ceph/ceph.client.admin.keyring
```

Finally, check on the health of the cluster:

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

Congratulations!

## Expanding the cluster

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

## Add more monitors

We add monitors to servers 2 and 3.

```console
vagrant@ceph-admin:~/test-cluster$ ceph-deploy mon create ceph-server-2 ceph-server-3
```

Watch the quorum status, and ensure it's happy:

```console
vagrant@ceph-admin:~/test-cluster$ ceph quorum_status --format json-pretty
```

## Install Ceph Object Gateway

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

## Cleanup

When you're all done, tell Vagrant to destroy the VMs.

```console
$ vagrant destroy -f
```




