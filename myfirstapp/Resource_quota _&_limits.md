# Kubernetes Resource Quota and Limits

Resource quota and limits are applied to namespaces in the kubernetes cluster
It is used to limit the cluster resources to the particular user's of that namespaces

ResouceQuota : It will limit the kubernetes objects in the namespace
If we want to limit the number of pods,cronjobs,services,deployment,replicaset,configmap etc to be create in that namespace.

In the following example we will see how to limit the object or resources to namespace "pankaj" 
1. Create namespace "pankaj"
2. Create ResourceQuota named "quota-demo1" in it
3. Create 1 configmap in the namespace
4. Now try to create another configmap in same ns and you will not be able to create it
5. Note- by default one configmap is created in the cluster for namspace i.e kube-root-ca.crt

Commands :
```console
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl create ns pankaj
namespace/pankaj created
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl get ns
NAME              STATUS   AGE
default           Active   160d
dev               Active   156d
kube-node-lease   Active   160d
kube-public       Active   160d
kube-system       Active   160d
pankaj            Active   39s
prod              Active   156d
stag              Active   156d
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl create -f 7-quota-count.yaml
resourcequota/quota-demo1 created
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj get resourcequota
NAME          AGE   REQUEST                      LIMIT
quota-demo1   19s   configmaps: 1/1, pods: 0/2   
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj get cm
NAME               DATA   AGE
kube-root-ca.crt   1      3m1s
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj describe quota quota-demo1
Name:       quota-demo1
Namespace:  pankaj
Resource    Used  Hard
--------    ----  ----
configmaps  1     1
pods        0     2
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl create -f quota-count.yaml
resourcequota/quota-demo1 created
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj describe quota quota-demo1
Name:       quota-demo1
Namespace:  pankaj
Resource    Used  Hard
--------    ----  ----
configmaps  1     2
pods        0     2
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj create configmap cm1 --from-literal=name=nikhil
configmap/cm1 created
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj describe quota quota-demo1
Name:       quota-demo1
Namespace:  pankaj
Resource    Used  Hard
--------    ----  ----
configmaps  2     2
pods        0     2

``` 


Below is ResourceQuota yaml file : quota-count.yaml
```console
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-demo1
  namespace: pankaj
spec:
  hard:
    pods: "2"
    configmaps: "2"
```
