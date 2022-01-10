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

pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj get cm
NAME               DATA   AGE
cm1                1      7m55s
kube-root-ca.crt   1      13m
``` 
Now resouce limit for creating configmap is reached and if you try to create new cm then it will give you following error
```console
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj create configmap cm2 --from-literal=name=nikhil
error: failed to create configmap: configmaps "cm2" is forbidden: exceeded quota: quota-demo1, requested: configmaps=1, used: configmaps=2, limited: configmaps=2
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

Now try this for Deployment-Replica-Pods
The current limit for pod is 2 means you can create only 2 pods in "pankaj" ns
```console
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj create deploy nginx --image=nginx:alpine --replicas=1
deployment.apps/nginx created

pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj get all 
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-565785f75c-6j6hn   1/1     Running   0          18s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           18s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-565785f75c   1         1         1       18s

pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj scale deploy nginx --replicas=2
deployment.apps/nginx scaled
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj get all 
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-565785f75c-452cz   1/1     Running   0          2s
pod/nginx-565785f75c-6j6hn   1/1     Running   0          50s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   2/2     2            2           50s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-565785f75c   2         2         2       50s

pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj scale deploy nginx --replicas=3
deployment.apps/nginx scaled
pankaj@pankajvare:~/Desktop/kubernetes/yamls$ kubectl -n pankaj get all 
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-565785f75c-452cz   1/1     Running   0          64s
pod/nginx-565785f75c-6j6hn   1/1     Running   0          112s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   2/3     2            2           112s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-565785f75c   3         2         2       112s
```

In last output when we scale the replicas to 3 it shows scaled but in actual only 2 pods are running



Limits :
Limits are applied to the memory and cpu usage in the namespace
