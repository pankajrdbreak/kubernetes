# Init Containers


Init Containers are containers that run before the main container runs with your containerized application. They normally contain setup scripts that prepares an environment for you containerized application. Init Containers also ensure the wider server environment is ready for your application to start to run.

Basically if you want to check whether particular services which will be used by main containers are running fine after that only main container will start.If init container is failed to satisfy the requirements of main container then the main container won't be start.With init containers you can check health of host,availabe cpu,memory,disk etc.Also if you want to verify any network configuration then also init containers are used.

init containers automatically terminated after completing its task.

Here init container works like puller which pulls static content once save it to shared volume and exit.


# Example 
```console
apiVersion: v1
kind: Pod
metadata:
   name: init-demo
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: htmldir
      mountPath: /usr/share/nginx/html
  initContainers: 
  - name: install
    image: busybox
    command: ["wget"]
    args: ["-O","/work-dir/index.html","https://groww.in/"]
    volumeMounts: 
    - name: htmldir
      mountPath: "/work-dir"
  volumes:
  - name: htmldir
    emptyDir: {} 
    
```
Execution of YAML

1) After running above yaml file one init container will be created which will retirieve content from https://groww.in/  web server and copy it to /work-dir/index.html i.e mount directory.
2) After that init container will get terminated as its task is finished
3) Now the main container "nginx" will be created which will mount "htmldir" on /usr/share/nginx/html directory
4) Now you can access that content of https://groww.in/ from nginx container.

Note :- htmldir is the name of /work-dir which will be shared by both the containers as we know container shares networking and storage in pod.

Output
```console
pankaj@kmaster:~/Desktop$ kubectl create -f init-cont-demo.yaml 
pod/init-demo created
pankaj@kmaster:~/Desktop$ kubectl get pods -o wide
NAME                              READY   STATUS     RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
html-webserver-5b459c9556-jgmzt   1/1     Running    5          10d   10.200.1.77   knode   <none>           <none>
html-webserver-5b459c9556-vf64q   1/1     Running    5          10d   10.200.1.74   knode   <none>           <none>
init-demo                         0/1     Init:0/1   0          9s    10.200.1.78   knode   <none>           <none>
multi-container-demo              2/2     Running    2          18h   10.200.1.75   knode   <none>           <none>
multi-container-demo2             2/2     Running    2          18h   10.200.1.76   knode   <none>           <none>
pankaj@kmaster:~/Desktop$ kubectl get pods -o wide
NAME                              READY   STATUS    RESTARTS   AGE   IP            NODE    NOMINATED NODE   READINESS GATES
html-webserver-5b459c9556-jgmzt   1/1     Running   5          10d   10.200.1.77   knode   <none>           <none>
html-webserver-5b459c9556-vf64q   1/1     Running   5          10d   10.200.1.74   knode   <none>           <none>
init-demo                         1/1     Running   0          25s   10.200.1.78   knode   <none>           <none>
multi-container-demo              2/2     Running   2          18h   10.200.1.75   knode   <none>           <none>
multi-container-demo2             2/2     Running   2          18h   10.200.1.76   knode   <none>           <none>


pankaj@kmaster:~/Desktop$ kubectl describe pod init-demo
Name:         init-demo
Namespace:    default
Priority:     0
Node:         knode/172.16.230.137
Start Time:   Mon, 21 Jun 2021 14:32:02 +0530
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.200.1.78
IPs:
  IP:  10.200.1.78
Init Containers:
  install:
    Container ID:  docker://1d83d8707df7b994922f18ca313fafb90826b48667c2d009cd1892328724b579
    Image:         busybox
    Image ID:      docker-pullable://busybox@sha256:930490f97e5b921535c153e0e7110d251134cc4b72bbb8133c6a5065cc68580d
    Port:          <none>
    Host Port:     <none>
    Command:
      wget
    Args:
      -O
      /work-dir/index.html
      https://groww.in/
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Mon, 21 Jun 2021 14:32:08 +0530
      Finished:     Mon, 21 Jun 2021 14:32:10 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-r857w (ro)
      /work-dir from htmldir (rw)
Containers:
  nginx:
    Container ID:   docker://28d7aba3598f8b7a3e7d7271899a5b8d827dfa54bf70969b972c157e64e5cd90
    Image:          nginx:alpine
    Image ID:       docker-pullable://nginx@sha256:0f8595aa040ec107821e0409a1dd3f7a5e989501d5c8d5b5ca1f955f33ac81a0
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 21 Jun 2021 14:32:11 +0530
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from htmldir (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-r857w (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  htmldir:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  kube-api-access-r857w:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  28m   default-scheduler  Successfully assigned default/init-demo to knode
  Normal  Pulling    28m   kubelet            Pulling image "busybox"
  Normal  Pulled     28m   kubelet            Successfully pulled image "busybox" in 4.915519291s
  Normal  Created    28m   kubelet            Created container install
  Normal  Started    28m   kubelet            Started container install
  Normal  Pulled     28m   kubelet            Container image "nginx:alpine" already present on machine
  Normal  Created    28m   kubelet            Created container nginx
  Normal  Started    28m   kubelet            Started container nginx
  
  
pankaj@kmaster:~/Desktop$ kubectl exec -it init-demo sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
Defaulted container "nginx" out of: nginx, install (init)
/ # curl 10.200.1.78

<!doctype html>
<html lang="en" data-theme=light>
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, minimum-scale=1, maximum-scale=1, user-scalable=no">
<link rel="preload" as="script" href="//assets-netstorage.groww.in/website-assets/prod/1.6.8/build/client/js/vendor.ce6f2f2d.js">
<link rel="preload" href=//assets-netstorage.groww.in/website-assets/prod/1.6.8/build/client/fonts/KFOlCnqEu92Fr1MmEU9fBBc4AMP6lQ.709f6f90.woff2 as="font" type="font/woff2" crossorigin>
<link rel="preload" href=//assets-netstorage.groww.in/website-assets/prod/1.6.8/build/client/fonts/KFOmCnqEu92Fr1Mu4mxKKTU1Kg.ece6673e.woff2 as="font" type="font/woff2" crossorigin>
<link rel="stylesheet" href="//assets-netstorage.groww.in/website-assets/prod/1.6.8/build/client/css/main.73daa036.css">
<link rel="stylesheet" href="//assets-netstorage.groww.in/website-assets/prod/1.6.8/build/client/css/HomePage.b689eaf3.css">
<link rel="manifest" href="/manifest.json?v=1">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="application-name" content="Groww">
<meta name="apple-mobile-web-app-title" content="Groww">
<meta name="theme-color" content="#5500eb">
<meta name="msapplication-navbutton-color" content="#5500eb">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="msapplication-starturl" content="/">
<link type="application/opensearchdescription+xml" rel="search" href="/osdd.xml?v=3" />
<link rel="icon" type="image/png" href="/favicon-32x32-groww.ico" sizes="32x32">
<title data-react-helmet="true">Groww - Online Demat, Trading and Direct Mutual Fund Investment in India</title>


```
