---
title: "v14.2.8 Nautilus released"
date: "2020-03-03"
author: "TheAnalyst"
tags:
  - "release"
  - "nautilus"
---

This is the eighth update to the Ceph Nautilus release series. This
release fixes issues across multiple subsystems. We recommend that
all users upgrade to this release.

## Notable Changes

- The default value of `bluestore_min_alloc_size_ssd` has been changed to 4KiB to improve performance across all workloads.
- The following OSD memory config options related to BlueStore cache autotuning can now be configured during runtime:
    
    > - osd\_memory\_base (default: 768 MB)
    > - osd\_memory\_cache\_min (default: 128 MB)
    > - osd\_memory\_expected\_fragmentation (default: 0.15)
    > - osd\_memory\_target (default: 4 GB)
    
    The above options can be set with:
    
    ceph config set osd <option\> <value\>
    
- The Manager now accepts `profile rbd` and `profile rbd-read-only` user caps. These caps can be used to provide users access to Manager-based RBD functionality such as `rbd perf image iostat` an `rbd perf image iotop`.
- The configuration value `osd_calc_pg_upmaps_max_stddev` used for upmap balancing
  has been removed. Instead use the mgr balancer config `upmap_max_deviation` which
  now is an integer number of PGs of deviation from the target PGs per OSD. This
  can be set with a command of the form `ceph config set mgr mgr/balancer/upmap_max_deviation 2`.
  The default `upmap_max_deviation` is 5. There are situations where CRUSH rules
  will not allow a pool to ever have perfect balanced PGs. For example, if CRUSH
  requires one replica in each of three racks, but there are fewer OSDs in one
  of the racks. In such cases, the configuration value can be increased.
- RGW: a mismatch between the bucket notification documentation and the
  actual message format was fixed. This means that any endpoints receiving
  bucket notifications will now receive the same notifications inside a
  JSON array named `Records`. Note that this does not affect pulling
  bucket notification from a subscription in a `pubsub` zone, as these
  are already wrapped inside that array.
- CephFS: multiple active MDS forward scrub is now rejected. Scrub
  currently only is permitted on a file system with a single rank.
  Reduce the ranks to one via `ceph fs set <fs_name> max_mds 1`.
