# CephFS 使用

[blackpiglet](https://www.jianshu.com/u/73bb3eeb6889)

2018.09.19 15:23:06字数 830阅读 23,563

Ceph

之前介绍了 RBD 的使用方法，有了 RBD，远程磁盘挂载的问题就解决了，但 RBD 的问题是不能多个主机共享一个磁盘，如果有一份数据很多客户端都要读写该怎么办呢？这时 CephFS 作为文件系统存储解决方案就派上用场了。

## 1. CephFS 安装

在已经部署好的 Ceph 集群上安装 CephFS 还是比较简单的，首先需要安装 MDS，原因是 CephFS 需要统一的元数据管理，所以 MDS 必须要有。

```bash
$ ceph-deploy --overwrite-conf mds create  <cephfs-master>
```

剩下的就是 pool 相关的处理的了：

```bash
$ ceph osd pool create cephfs_data 1024
$ ceph osd pool create cephfs_metadata 100
$ ceph fs new cephfs cephfs_metadata cephfs_data
```

这样 CephFS 就已经安装好了，执行下面的命令验证一下状态，看到 up 和 active 就表示状态正常。

```bash
$ ceph fs ls
name: cephfs, metadata pool: cephfs_metadata, data pools: [cephfs_data ]
$ ceph mds stat
cephfs-1/1/1 up  {0=cephfs-master1=up:active}
```

## 2. CephFS 物理机挂载

接下来尝试一下远程挂载刚刚创建出来的 CephFS，首先还是要在客户机上安装 Ceph，这步和远程挂载 RBD 是一样的：

```bash
# On Linux client
$ echo "<user-name> ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/<user-name>
$ sudo chmod 0440 /etc/sudoers.d/<user-name>
$ sudo apt-get install -y python
# On Ceph master
$ ceph-deploy install <Linux-client-IP>
$ ceph-deploy admin <Linux-client-IP>
```

然后获取 Ceph admin 用户的密钥，就是 `<admin-key>` 对应的部分。

```bash
$ ceph auth get client.admin
exported keyring for client.admin
[client.admin]
    key = <admin-key>
    caps mds = "allow *"
    caps mgr = "allow *"
    caps mon = "allow *"
    caps osd = "allow *"
```

接下来创建挂载目录，并挂载，注意替换 `<monitor-ip>` 和 `<admin-key>`，分别对应 Ceph monitor 所在的 IP 和刚刚在上一步获取的 admin 密钥。

```bash
$ sudo mkdir /mnt/cephfs
$ sudo mount -t ceph <monitor-ip>:6789:/ /mnt/cephfs/ -o name=admin,secret=<admin-key>
```

至此就已经挂载成功了，可以在挂载的目录下创建新目录，写入新文件，在其他同样挂载该 CephFS 的服务器上，也能看到同样的变化。

如果想做到开机启动就挂载 CephFS，请参考 [使用ceph的文件存储CephFS](https://blog.csdn.net/zzq900503/article/details/80470785)

## 3. CephFS 用户隔离

上一步已经实现了远程挂载，和文件共享，但如果多个产品同时使用一个 CephFS，如果数据都互相可见，显然是不够安全的，那么就要做隔离，咱们接下来就来看看 CephFS 是如何做到这点的。

CephFS 这个文件系统还是统一的，所以隔离是在用户层面做的，具体的做法是：限定某个用户只能访问某个目录。

下面这个命令创建了一个叫 **bruce** 的用户，这个用户只能访问目录 **/bruce**，数据存储在 pool cephfs_data 中。

```bash
$ ceph auth get-or-create client.bruce mon 'allow r' mds 'allow r, allow rw path=/bruce' osd 'allow rw pool=cephfs_data'
```

如果要做进一步的隔离，想让不通用户的数据存储在不同的 pool，可以用命令将 pool 加入的 CephFS 中，再用命令指定，加入 pool 的命令如下：

```bash
$ ceph mds add_data_pool bruce
$ ceph fs ls 
.... data pools: [cephfs_data bruce ]
```

挂载方式和 admin 用户挂载一样：

```bash
$ sudo mount -t ceph 10.19.250.136:6789:/ /mnt/bruce -o name=bruce,secret=AQCt8qBbx4XGKBAACWG5lQHRX7FTo0nVZCYxNA==
```

挂载后，可以看到其他目录，但没有操作权限，只能在 /mnt/bruce/bruce 下操作。

```bash
$ /mnt/bruce$ ls
abc bruce
$ /mnt/bruce$ cd abc
$ /mnt/bruce/abc$ sudo mkdir test
mkdir: cannot create directory 'test': Permission denied
$ /mnt/bruce/abc$ cd ../bruce
$ /mnt/bruce/bruce$ mkdir test
$ /mnt/bruce/bruce$ ls
test
```

## 4. Kubernetes 集群使用 CephFS

首先把 Ceph 用户的密钥以 secret 形式存储起来，下面的命令是获取 admin 用户的密钥，如果使用其他用户，可以把 **admin** 替换为要使用的用户名即可。

```bash
$ ceph auth get-key client.admin | base64
QVFEMDFWVmFWdnp6TFJBQWFUVVJ5VVp3STlBZDN1WVlGUkwrVkE9PQ==

$ cat ceph-secret.yaml 
apiVersion: v1
kind: Secret
metadata:
  name: ceph-secret
data:
  key: QVFEMDFWVmFWdnp6TFJBQWFUVVJ5VVp3STlBZDN1WVlGUkwrVkE9PQ==
```

接下来创建 PV：

```bash
$ cat cephfs-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: cephfs-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  cephfs:
    monitors:
      - <monitor1-id>:6789
      - <monitor2-id>:6789
    user: admin
    secretRef:
      name: ceph-secret
    readOnly: false
  persistentVolumeReclaimPolicy: Recycle
```

最后创建 PVC：

```bash
$ cat cephfs-pvc.yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: cephfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```

PV 和 PVC 都有以后，使用方式和普通的 PV 和 PVC 无异。

## 5. 参考文档

- [初试 Kubernetes 集群使用 CephFS 文件存储](https://blog.csdn.net/aixiaoyang168/article/details/79056864)
- [在Kubernetes上使用CephFS作为文件存储](https://blog.frognew.com/2018/04/kubernetes-pv-cephfs.html)
- [FILE LAYOUTS](http://docs.ceph.com/docs/jewel/cephfs/file-layouts/)
- [[ceph-users\] can I create multiple pools for cephfs](http://lists.ceph.com/pipermail/ceph-users-ceph.com/2016-October/013596.html)
- [使用ceph的文件存储CephFS](https://blog.csdn.net/zzq900503/article/details/80470785)



[CephFS 使用 - 简书 (jianshu.com)](https://www.jianshu.com/p/c22ff79c4452)