# CEPH FILE SYSTEM[](https://docs.ceph.com/en/quincy/cephfs/#ceph-file-system)

The Ceph File System, or **CephFS**, is a POSIX-compliant file system built on top of Ceph’s distributed object store, **RADOS**. CephFS endeavors to provide a state-of-the-art, multi-use, highly available, and performant file store for a variety of applications, including traditional use-cases like shared home directories, HPC scratch space, and distributed workflow shared storage.

CephFS achieves these goals through the use of some novel architectural choices. Notably, file metadata is stored in a separate RADOS pool from file data and served via a resizable cluster of *Metadata Servers*, or **MDS**, which may scale to support higher throughput metadata workloads. Clients of the file system have direct access to RADOS for reading and writing file data blocks. For this reason, workloads may linearly scale with the size of the underlying RADOS object store; that is, there is no gateway or broker mediating data I/O for clients.

Access to data is coordinated through the cluster of MDS which serve as authorities for the state of the distributed metadata cache cooperatively maintained by clients and MDS. Mutations to metadata are aggregated by each MDS into a series of efficient writes to a journal on RADOS; no metadata state is stored locally by the MDS. This model allows for coherent and rapid collaboration between clients within the context of a POSIX file system.

![../_images/cephfs-architecture.svg](https://docs.ceph.com/en/quincy/_images/cephfs-architecture.svg)

CephFS is the subject of numerous academic papers for its novel designs and contributions to file system research. It is the oldest storage interface in Ceph and was once the primary use-case for RADOS. Now it is joined by two other storage interfaces to form a modern unified storage system: RBD (Ceph Block Devices) and RGW (Ceph Object Storage Gateway).

## GETTING STARTED WITH CEPHFS[](https://docs.ceph.com/en/quincy/cephfs/#getting-started-with-cephfs)

For most deployments of Ceph, setting up a CephFS file system is as simple as:

```
ceph fs volume create <fs name>
```

The Ceph [Orchestrator](https://docs.ceph.com/en/quincy/mgr/orchestrator) will automatically create and configure MDS for your file system if the back-end deployment technology supports it (see [Orchestrator deployment table](https://docs.ceph.com/en/quincy/mgr/orchestrator/#current-implementation-status)). Otherwise, please [deploy MDS manually as needed](https://docs.ceph.com/en/quincy/cephfs/add-remove-mds).

Finally, to mount CephFS on your client nodes, see [Mount CephFS: Prerequisites](https://docs.ceph.com/en/quincy/cephfs/mount-prerequisites) page. Additionally, a command-line shell utility is available for interactive access or scripting via the [cephfs-shell](https://docs.ceph.com/en/quincy/man/8/cephfs-shell).

[ Previous](https://docs.ceph.com/en/quincy/rados/api/objclass-sdk/)[Next ](https://docs.ceph.com/en/quincy/cephfs/createfs/)

------

[![Sponsored: Read the Docs](https://media.ethicalads.io/media/images/2020/01/sticker-wtd-colors-240-180_b9300nU.png)](https://server.ethicalads.io/proxy/click/2723/daa31ee2-ad7a-4309-ba02-56bd45ef2ef3/)

[Love Documentation? Write the Docs is for people like you! Join our virtual conferences or Slack.](https://server.ethicalads.io/proxy/click/2723/daa31ee2-ad7a-4309-ba02-56bd45ef2ef3/)

*[Community Ad](https://docs.readthedocs.io/en/latest/advertising/ethical-advertising.html#community-ads)*

© Copyright 2016, Ceph authors and contributors. Licensed under Creative Commons Attribution Share Alike 3.0 (CC-BY-SA-3.0). Revision `5da2ce69`.