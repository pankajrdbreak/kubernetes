# Persistent Volume and Claim using NFS 
![WhatsApp Image 2022-01-11 at 12 48 03 PM (1)](https://user-images.githubusercontent.com/76647860/148898152-4e1acda5-9bfb-42bb-a3a3-2a39e0e2ee7e.jpeg)

In this tutorial we will be creating PV and PVC that will be created on StorageClass(NFS)

1. We will create NFS server on host machine 
2. Then will create mount point /srv/nfs/kubedata to hold index.html file that will be used by nginx pod
3. Then create Persistent Volume(nfs-pv) and Claim(nfs-pvc)
4. Then create a nginx pod that will use this PVC to get data from /srv/nfs/kubedata mount point

# Setup NFS Server on host machine
```console
pankaj@pankajvare:~$ sudo apt update

pankaj@pankajvare:~$ sudo apt install nfs-kernel-server

pankaj@pankajvare:~$ sudo systemctl status nfs-server
● nfs-server.service - NFS server and services
     Loaded: loaded (/lib/systemd/system/nfs-server.service; enabled; vendor preset: enabled)
     Active: active (exited) since Tue 2022-01-11 10:22:16 IST; 1min 19s ago
   Main PID: 10434 (code=exited, status=0/SUCCESS)
      Tasks: 0 (limit: 13975)
     Memory: 0B
     CGroup: /system.slice/nfs-server.service

Jan 11 10:22:15 pankajvare systemd[1]: Starting NFS server and services...
Jan 11 10:22:16 pankajvare systemd[1]: Finished NFS server and services.

pankaj@pankajvare:~$ sudo mkdir -p /srv/nfs/kubedata

pankaj@pankajvare:~$ sudo chmod -R 777 /srv/nfs/kubedata

pankaj@pankajvare:~$ sudo vim /etc/exports 

/srv/nfs/kubedata       192.168.246.128(rw,sync,no_subtree_check,insecure)
add the above line in the file and save exit where /srv/nfs/kubedata is mountpoint and 192.168.246.128 is NFS server IP

pankaj@pankajvare:~$ sudo exportfs -rav
exporting *:/srv/nfs/kubedata

pankaj@pankajvare:~$ sudo exportfs -v
/srv/nfs/kubedata
		<world>(rw,wdelay,insecure,root_squash,no_subtree_check,sec=sys,rw,insecure,root_squash,no_all_squash)
    
pankaj@pankajvare:~$ showmount -e
Export list for pankajvare:
/srv/nfs/kubedata *

```
Now NFS server and mount point is ready now we will mount this point on cluster node

```console
pankaj@pankajvare:~$ kubectl get nodes
NAME      STATUS     ROLES                  AGE    VERSION
kmaster   Ready      control-plane,master   161d   v1.21.3
knode     NotReady   <none>                 161d   v1.21.3

pankaj@pankajvare:~$ ssh kmaster@kmaster 

kmaster@kmaster:~$ showmount -e 192.168.234.18

Command 'showmount' not found, but can be installed with:

sudo apt install nfs-common

kmaster@kmaster:~$ sudo apt install nfs-common

kmaster@kmaster:~$ showmount -e 192.168.234.18
Export list for 192.168.234.18:
/srv/nfs/kubedata *

kmaster@kmaster:~$ sudo su

root@kmaster:/home/kmaster# mount -t nfs 192.168.234.18:/srv/nfs/kubedata /mnt

root@kmaster:/home/kmaster# mount | grep kubedata
192.168.234.18:/srv/nfs/kubedata on /mnt type nfs4 (rw,relatime,vers=4.2,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.246.128,local_lock=none,addr=192.168.234.18)

```
Now NFS server & node are ready and you can create PV,PVC and Nginx pod to use it

1. YAML to create PV
```console
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv1
  labels:
    type: local
spec:
  storageClassName: sc1
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: 192.168.234.18
    path: "/srv/nfs/kubedata"
```
```console
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl create -f pv-nfs.yaml
persistentvolume/nfs-pv1 created

pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
nfs-pv1   1Gi        RWX            Retain           Available           sc1                     5s
```
2. YAML to create PVC
```console
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc1
spec:
  storageClassName: sc1
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 500Mi
```
```console
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl create -f pvc-nfs.yaml
persistentvolumeclaim/nfs-pvc1 created

pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl get pvc
NAME       STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
nfs-pvc1   Bound    nfs-pv1   1Gi        RWX            sc1            7s

pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM              STORAGECLASS   REASON   AGE
nfs-pv1   1Gi        RWX            Retain           Bound    default/nfs-pvc1   sc1                     113s
```
3. YAML to create nginx pod
```console
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx-app
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
    spec:
      volumes:
      - name: webserver
        persistentVolumeClaim:
          claimName: nfs-pvc1
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: webserver
          mountPath: /usr/share/nginx/html
```
```console
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl create -f 4-nfs-nginx.yaml
deployment.apps/nginx-app created

pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl get all
NAME                             READY   STATUS              RESTARTS   AGE
pod/nginx-app-557c99779f-vq27l   0/1     ContainerCreating   0          6s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.32.0.1    <none>        443/TCP   161d

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-app   0/1     1            0           6s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-app-557c99779f   1         1         0       6s
```
Now you can create html file in /srv/nfs/kubedata and write some html code and then access the pod shell and check whether in /usr/share/nginx/html folder the file is present.
```console
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ vim /srv/nfs/kubedata/index.html

I added below code in the index.html
<marquee width="80%" direction="left" height="100px" behavior="scroll" direction="left" scrollamount="20"><center><h1>Welcome to Kubernetes...!</h1></center></marquee>

pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl exec -it nginx-app-557c99779f-vq27l /bin/bash

and check the same file /usr/share/nginx/html/index.html is available here
```

Now you can deploy the application as service and access it
```console
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl expose deploy nginx-app --port 80 --type NodePort
service/nginx-app exposed

pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl get all
NAME                             READY   STATUS    RESTARTS   AGE
pod/nginx-app-557c99779f-vq27l   1/1     Running   0          4m8s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.32.0.1      <none>        443/TCP        161d
service/nginx-app    NodePort    10.32.71.201   <none>        80:32136/TCP   5s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-app   1/1     1            1           4m8s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-app-557c99779f   1         1         1       4m8s
```
Now you can access the application using http://kmaster:32136/ 

![WhatsApp Image 2022-01-11 at 2 25 24 PM](https://user-images.githubusercontent.com/76647860/148911406-5a618983-e687-48d2-9188-0d6433a413e3.jpeg)



# Thanks for reading...!