- Ceph now refuses to create a file system with a default EC data pool. For
  further explanation, see: [https://docs.ceph.com/docs/nautilus/cephfs/createfs/#creating-pools](https://docs.ceph.com/docs/nautilus/cephfs/createfs/#creating-pools)
- Ceph will now issue a health warning if a RADOS pool has a `pg_num` value
  that is not a power of two. This can be resolved by adjusting the pool to
  a nearby power of two:
    
    ceph osd pool set <pool\-name\> pg\_num <new\-pg\-num\>
    
    Alternatively, the warning can be silenced with:
    
    ceph config set global mon\_warn\_on\_pool\_pg\_num\_not\_power\_of\_two false
    

## Changelog

- bluestore: common/options: bluestore 4k min\_alloc\_size for SSD ([pr#32998](https://github.com/ceph/ceph/pull/32998), Mark Nelson, Sage Weil)
- bluestore: os/bluestore: Add config observer for osd memory specific options ([pr#31852](https://github.com/ceph/ceph/pull/31852), Sridhar Seshasayee)
- bluestore: os/bluestore/BlueStore.cc: set priorities for compression stats ([pr#32845](https://github.com/ceph/ceph/pull/32845), Neha Ojha)
- bluestore: os/bluestore: default bluestore\_block\_size 1T -> 100G ([pr#32283](https://github.com/ceph/ceph/pull/32283), Sage Weil)
- build/ops: cmake: remove seastar tests from “make check” ([pr#32658](https://github.com/ceph/ceph/pull/32658), Kefu Chai)
- build/ops: install-deps,rpm: enable devtoolset-8 on aarch64 also ([issue#38892](http://tracker.ceph.com/issues/38892), [pr#32651](https://github.com/ceph/ceph/pull/32651), Kefu Chai)
- build/ops: rpm: add rpm-build to SUSE-specific make check deps ([pr#32208](https://github.com/ceph/ceph/pull/32208), Nathan Cutler)
- build/ops: switch to boost 1.72 ([pr#32441](https://github.com/ceph/ceph/pull/32441), Willem Jan Withagen, Kefu Chai)
- build/ops: tools/setup-virtualenv.sh: do not default to python2.7 ([pr#30739](https://github.com/ceph/ceph/pull/30739), Nathan Cutler)
- cephfs: cephfs-journal-tool: fix crash and usage ([pr#32913](https://github.com/ceph/ceph/pull/32913), Xiubo Li)
- cephfs: client: Add is\_dir() check before changing directory ([pr#32916](https://github.com/ceph/ceph/pull/32916), Varsha Rao)
- cephfs: client: add procession of SEEK\_HOLE and SEEK\_DATA in lseek ([pr#30764](https://github.com/ceph/ceph/pull/30764), Shen Hang)
- cephfs: client: add warning when cap != in->auth\_cap ([pr#32065](https://github.com/ceph/ceph/pull/32065), Shen Hang)
- cephfs: client: EINVAL may be returned when offset is 0 ([pr#30762](https://github.com/ceph/ceph/pull/30762), wenpengLi)
- cephfs: client: fix lazyio\_synchronize() to update file size and libcephfs: Add Tests for LazyIO ([pr#30769](https://github.com/ceph/ceph/pull/30769), Sidharth Anupkrishnan)
- cephfs: client: \_readdir\_cache\_cb() may use the readdir\_cache already clear ([issue#41148](http://tracker.ceph.com/issues/41148), [pr#30763](https://github.com/ceph/ceph/pull/30763), huanwen ren)
- cephfs: client: remove Inode.dir\_contacts field and handle bad whence value to llseek gracefully ([pr#30766](https://github.com/ceph/ceph/pull/30766), Jeff Layton)
- cephfs,common: osdc/objecter: Fix last\_sent in scientific format and add age to ops ([pr#31081](https://github.com/ceph/ceph/pull/31081), Varsha Rao)
- cephfs: disallow changing fuse\_default\_permissions option at runtime ([pr#32915](https://github.com/ceph/ceph/pull/32915), Zhi Zhang)
- cephfs: mds: add command that config individual client session ([issue#40811](http://tracker.ceph.com/issues/40811), [pr#32245](https://github.com/ceph/ceph/pull/32245), “Yan, Zheng”)
- cephfs: mds: “apply configuration changes through MDSRank” and “recall caps from quiescent sessions” and “drive cap recall while dropping cache” ([pr#30761](https://github.com/ceph/ceph/pull/30761), Patrick Donnelly, Jeff Layton)
- cephfs: mds: fix assert(omap\_num\_objs <= MAX\_OBJECTS) of OpenFileTable ([pr#32756](https://github.com/ceph/ceph/pull/32756), “Yan, Zheng”)
- cephfs: mds: fix revoking caps after after stale->resume circle ([pr#32909](https://github.com/ceph/ceph/pull/32909), “Yan, Zheng”)
- cephfs: mds: free heap memory may grow too large for some workloads ([pr#31802](https://github.com/ceph/ceph/pull/31802), Patrick Donnelly)
- cephfs: MDSMonitor: warn if a new file system is being created with an EC default data pool ([pr#32600](https://github.com/ceph/ceph/pull/32600), Patrick Donnelly)
- cephfs: mds: no assert on frozen dir when scrub path ([pr#32071](https://github.com/ceph/ceph/pull/32071), Zhi Zhang)
- cephfs: mds: note client features when rejecting client ([pr#32914](https://github.com/ceph/ceph/pull/32914), Patrick Donnelly)
- cephfs: mds/OpenFileTable: match MAX\_ITEMS\_PER\_OBJ to osd\_deep\_scrub\_large\_omap\_object\_key\_threshold ([pr#32921](https://github.com/ceph/ceph/pull/32921), Vikhyat Umrao, Varsha Rao)
- cephfs: mds: properly evaluate unstable locks when evicting client ([pr#32073](https://github.com/ceph/ceph/pull/32073), “Yan, Zheng”)
- cephfs: mds: reject forward scrubs when cluster has multiple active MDS (more than one rank) ([pr#32602](https://github.com/ceph/ceph/pull/32602), Patrick Donnelly, Milind Changire)
- cephfs: mds: reject sessionless messages ([issue#40784](http://tracker.ceph.com/issues/40784), [pr#30843](https://github.com/ceph/ceph/pull/30843), “Yan, Zheng”, Xiao Guodong, Shen Hang)
- cephfs: mds: remove unnecessary debug warning ([pr#32077](https://github.com/ceph/ceph/pull/32077), Patrick Donnelly)
- cephfs: mds returns -5(EIO) error when the deleted file does not exist ([pr#30767](https://github.com/ceph/ceph/pull/30767), huanwen ren)
- cephfs: mds: split the dir if the op makes it oversized, because some ops maybe in flight ([pr#31302](https://github.com/ceph/ceph/pull/31302), simon gao)
- cephfs: mds: tolerate no snaprealm encoded in on-disk root inode ([pr#32079](https://github.com/ceph/ceph/pull/32079), “Yan, Zheng”)
- cephfs: mgr: “mds metadata” to setup new DaemonState races with fsmap ([pr#31905](https://github.com/ceph/ceph/pull/31905), Patrick Donnelly)
- cephfs: mgr/volumes: allow setting uid, gid of subvolume and subvolume group during creation ([issue#42923](http://tracker.ceph.com/issues/42923), [pr#31741](https://github.com/ceph/ceph/pull/31741), Venky Shankar, Jos Collin)
- cephfs: mgr/volumes: fetch trash and clone entries without blocking volume access ([issue#44282](http://tracker.ceph.com/issues/44282), [pr#33526](https://github.com/ceph/ceph/pull/33526), Venky Shankar)
- cephfs: mgr/volumes: fs subvolume resize command ([pr#31332](https://github.com/ceph/ceph/pull/31332), Jos Collin)
- cephfs: mgr/volumes: misc fix and feature enhancements ([issue#42646](http://tracker.ceph.com/issues/42646), [issue#43645](http://tracker.ceph.com/issues/43645), [pr#33122](https://github.com/ceph/ceph/pull/33122), Rishabh Dave, Joshua Schmid, Venky Shankar, Ramana Raja, Jos Collin)
- cephfs: mgr/volumes: unregister job upon async threads exception ([issue#44315](http://tracker.ceph.com/issues/44315), [pr#33569](https://github.com/ceph/ceph/pull/33569), Venky Shankar)
- cephfs: mon: print FSMap regardless of file system count ([pr#32912](https://github.com/ceph/ceph/pull/32912), Patrick Donnelly)
- cephfs: pybind/mgr/volumes: idle connection drop is not working ([pr#33116](https://github.com/ceph/ceph/pull/33116), Patrick Donnelly)
- cephfs: RuntimeError: Files in flight high water is unexpectedly low (0 / 6) ([pr#33115](https://github.com/ceph/ceph/pull/33115), Patrick Donnelly)
- ceph.in: check ceph-conf returncode ([pr#31367](https://github.com/ceph/ceph/pull/31367), Dimitri Savineau)
- ceph-monstore-tool: correct the key for storing mgr\_command\_descs ([pr#33278](https://github.com/ceph/ceph/pull/33278), Kefu Chai)
- ceph-volume: add db and wal support to raw mode ([pr#32979](https://github.com/ceph/ceph/pull/32979), Sébastien Han)
- ceph-volume: add methods to pass filters to pvs, vgs and lvs commands ([pr#33217](https://github.com/ceph/ceph/pull/33217), Rishabh Dave)
- ceph-volume: add raw (–bluestore) mode ([pr#32733](https://github.com/ceph/ceph/pull/32733), Jan Fajerski, Sage Weil)
- ceph-volume: add sizing arguments to prepare ([pr#33231](https://github.com/ceph/ceph/pull/33231), Jan Fajerski)
- ceph-volume: allow raw block devices everywhere ([pr#32868](https://github.com/ceph/ceph/pull/32868), Jan Fajerski)
- ceph-volume: assume msgrV1 for all branches containing mimic ([pr#31616](https://github.com/ceph/ceph/pull/31616), Jan Fajerski)
- ceph-volume: avoid calling zap\_lv with a LV-less VG ([pr#33297](https://github.com/ceph/ceph/pull/33297), Jan Fajerski)
- ceph-volume: batch bluestore fix create\_lvs call ([pr#33232](https://github.com/ceph/ceph/pull/33232), Jan Fajerski)
- ceph-volume: batch bluestore fix create\_lvs call ([pr#33301](https://github.com/ceph/ceph/pull/33301), Jan Fajerski)
- ceph-volume/batch: fail on filtered devices when non-interactive ([pr#33202](https://github.com/ceph/ceph/pull/33202), Jan Fajerski)
- ceph-volume: Dereference symlink in lvm list ([pr#32877](https://github.com/ceph/ceph/pull/32877), Benoît Knecht)
- ceph-volume: don’t remove vg twice when zapping filestore ([pr#33337](https://github.com/ceph/ceph/pull/33337), Jan Fajerski)
- ceph-volume: finer grained availability notion in inventory ([pr#33240](https://github.com/ceph/ceph/pull/33240), Jan Fajerski)
- ceph-volume: fix has\_bluestore\_label() function ([pr#33239](https://github.com/ceph/ceph/pull/33239), Guillaume Abrioux)
- ceph-volume: fix is\_ceph\_device for lvm batch ([pr#33253](https://github.com/ceph/ceph/pull/33253), Jan Fajerski, Dimitri Savineau)
- ceph-volume: fix the integer overflow ([pr#32873](https://github.com/ceph/ceph/pull/32873), dongdong tao)
- ceph-volume: import mock.mock instead of unittest.mock (py2) ([pr#32870](https://github.com/ceph/ceph/pull/32870), Jan Fajerski)
- ceph-volume/lvm/activate.py: clarify error message: fsid refers to osd\_fsid ([pr#32864](https://github.com/ceph/ceph/pull/32864), Yaniv Kaul)
- ceph-volume: lvm/deactivate: add unit tests, remove –all ([pr#32863](https://github.com/ceph/ceph/pull/32863), Jan Fajerski)
- ceph-volume: lvm deactivate command ([pr#33209](https://github.com/ceph/ceph/pull/33209), Jan Fajerski)
- ceph-volume: make get\_devices fs location independent ([pr#33200](https://github.com/ceph/ceph/pull/33200), Jan Fajerski)
- ceph-volume: minor clean-up of “simple scan” subcommand help ([pr#32556](https://github.com/ceph/ceph/pull/32556), Michael Fritch)
- ceph-volume: pass journal\_size as Size not string ([pr#33334](https://github.com/ceph/ceph/pull/33334), Jan Fajerski)
- ceph-volume: refactor listing.py + fixes ([pr#33238](https://github.com/ceph/ceph/pull/33238), Jan Fajerski, Rishabh Dave, Guillaume Abrioux)
- ceph-volume: reject disks smaller then 5GB in inventory ([issue#40776](http://tracker.ceph.com/issues/40776), [pr#31554](https://github.com/ceph/ceph/pull/31554), Jan Fajerski)
- ceph-volume: skip osd creation when already done ([pr#33242](https://github.com/ceph/ceph/pull/33242), Guillaume Abrioux)
- ceph-volume/test: patch VolumeGroups ([pr#32558](https://github.com/ceph/ceph/pull/32558), Jan Fajerski)
- ceph-volume: use correct extents if using db-devices and >1 osds\_per\_device ([pr#32874](https://github.com/ceph/ceph/pull/32874), Fabian Niepelt)
- ceph-volume: use fsync for dd command ([pr#31553](https://github.com/ceph/ceph/pull/31553), Rishabh Dave)
- ceph-volume: use get\_device\_vgs in has\_common\_vg ([pr#33254](https://github.com/ceph/ceph/pull/33254), Jan Fajerski)
- ceph-volume: util: look for executable in $PATH ([pr#32860](https://github.com/ceph/ceph/pull/32860), Shyukri Shyukriev)
- ceph-volume/zfs: add the inventory command ([pr#31295](https://github.com/ceph/ceph/pull/31295), Willem Jan Withagen)
- common/admin\_socket: Increase socket timeouts ([pr#32063](https://github.com/ceph/ceph/pull/32063), Brad Hubbard)
- common/bl: fix the dangling last\_p issue ([pr#33277](https://github.com/ceph/ceph/pull/33277), Radoslaw Zarzynski)
- common/config: update values when they are removed via mon ([pr#32846](https://github.com/ceph/ceph/pull/32846), Sage Weil)
- common: FIPS: audit and switch some memset & bzero users ([pr#32167](https://github.com/ceph/ceph/pull/32167), Radoslaw Zarzynski)
- common: fix deadlocky inflight op visiting in OpTracker ([pr#32858](https://github.com/ceph/ceph/pull/32858), Radoslaw Zarzynski)
- common/options: remove unused ms\_msgr2\_{sign,encrypt} ([pr#31850](https://github.com/ceph/ceph/pull/31850), Ilya Dryomov)
- common/util: use ifstream to read from /proc files ([pr#32901](https://github.com/ceph/ceph/pull/32901), Kefu Chai, songweibin)
- core: auth/Crypto: fallback to /dev/urandom if getentropy() fails ([pr#31301](https://github.com/ceph/ceph/pull/31301), Kefu Chai)
- core: mon: keep v1 address type when explicitly set ([pr#32028](https://github.com/ceph/ceph/pull/32028), Ricardo Dias)
- core: mon/OSDMonitor: Fix pool set target\_size\_bytes (etc) with unit suffix ([pr#31740](https://github.com/ceph/ceph/pull/31740), Prashant D)
- core: osd/OSDMap: health alert for non-power-of-two pg\_num ([pr#30689](https://github.com/ceph/ceph/pull/30689), Sage Weil)
- crush/CrushWrapper: behave with empty weight vector ([pr#32905](https://github.com/ceph/ceph/pull/32905), Kefu Chai)
- doc/cephfs/client-auth: description and example are inconsistent ([pr#32781](https://github.com/ceph/ceph/pull/32781), Ilya Dryomov)
- doc/cephfs: improve add/remove MDS section ([issue#39620](http://tracker.ceph.com/issues/39620), [pr#31116](https://github.com/ceph/ceph/pull/31116), Patrick Donnelly)
- doc/ceph-fuse: mention -k option in ceph-fuse man page ([pr#30765](https://github.com/ceph/ceph/pull/30765), Rishabh Dave)
- doc/ceph-volume: initial docs for zfs/inventory and zfs/api ([pr#32746](https://github.com/ceph/ceph/pull/32746), Willem Jan Withagen)
- doc: remove invalid option mon\_pg\_warn\_max\_per\_osd ([pr#31300](https://github.com/ceph/ceph/pull/31300), zhang daolong)
- doc/\_templates/page.html: redirect to etherpad ([pr#32248](https://github.com/ceph/ceph/pull/32248), Neha Ojha)
- doc: wrong datatype describing crush\_rule ([pr#32254](https://github.com/ceph/ceph/pull/32254), Kefu Chai)
- global: disable THP for Ceph daemons ([pr#31646](https://github.com/ceph/ceph/pull/31646), Patrick Donnelly, Mark Nelson)
- kv: fix shutdown vs async compaction ([pr#32715](https://github.com/ceph/ceph/pull/32715), Sage Weil)
- librbd: diff iterate with fast-diff now correctly includes parent ([pr#32469](https://github.com/ceph/ceph/pull/32469), Jason Dillaman)
- librbd: fix rbd\_open\_by\_id, rbd\_open\_by\_id\_read\_only ([pr#32837](https://github.com/ceph/ceph/pull/32837), yangjun)
- librbd: remove pool objects when removing a namespace ([pr#32839](https://github.com/ceph/ceph/pull/32839), Jason Dillaman)
- librbd: skip stale child with non-existent pool for list descendants ([pr#32841](https://github.com/ceph/ceph/pull/32841), songweibin)
- librbd: support compression allocation hints to the OSD ([pr#32842](https://github.com/ceph/ceph/pull/32842), Jason Dillaman)
- mgr: add ‘rbd’ profiles to support ‘rbd\_support’ module commands ([pr#32086](https://github.com/ceph/ceph/pull/32086), Jason Dillaman)
- mgr/alerts: simple health alerts ([pr#30820](https://github.com/ceph/ceph/pull/30820), Sage Weil)
- mgr: Balancer fixes ([pr#31956](https://github.com/ceph/ceph/pull/31956), Neha Ojha, Kefu Chai, David Zafman)
- mgr/DaemonServer: fix ‘osd ok-to-stop’ for EC pools ([pr#32844](https://github.com/ceph/ceph/pull/32844), Sage Weil)
- mgr/dashboard: add debug mode, and accept expected exception when SSL handshaking ([pr#31190](https://github.com/ceph/ceph/pull/31190), Kefu Chai, Ernesto Puerta, Joshua Schmid)
- mgr/dashboard: block mirroring page results in internal server error ([pr#32133](https://github.com/ceph/ceph/pull/32133), Jason Dillaman)
- mgr/dashboard: check embedded Grafana dashboard references ([issue#40008](http://tracker.ceph.com/issues/40008), [pr#31808](https://github.com/ceph/ceph/pull/31808), Kiefer Chang)
- mgr/dashboard: check if user has config-opt permissions ([pr#32827](https://github.com/ceph/ceph/pull/32827), Alfonso Martínez)
- mgr/dashboard: Cross sign button not working for some modals ([pr#32012](https://github.com/ceph/ceph/pull/32012), Ricardo Marques)
- mgr/dashboard: Dashboard can’t handle self-signed cert on Grafana API ([pr#31792](https://github.com/ceph/ceph/pull/31792), Volker Theile)
- mgr/dashboard: disable ‘Add Capability’ button in rgw user edit ([pr#32930](https://github.com/ceph/ceph/pull/32930), Alfonso Martínez)
- mgr/dashboard: fix restored RBD image naming issue ([pr#31810](https://github.com/ceph/ceph/pull/31810), Kiefer Chang)
- mgr/dashboard: grafana charts match time picker selection ([pr#31999](https://github.com/ceph/ceph/pull/31999), Alfonso Martínez)
- mgr/dashboard,grafana: remove shortcut menu ([pr#31980](https://github.com/ceph/ceph/pull/31980), Ernesto Puerta)
- mgr/dashboard: Handle always-on Ceph Manager modules correctly ([pr#31782](https://github.com/ceph/ceph/pull/31782), Volker Theile)
- mgr/dashboard: Hardening accessing the metadata ([pr#32128](https://github.com/ceph/ceph/pull/32128), Volker Theile)
- mgr/dashboard: iSCSI targets not available if any gateway is down (and more…) ([pr#32304](https://github.com/ceph/ceph/pull/32304), Ricardo Marques)
- mgr/dashboard: KeyError on dashboard reload ([pr#32233](https://github.com/ceph/ceph/pull/32233), Patrick Seidensal)
- mgr/dashboard: key-value-table doesn’t render booleans ([pr#31789](https://github.com/ceph/ceph/pull/31789), Patrick Seidensal)
- mgr/dashboard: Remove compression mode unset in pool from ([pr#31784](https://github.com/ceph/ceph/pull/31784), Stephan Müller)
- mgr/dashboard: show “Rename” in header & button when renaming RBD ([pr#31779](https://github.com/ceph/ceph/pull/31779), Alfonso Martínez)
- mgr/dashboard: sort monitors by open sessions correctly ([pr#31791](https://github.com/ceph/ceph/pull/31791), Alfonso Martínez)
- mgr/dashboard: Standby Dashboards don’t handle all requests properly ([pr#32299](https://github.com/ceph/ceph/pull/32299), Volker Theile)
- mgr/dashboard: Trim IQN on iSCSI target form ([pr#31942](https://github.com/ceph/ceph/pull/31942), Ricardo Marques)
- mgr/dashboard: Unable to set boolean values to false when default is true ([pr#31941](https://github.com/ceph/ceph/pull/31941), Ricardo Marques)
- mgr/dashboard: Using wrong identifiers in RGW user/bucket datatables ([pr#32888](https://github.com/ceph/ceph/pull/32888), Volker Theile)
- mgr/devicehealth: ensure we don’t store empty objects ([pr#31735](https://github.com/ceph/ceph/pull/31735), Sage Weil)
- mgr/devicehealth: fix telemetry stops sending device reports after 48 hours ([pr#33346](https://github.com/ceph/ceph/pull/33346), Yaarit Hatuka, Sage Weil)
- mgr: drop reference to msg on return ([pr#33498](https://github.com/ceph/ceph/pull/33498), Patrick Donnelly)
- mgr/MgrClient: fix open condition ([pr#32769](https://github.com/ceph/ceph/pull/32769), Sage Weil)
- mgr/pg\_autoscaler: calculate pool\_pg\_target using pool size ([pr#33170](https://github.com/ceph/ceph/pull/33170), Dan van der Ster)
- mgr/pg\_autoscaler: default to pg\_num\[\_min\] = 16 ([pr#32069](https://github.com/ceph/ceph/pull/32069), Sage Weil)
- mgr/pg\_autoscaler: default to pg\_num\[\_min\] = 32 ([pr#32931](https://github.com/ceph/ceph/pull/32931), Neha Ojha)
- mgr/pg\_autoscaler: implement shutdown method ([pr#32068](https://github.com/ceph/ceph/pull/32068), Patrick Donnelly)
- mgr/pg\_autoscaler: only generate target\_\* health warnings if targets set ([pr#32067](https://github.com/ceph/ceph/pull/32067), Sage Weil)
- mgr/prometheus: assign a value to osd\_dev\_node when obj\_store is not filestore or bluestore ([pr#31556](https://github.com/ceph/ceph/pull/31556), jiahuizeng)
- mgr/prometheus: report per-pool pg states ([pr#33157](https://github.com/ceph/ceph/pull/33157), Aleksei Zakharov)
- mgr/telemetry: anonymizing smartctl report itself ([pr#33082](https://github.com/ceph/ceph/pull/33082), Yaarit Hatuka)
- mgr/telemetry: check get\_metadata return val ([pr#33095](https://github.com/ceph/ceph/pull/33095), Yaarit Hatuka)
- mgr/telemetry: split entity\_name only once (handle ids with dots) ([pr#33168](https://github.com/ceph/ceph/pull/33168), Dan Mick)
- mgr/zabbix: Adds possibility to send data to multiple zabbix servers ([pr#30009](https://github.com/ceph/ceph/pull/30009), slivik, Jakub Sliva)
- mon/ConfigMonitor: fix handling of NO\_MON\_UPDATE settings ([pr#32856](https://github.com/ceph/ceph/pull/32856), Sage Weil)
- mon/ConfigMonitor: only propose if leader ([pr#33155](https://github.com/ceph/ceph/pull/33155), Sage Weil)
- mon: Don’t put session during feature change ([pr#33152](https://github.com/ceph/ceph/pull/33152), Brad Hubbard)
- mon: elector: return after triggering a new election ([pr#33007](https://github.com/ceph/ceph/pull/33007), Greg Farnum)
- monitoring: wait before firing osd full alert ([pr#32070](https://github.com/ceph/ceph/pull/32070), Patrick Seidensal)
- mon/MgrMonitor.cc: add always\_on\_modules to the output of “ceph mgr module ls” ([pr#32997](https://github.com/ceph/ceph/pull/32997), Neha Ojha)
- mon/MgrMonitor.cc: warn about missing mgr in a cluster with osds ([pr#33142](https://github.com/ceph/ceph/pull/33142), Neha Ojha)
- mon/OSDMonitor: Don’t update mon cache settings if rocksdb is not used ([pr#32520](https://github.com/ceph/ceph/pull/32520), Sridhar Seshasayee, Sage Weil)
- mon/OSDMonitor: fix format error ceph osd stat –format json ([pr#32062](https://github.com/ceph/ceph/pull/32062), Zheng Yin)
- mon/PGMap.h: disable network stats in dump\_osd\_stats ([pr#32466](https://github.com/ceph/ceph/pull/32466), Neha Ojha, David Zafman)
- mon: remove the restriction of address type in init\_with\_hosts ([pr#31844](https://github.com/ceph/ceph/pull/31844), Hao Xiong)
- mon/Session: only index osd ids >= 0 ([pr#32908](https://github.com/ceph/ceph/pull/32908), Sage Weil)
- mount.ceph: give a hint message when no mds is up or cluster is laggy ([pr#32910](https://github.com/ceph/ceph/pull/32910), Xiubo Li)
- mount.ceph: remove arbitrary limit on size of name= option ([pr#32807](https://github.com/ceph/ceph/pull/32807), Jeff Layton)
- msg: async/net\_handler.cc: Fix compilation ([pr#31736](https://github.com/ceph/ceph/pull/31736), Carlos Valiente)
- osd: add osd\_fast\_shutdown option (default true) ([pr#32743](https://github.com/ceph/ceph/pull/32743), Sage Weil)
- osd: Allow 64-char hostname to be added as the “host” in CRUSH ([pr#33147](https://github.com/ceph/ceph/pull/33147), Michal Skalski)
- osd: Diagnostic logging for upmap cleaning ([pr#32716](https://github.com/ceph/ceph/pull/32716), David Zafman)
- osd/OSD: enhance osd numa affinity compatibility ([pr#32843](https://github.com/ceph/ceph/pull/32843), luo rixin, Dai zhiwei)
- osd/PeeringState.cc: don’t let num\_objects become negative ([pr#32857](https://github.com/ceph/ceph/pull/32857), Neha Ojha)
- osd/PeeringState.cc: skip peer\_purged when discovering all missing ([pr#32847](https://github.com/ceph/ceph/pull/32847), Neha Ojha)
- osd/PeeringState: do not exclude up from acting\_recovery\_backfill ([pr#32064](https://github.com/ceph/ceph/pull/32064), Nathan Cutler, xie xingguo)
- osd/PrimaryLogPG: skip obcs that don’t exist during backfill scan\_range ([pr#31028](https://github.com/ceph/ceph/pull/31028), Sage Weil)
- osd: set affinity for \*all\* threads ([pr#31359](https://github.com/ceph/ceph/pull/31359), Sage Weil)
- osd: set collection pool opts on collection create, pg load ([pr#32123](https://github.com/ceph/ceph/pull/32123), Sage Weil)
- osd: Use physical ratio for nearfull (doesn’t include backfill resserve) ([pr#32773](https://github.com/ceph/ceph/pull/32773), David Zafman)
- pybind/mgr: Cancel output color control ([pr#31697](https://github.com/ceph/ceph/pull/31697), Zheng Yin)
- rbd: creating thick-provision image progress percent info exceeds 100% ([pr#32840](https://github.com/ceph/ceph/pull/32840), Xiangdong Mu)
- rbd: librbd: don’t call refresh from mirror::GetInfoRequest state machine ([pr#32900](https://github.com/ceph/ceph/pull/32900), Mykola Golub)
- rbd-mirror: clone v2 mirroring improvements ([pr#31518](https://github.com/ceph/ceph/pull/31518), Mykola Golub)
- rbd-mirror: fix ‘rbd mirror status’ asok command output ([pr#32447](https://github.com/ceph/ceph/pull/32447), Mykola Golub)
- rbd-mirror: make logrotate work ([pr#32593](https://github.com/ceph/ceph/pull/32593), Mykola Golub)
    
- rgw: add bucket permission verify when copy obj ([pr#31089](https://github.com/ceph/ceph/pull/31089), NancySu05)
- rgw: Adding ‘iam’ namespace for Role and User Policy related REST APIs ([pr#32437](https://github.com/ceph/ceph/pull/32437), Pritha Srivastava)
- rgw: adding mfa code validation when bucket versioning status is changed ([pr#32759](https://github.com/ceph/ceph/pull/32759), Pritha Srivastava)
- rgw: add num\_shards to radosgw-admin bucket stats ([pr#31182](https://github.com/ceph/ceph/pull/31182), Paul Emmerich)
- rgw: allow reshard log entries for non-existent buckets to be cancelled ([pr#32056](https://github.com/ceph/ceph/pull/32056), J. Eric Ivancich)
- rgw: auto-clean reshard queue entries for non-existent buckets ([pr#32055](https://github.com/ceph/ceph/pull/32055), J. Eric Ivancich)
- rgw: build\_linked\_oids\_for\_bucket and build\_buckets\_instance\_index should return negative value if it fails ([pr#32820](https://github.com/ceph/ceph/pull/32820), zhangshaowen)
- rgw: crypt: permit RGW-AUTO/default with SSE-S3 headers ([pr#31862](https://github.com/ceph/ceph/pull/31862), Matt Benjamin)
- rgw: data sync markers include timestamp from datalog entry ([pr#32819](https://github.com/ceph/ceph/pull/32819), Casey Bodley)
- rgw\_file: avoid string::front() on empty path ([pr#33008](https://github.com/ceph/ceph/pull/33008), Matt Benjamin)
- rgw: fix a bug that bucket instance obj can’t be removed after resharding completed ([pr#32822](https://github.com/ceph/ceph/pull/32822), zhang Shaowen)
- rgw: fix an endless loop error when to show usage ([pr#31684](https://github.com/ceph/ceph/pull/31684), lvshuhua)
- rgw: fix bugs in listobjectsv1 ([pr#32239](https://github.com/ceph/ceph/pull/32239), Albin Antony)
- rgw: fix compile errors with boost 1.70 ([pr#31289](https://github.com/ceph/ceph/pull/31289), Casey Bodley)
- rgw: fix data consistency error casued by rgw sent timeout ([pr#32821](https://github.com/ceph/ceph/pull/32821), 李纲彬82225)
- rgw: fix list versions starts with version\_id=null ([pr#30743](https://github.com/ceph/ceph/pull/30743), Tianshan Qu)
- rgw: fix one part of the bulk delete(RGWDeleteMultiObj\_ObjStore\_S3) fails but no error messages ([pr#33151](https://github.com/ceph/ceph/pull/33151), Snow Si)
- rgw: fix opslog operation field as per Amazon s3 ([issue#20978](http://tracker.ceph.com/issues/20978), [pr#32834](https://github.com/ceph/ceph/pull/32834), Jiaying Ren)
- rgw: fix refcount tags to match and update object’s idtag ([pr#30741](https://github.com/ceph/ceph/pull/30741), J. Eric Ivancich)
- rgw: fix rgw crash when token is not base64 encode ([pr#32050](https://github.com/ceph/ceph/pull/32050), yuliyang)
- rgw: gc remove tag after all sub io finish ([issue#40903](http://tracker.ceph.com/issues/40903), [pr#30733](https://github.com/ceph/ceph/pull/30733), Tianshan Qu)
- rgw: Incorrectly calling ceph::buffer::list::decode\_base64 in bucket policy ([pr#32832](https://github.com/ceph/ceph/pull/32832), GaryHyg)
- rgw: maybe coredump when reload operator happened ([pr#33149](https://github.com/ceph/ceph/pull/33149), Richard Bai(白学余))
- rgw: move forward marker even in case of many rgw.none indexes ([pr#32824](https://github.com/ceph/ceph/pull/32824), Ilsoo Byun)
- rgw multisite: fixes for concurrent version creation ([pr#32057](https://github.com/ceph/ceph/pull/32057), Or Friedmann, Casey Bodley)
- rgw: prevent bucket reshard scheduling if bucket is resharding ([pr#31298](https://github.com/ceph/ceph/pull/31298), J. Eric Ivancich)
- rgw/pubsub: fix records/event json format to match documentation ([pr#32221](https://github.com/ceph/ceph/pull/32221), Yuval Lifshitz)
- rgw: radosgw-admin: sync status displays id of shard furthest behind ([pr#32818](https://github.com/ceph/ceph/pull/32818), Casey Bodley)
- rgw: return error if lock log shard fails ([pr#32825](https://github.com/ceph/ceph/pull/32825), zhangshaowen)
- rgw/rgw\_rest\_conn.h: fix build with clang ([pr#32489](https://github.com/ceph/ceph/pull/32489), Bernd Zeimetz)
- rgw: Select the std::bitset to resolv ambiguity ([pr#32504](https://github.com/ceph/ceph/pull/32504), Willem Jan Withagen)
- rgw: support radosgw-admin zone/zonegroup placement get command ([pr#32835](https://github.com/ceph/ceph/pull/32835), jiahuizeng)
- rgw: the http response code of delete bucket should not be 204-no-content ([pr#32833](https://github.com/ceph/ceph/pull/32833), Chang Liu)
- rgw: update s3-test download code for s3-test tasks ([pr#32229](https://github.com/ceph/ceph/pull/32229), Ali Maredia)
- rgw: update the hash source for multipart entries during resharding ([pr#33183](https://github.com/ceph/ceph/pull/33183), dongdong tao)
- rgw: url encode common prefixes for List Objects response ([pr#32058](https://github.com/ceph/ceph/pull/32058), Abhishek Lekshmanan)
- rgw: when resharding store progress json ([pr#31683](https://github.com/ceph/ceph/pull/31683), Mark Kogan, Mark Nelson)
- selinux: Allow ceph to read udev db ([pr#32259](https://github.com/ceph/ceph/pull/32259), Boris Ranto)
