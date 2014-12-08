# Prerequisite

* Vagrant with Virtualbox
* Vagrant plugins
  * [vagrant-hostmanager] https://github.com/smdahlen/vagrant-hostmanager
  * [vagrant-cachier] https://github.com/fgrehm/vagrant-cachier
* nfs-kernel-server for vagrant-hostmaster

# Init

* vagrant up
* vagrant ssh ceph-admin
  * cd ceph-cluster
  * ceph-deploy install ceph-admin ceph-mon-1 ceph-mon-2 ceph-mon-3 ceph-osd-1 ceph-osd-2 ceph-osd-3 
  * ceph-deploy new ceph-mon-1
  * ceph-deploy --overwrite-conf mon create ceph-mon-1
  * ceph-deploy gatherkeys ceph-mon-1
  * sudo sudo cp ceph.client.admin.keyring /etc/ceph/
  * vi ceph.conf
```
 [global]
 fsid = f445846b-fdcc-4e8d-8499-37d9ace02f89
 mon_initial_members = ceph-mon-1,ceph-mon-2,ceph-mon-3
 mon_host = 172.21.12.21,172.21.12.22,172.21.12.23
 auth_cluster_required = cephx
 auth_service_required = cephx
 auth_client_required = cephx
 filestore_xattr_use_omap = true
 osd_journal_size = 1024
 public_network = 172.21.12.0/24
 ```
  * ceph-deploy --overwrite-conf config push ceph-admin ceph-mon-1 ceph-mon-2 ceph-mon-3
  * ceph -s
```
    cluster f445846b-fdcc-4e8d-8499-37d9ace02f89
     health HEALTH_ERR 192 pgs stuck inactive; 192 pgs stuck unclean; no osds
     monmap e1: 1 mons at {ceph-mon-1=172.21.12.21:6789/0}, election epoch 2, quorum 0 ceph-mon-1
     osdmap e1: 0 osds: 0 up, 0 in
      pgmap v2: 192 pgs, 3 pools, 0 bytes data, 0 objects
            0 kB used, 0 kB / 0 kB avail
                 192 creating
```
  * ceph-deploy --overwrite-conf mon create ceph-mon-2 ceph-mon-3
  * ceph -s
```
    cluster f445846b-fdcc-4e8d-8499-37d9ace02f89
     health HEALTH_ERR 192 pgs stuck inactive; 192 pgs stuck unclean; no osds
     monmap e3: 3 mons at {ceph-mon-1=172.21.12.21:6789/0,ceph-mon-2=172.21.12.22:6789/0,ceph-mon-3=172.21.12.23:6789/0}, election epoch 8, quorum 0,1,2 ceph-mon-1,ceph-mon-2,ceph-mon-3
     osdmap e1: 0 osds: 0 up, 0 in
      pgmap v2: 192 pgs, 3 pools, 0 bytes data, 0 objects
            0 kB used, 0 kB / 0 kB avail
                 192 creating
```
  * ceph-deploy  disk zap ceph-osd-1:sdb ceph-osd-2:sdb ceph-osd-3:sdb
  * ceph-deploy  --overwrite-conf osd create ceph-osd-1:sdb ceph-osd-2:sdb ceph-osd-3:sdb
  * ceph -s
```
    cluster f445846b-fdcc-4e8d-8499-37d9ace02f89
     health HEALTH_OK
     monmap e3: 3 mons at {ceph-mon-1=172.21.12.21:6789/0,ceph-mon-2=172.21.12.22:6789/0,ceph-mon-3=172.21.12.23:6789/0}, election epoch 8, quorum 0,1,2 ceph-mon-1,ceph-mon-2,ceph-mon-3
     osdmap e13: 3 osds: 3 up, 3 in
      pgmap v20: 192 pgs, 3 pools, 0 bytes data, 0 objects
            105 MB used, 21365 MB / 21470 MB avail
                 192 active+clean
```
