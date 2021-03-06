
本文主要讲 硬件推荐

## step

Ceph 为普通硬件设计

通常，我们推荐在一台机器上只运行一种类型的守护进程。我们推荐把使用数据集群的进程（如 OpenStack 、 CloudStack 等）安装在别的机器上。

### CPU && RAM

| 部件   |    OSD      |  Monitor | Managers | MDSs |
|:----------|:-------------|:-------------|:-------------|:------|
| host(台) |  >= 3  | >= 3 | >= 2|  |
| cpu(u) |  >= 2  |   |  | >=4  |
| ram(GB) | bluestor >= (3-5) 与 PG 数相关  | >= (1+2)  | >= (1+2) | >= 1  |

### 数据存储

#### 硬盘驱动器

- 单个驱动器容量越大，其对应的 OSD 所需内存就越大，特别是在重均衡、回填、恢复期间。根据经验， 1TB 的存储空间大约需要 1GB 内存。
- Ceph 最佳实践指示，你应该分别在单独的硬盘运行操作系统、 OSD 数据和 OSD 日志。
    - 我们推荐独立的驱动器用于安装操作系统和软件，另外每个 OSD 守护进程占用一个驱动器。
    - 允许你在每块硬盘驱动器上运行多个 OSD ，但这会导致资源竞争并降低总体吞吐量。

#### SSD

- SSD 用于对象存储太昂贵了，但是把 OSD 的日志存到 SSD 、把对象数据存储到独立的硬盘可以明显提升性能。 

#### 控制器

硬盘控制器对写吞吐量也有显著影响，要谨慎地选择，以免产生性能瓶颈。

#### 其他注意事项

如果每台主机运行多个 OSD ，也得保证内核是最新的。


### 网络

建议每台机器最少两个千兆网卡，分别用于公网（前端）和集群网络（后端）。

### 故障域
故障域指任何导致不能访问一个或多个 OSD 的故障，可以是主机上停止的进程、硬盘故障、操作系统崩溃、有问题的网卡、损坏的电源、断网、断电等等。规划硬件需求时，要在多个需求间寻求平衡点，像付出很多努力减少故障域带来的成本削减、隔离每个潜在故障域增加的成本。

### 最低硬件推荐

直接看 [这里](http://docs.ceph.com/docs/master/start/hardware-recommendations/#minimum-hardware-recommendations)

对比我们的 env ，应该是

- cpu和RAM相对于存储小了。
- 网卡没达到 千兆网卡。

但是希望不影响我们的研究工作。

## ref

- http://docs.ceph.com/docs/master/start/hardware-recommendations/