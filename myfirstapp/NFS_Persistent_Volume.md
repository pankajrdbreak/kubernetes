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

