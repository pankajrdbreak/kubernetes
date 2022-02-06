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
