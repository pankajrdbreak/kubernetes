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

You can check the pods are running
```console
pankaj@pankajvare:~$ curl -sI knode:31322
HTTP/1.1 200 OK
Server: nginx/1.9.1
Date: Sat, 03 Jul 2021 16:29:26 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 26 May 2015 15:02:09 GMT
Connection: keep-alive
ETag: "55648af1-264"
Accept-Ranges: bytes

pankaj@pankajvare:~$ curl -sI knode:31322 | grep Server
Server: nginx/1.9.1
pankaj@pankajvare:~$ while true; do curl -sI knode:31322 | grep Server; sleep 5; done;
Server: nginx/1.9.1
Server: nginx/1.9.1
Server: nginx/1.9.1
Server: nginx/1.9.1
Server: nginx/1.9.1
Server: nginx/1.9.1
Server: nginx/1.9.1
Server: nginx/1.9.1
Server: nginx/1.9.1
Server: nginx/1.9.1
Server: nginx/1.9.1
```

# How to upgrade version on running pods?

1) After upgrading pod on new version it will take time to download new version on pods one by one

```console
pankaj@pankajvare:~$ kubectl set image deployment nginx nginx=nginx:1.9.5 --record
deployment.apps/nginx image updated
pankaj@pankajvare:~$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
nginx-69c44dfb78-4qct4   1/1     Running             0          15m
nginx-69c44dfb78-5hjjr   0/1     Terminating         0          15m
nginx-69c44dfb78-g4s2d   1/1     Running             0          15m
nginx-69c44dfb78-glwx6   1/1     Running             0          15m
nginx-69c44dfb78-rb89t   1/1     Running             0          22m
nginx-7d76d9cffd-pj4d9   0/1     ContainerCreating   0          6s
nginx-7d76d9cffd-sjqhw   0/1     ContainerCreating   0          6s
nginx-7d76d9cffd-wj99c   0/1     ContainerCreating   0          6s
pankaj@pankajvare:~$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
nginx-69c44dfb78-rb89t   1/1     Running             0          24m
nginx-7d76d9cffd-7c6m8   0/1     ContainerCreating   0          38s
nginx-7d76d9cffd-pj4d9   1/1     Running             0          101s
nginx-7d76d9cffd-sjqhw   0/1     ContainerCreating   0          101s
nginx-7d76d9cffd-thxvj   1/1     Running             0          23s
nginx-7d76d9cffd-wj99c   1/1     Running             0          101s
pankaj@pankajvare:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7d76d9cffd-7c6m8   1/1     Running   0          2m4s
nginx-7d76d9cffd-pj4d9   1/1     Running   0          3m7s
nginx-7d76d9cffd-sjqhw   1/1     Running   0          3m7s
nginx-7d76d9cffd-thxvj   1/1     Running   0          109s
nginx-7d76d9cffd-wj99c   1/1     Running   0          3m7s
```
Then you can check the version of pods if some are under upgrdae then it will show older version & if some pods are upgraded then it will show newer version

```console
pankaj@pankajvare:~$ while true; do curl -sI knode:31322 | grep Server; sleep 1; done;
Server: nginx/1.9.5
Server: nginx/1.9.5
Server: nginx/1.9.5
Server: nginx/1.9.5
Server: nginx/1.9.5
Server: nginx/1.9.1
Server: nginx/1.9.5
Server: nginx/1.9.1
Server: nginx/1.9.5
Server: nginx/1.9.1
Server: nginx/1.9.5
Server: nginx/1.9.5
Server: nginx/1.9.5
Server: nginx/1.9.5
Server: nginx/1.9.1
```
You can check rollout status (whether pods are updated or its in progress) 
```console
pankaj@pankajvare:~$ kubectl rollout status deployment nginx
deployment "nginx" successfully rolled out
```
You can also check rollout history
```console
pankaj@pankajvare:~$ kubectl rollout history deployment nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx nginx=nginx:1.9.5 --record=true
```
If you give wrong version number then nothing will change
```console
pankaj@pankajvare:~$ kubectl set image deployment nginx nginx=nginx:100.200.300 --record
deployment.apps/nginx image updated
pankaj@pankajvare:~$ kubectl rollout status deployment nginx
Waiting for deployment "nginx" rollout to finish: 3 out of 5 new replicas have been updated...
```
But if you see the version of pods its not affected bcs 100.200.300 is not a valid version
Note: One pod tries to pull the image of version 100.200.300 but as the image is not available the pod is in ImagePullErr condition and later we can undo the update and all replicas of pod will be ready
```console
pankaj@pankajvare:~$ kubectrollout history deployment nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
2         kubectl set image deployment nginx nginx=nginx:1.9.5 --record=true
3         kubectl set image deployment nginx nginx=nginx:100.200.300 --record=true
pankaj@pankajvare:~$ while true; do curl -sI knode:31322 | grep Server; sleep 1; done;
Server: nginx/1.9.5
Server: nginx/1.9.5
Server: nginx/1.9.5
Server: nginx/1.9.5
Server: nginx/1.9.5
Server: nginx/1.9.5
Server: nginx/1.9.5
```
Kubernetes try to get 100.200.300 on one pod but bcs of image not available it gives error and then
```console
pankaj@pankajvare:~$ kubectl get pods
NAME                     READY   STATUS             RESTARTS   AGE
nginx-7d76d9cffd-7c6m8   1/1     Running            0          13m
nginx-7d76d9cffd-pj4d9   1/1     Running            0          14m
nginx-7d76d9cffd-sjqhw   1/1     Running            0          14m
nginx-7d76d9cffd-wj99c   1/1     Running            0          14m
nginx-86f554448f-2smrt   0/1     ImagePullBackOff   0          3m31s
nginx-86f554448f-47kxw   0/1     ImagePullBackOff   0          3m31s
nginx-86f554448f-jr2c9   0/1     ImagePullBackOff   0          3m31s
```
Now we can undo the update and make all replicas ready(currently only one replica or pod is affected)
```console
pankaj@pankajvare:~$ kubectl rollout undo deployment nginx
deployment.apps/nginx rolled back
pankaj@pankajvare:~$ kubectl rollout status deployment nginx
deployment "nginx" successfully rolled out
pankaj@pankajvare:~$ kubectl rollout history deployment nginx
deployment.apps/nginx 
REVISION  CHANGE-CAUSE
1         <none>
3         kubectl set image deployment nginx nginx=nginx:100.200.300 --record=true
4         kubectl set image deployment nginx nginx=nginx:1.9.5 --record=true

pankaj@pankajvare:~$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-7d76d9cffd-7c6m8   1/1     Running   0          19m
nginx-7d76d9cffd-k55p4   1/1     Running   0          17s
nginx-7d76d9cffd-pj4d9   1/1     Running   0          20m
nginx-7d76d9cffd-sjqhw   1/1     Running   0          20m
nginx-7d76d9cffd-wj99c   1/1     Running   0          20m
```

