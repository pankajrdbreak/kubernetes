# ETCD backup and restore k8s cluster 
1. Install etcd in your cluster
2. Take backup of cluster
3. Delete the resources to restore them again
4. Restore the cluster
For command we can take help from https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

1. To take backup of cluster command is given below
```console
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>
```
Here 
--cacert=/etc/kubernetes/pki/etcd/ca.crt
--cert==/etc/kubernetes/pki/etcd/server.crt
--key=/etc/kubernetes/pki/etcd/server.key
<backup-file-location>= /opt/snapshot-pre-boot.db  ..You can give any name to db file
  You can find the above paths from description of etcd pod
```console
  kmaster@kmaster:~$ kubectl -n kube-system describe pod etcd-kmaster 
Name:                 etcd-kmaster
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 kmaster/192.168.246.128
Start Time:           Sun, 06 Feb 2022 12:13:04 +0530
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://192.168.246.128:2379
                      kubernetes.io/config.hash: deecb8271f4253cae37dc0c697c927fd
                      kubernetes.io/config.mirror: deecb8271f4253cae37dc0c697c927fd
                      kubernetes.io/config.seen: 2022-02-01T13:20:24.996742023+05:30
                      kubernetes.io/config.source: file
Status:               Running
IP:                   192.168.246.128
IPs:
  IP:           192.168.246.128
Controlled By:  Node/kmaster
Containers:
  etcd:
    Container ID:  docker://db0a2282e149135e7160a577c2051c8e0173c4be63e686c8fb3de1e248732dab
    Image:         k8s.gcr.io/etcd:3.4.13-0
    Image ID:      docker-pullable://k8s.gcr.io/etcd@sha256:4ad90a11b55313b182afc186b9876c8e891531b8db4c9bf1541953021618d0e2
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://192.168.246.128:2379
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --initial-advertise-peer-urls=https://192.168.246.128:2380
      --initial-cluster=kmaster=https://192.168.246.128:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://192.168.246.128:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://192.168.246.128:2380
      --name=kmaster
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    State:          Running
      Started:      Sun, 06 Feb 2022 12:28:45 +0530
    Last State:     Terminated
      Reason:       Error
      Exit Code:    137
      Started:      Sun, 06 Feb 2022 12:13:07 +0530
      Finished:     Sun, 06 Feb 2022 12:28:45 +0530
    Ready:          True
    Restart Count:  2
    Requests:
      cpu:        100m
      memory:     100Mi
    Liveness:     http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:      http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:  <none>
    Mounts:
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
      /var/lib/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/etcd
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:
  Type    Reason          Age                From     Message
  ----    ------          ----               ----     -------
  Normal  SandboxChanged  88m                kubelet  Pod sandbox changed, it will be killed and re-created.
  Normal  Pulled          72m (x2 over 88m)  kubelet  Container image "k8s.gcr.io/etcd:3.4.13-0" already present on machine
  Normal  Created         72m (x2 over 88m)  kubelet  Created container etcd
  Normal  Started         72m (x2 over 88m)  kubelet  Started container etcd
```
```console
  ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/snapshot-pre-boot.db
```
```console
  root@kmaster:/home/kmaster# ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
> >   --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
> >   snapshot save /opt/snapshot-pre-boot.db
bash: --cacert=/etc/kubernetes/pki/etcd/ca.crt: No such file or directory
root@kmaster:/home/kmaster# ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
>   --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
>   snapshot save /opt/snapshot-pre-boot.db
{"level":"info","ts":1644135212.9267645,"caller":"snapshot/v3_snapshot.go:68","msg":"created temporary db file","path":"/opt/snapshot-pre-boot.db.part"}
{"level":"info","ts":1644135212.9370403,"logger":"client","caller":"v3/maintenance.go:211","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":1644135212.9370914,"caller":"snapshot/v3_snapshot.go:76","msg":"fetching snapshot","endpoint":"https://127.0.0.1:2379"}
{"level":"info","ts":1644135213.0017488,"logger":"client","caller":"v3/maintenance.go:219","msg":"completed snapshot read; closing"}
{"level":"info","ts":1644135213.0038648,"caller":"snapshot/v3_snapshot.go:91","msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","size":"2.8 MB","took":"now"}
{"level":"info","ts":1644135213.0039446,"caller":"snapshot/v3_snapshot.go:100","msg":"saved","path":"/opt/snapshot-pre-boot.db"}
Snapshot saved at /opt/snapshot-pre-boot.db
```
Now your backup db file is located at /opt and you can use this file to restore the cluster assuming that you have deleted all the resources manually
  
 To restore the cluster use below command from k8s documentation
  ```console
  ETCDCTL_API=3 etcdctl --data-dir <data-dir-location> snapshot restore snapshotdb
  ```
  Here default data-dir-location will be the Path of etcd-data (/var/lib/etcd) and in below command we will create a new path(/var/lib/etcd-bkp) or directory to restore the db
  ```console
  ETCDCTL_API=3 etcdctl --data-dir=/var/lib/etcd-bkp snapshot restore /opt/snapshot-pre-boot.db
  ```
  ```console
  root@kmaster:/home/kmaster# ETCDCTL_API=3 etcdctl --data-dir=/var/lib/etcd-bkp snapshot restore /opt/snapshot-pre-boot.db
Deprecated: Use `etcdutl snapshot restore` instead.

2022-02-06T17:18:47+05:30	info	snapshot/v3_snapshot.go:251	restoring snapshot	{"path": "/opt/snapshot-pre-boot.db", "wal-dir": "/var/lib/etcd-bkp/member/wal", "data-dir": "/var/lib/etcd-bkp", "snap-dir": "/var/lib/etcd-bkp/member/snap", "stack": "go.etcd.io/etcd/etcdutl/v3/snapshot.(*v3Manager).Restore\n\t/tmp/etcd-release-3.5.1/etcd/release/etcd/etcdutl/snapshot/v3_snapshot.go:257\ngo.etcd.io/etcd/etcdutl/v3/etcdutl.SnapshotRestoreCommandFunc\n\t/tmp/etcd-release-3.5.1/etcd/release/etcd/etcdutl/etcdutl/snapshot_command.go:147\ngo.etcd.io/etcd/etcdctl/v3/ctlv3/command.snapshotRestoreCommandFunc\n\t/tmp/etcd-release-3.5.1/etcd/release/etcd/etcdctl/ctlv3/command/snapshot_command.go:128\ngithub.com/spf13/cobra.(*Command).execute\n\t/home/remote/sbatsche/.gvm/pkgsets/go1.16.3/global/pkg/mod/github.com/spf13/cobra@v1.1.3/command.go:856\ngithub.com/spf13/cobra.(*Command).ExecuteC\n\t/home/remote/sbatsche/.gvm/pkgsets/go1.16.3/global/pkg/mod/github.com/spf13/cobra@v1.1.3/command.go:960\ngithub.com/spf13/cobra.(*Command).Execute\n\t/home/remote/sbatsche/.gvm/pkgsets/go1.16.3/global/pkg/mod/github.com/spf13/cobra@v1.1.3/command.go:897\ngo.etcd.io/etcd/etcdctl/v3/ctlv3.Start\n\t/tmp/etcd-release-3.5.1/etcd/release/etcd/etcdctl/ctlv3/ctl.go:107\ngo.etcd.io/etcd/etcdctl/v3/ctlv3.MustStart\n\t/tmp/etcd-release-3.5.1/etcd/release/etcd/etcdctl/ctlv3/ctl.go:111\nmain.main\n\t/tmp/etcd-release-3.5.1/etcd/release/etcd/etcdctl/main.go:59\nruntime.main\n\t/home/remote/sbatsche/.gvm/gos/go1.16.3/src/runtime/proc.go:225"}
2022-02-06T17:18:47+05:30	info	membership/store.go:141	Trimming membership information from the backend...
2022-02-06T17:18:47+05:30	info	membership/cluster.go:421	added member	{"cluster-id": "cdf818194e3a8c32", "local-member-id": "0", "added-peer-id": "8e9e05c52164694d", "added-peer-peer-urls": ["http://localhost:2380"]}
2022-02-06T17:18:47+05:30	info	snapshot/v3_snapshot.go:272	restored snapshot	{"path": "/opt/snapshot-pre-boot.db", "wal-dir": "/var/lib/etcd-bkp/member/wal", "data-dir": "/var/lib/etcd-bkp", "snap-dir": "/var/lib/etcd-bkp/member/snap"}
```
 Now you can check all the resources have been restored in your cluster
  
  Thank you....
