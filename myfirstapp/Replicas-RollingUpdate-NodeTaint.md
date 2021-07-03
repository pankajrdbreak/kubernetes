# How to Scale-Up Replicas of running Deployment in Kubernetes?

1) Create a Nginx deployment with single replica
2) Create NodePort Service for the same deployment
3) Scale-up deployment to replicas you want

```console
pankaj@pankajvare:~$ kubectl create deployment nginx --image=nginx:1.9.1 --port=80
deployment.apps/nginx created
pankaj@pankajvare:~$ kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           8s
pankaj@pankajvare:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-69c44dfb78-rb89t   1/1     Running   0          12s
pankaj@pankajvare:~$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE     IP             NODE    NOMINATED NODE   READINESS GATES
nginx-69c44dfb78-rb89t   1/1     Running   0          2m11s   10.200.1.130   knode   <none>           <none>
```

```console
pankaj@pankajvare:~$ kubectl expose deployment nginx --type=NodePort
service/nginx exposed
pankaj@pankajvare:~$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.32.0.1       <none>        443/TCP        27d
nginx        NodePort    10.32.215.228   <none>        80:31322/TCP   2s
```

```console
pankaj@pankajvare:~$ kubectl scale deployment nginx --replicas=5
deployment.apps/nginx scaled
pankaj@pankajvare:~$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
nginx-69c44dfb78-4qct4   0/1     ContainerCreating   0          2s
nginx-69c44dfb78-5hjjr   0/1     ContainerCreating   0          2s
nginx-69c44dfb78-g4s2d   0/1     ContainerCreating   0          2s
nginx-69c44dfb78-glwx6   1/1     Running             0          2s
nginx-69c44dfb78-rb89t   1/1     Running             0          7m23s
pankaj@pankajvare:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-69c44dfb78-4qct4   1/1     Running   0          24s
nginx-69c44dfb78-5hjjr   1/1     Running   0          24s
nginx-69c44dfb78-g4s2d   1/1     Running   0          24s
nginx-69c44dfb78-glwx6   1/1     Running   0          24s
nginx-69c44dfb78-rb89t   1/1     Running   0          7m45s
```
