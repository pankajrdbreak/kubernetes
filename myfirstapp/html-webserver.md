# Deploy an Application on Kubernetes Cluster
Here we will see how to run a simple html application using kubernetes.

Prequisites
1. Running kubernetes cluster- (running on vmware)

   VM-1 :- Hostname/IP : kmaster/172.16.230.138
   
   VM-2 :- Hostname/IP : knode/172.16.230.137
   
3. A Docker image with nginx & html application on Docker Hub so that we can pull that image 
 
Note: In this example my application and nginx web server both are available in the image pankutech/html-webserver


In the below screenshot you can see my kubernetes cluster.

![WhatsApp Image 2021-06-07 at 3 11 12 PM](https://user-images.githubusercontent.com/76647860/120997531-e9711b80-c7a4-11eb-8739-b2aba4562fc7.jpeg)

Steps we are going to follow :
1. Create a YAML file to create deployment named "html-webserver"
2. This will create a pod and one container will run in that pod
3. Then to access this html page we have to expose the deployment as a NodePort service
4. Finally we can access the application from the cluster and outside the cluster 


# Create Deployment using YAML file
1.Copy the below code to html-webserver.yaml

2.Run the kubectl create command to create deployment

```console
apiVersion: apps/v1
kind: Deployment
metadata:
  name: html-webserver
  labels:
    app: html-webserver          # arbitrary label on deployment
spec:
  replicas: 1
  selector:
    matchLabels:        # labels the replica selector should match
      app: html-webserver
  template:
    metadata:
      labels:
        app: html-webserver      # label for replica selector to match
        version: 1.7.9  # arbitrary label we can match on elsewhere
    spec:
      containers:
      - name: html-webserver-cont
        image: pankutech/html-webserver
        ports:
        - containerPort: 80
```

```console
pankaj@kmaster:~$ sudo vim html-webserver.yaml
pankaj@kmaster:~$ kubectl create -f html-webserver.yaml
```
3.Check the deployment is created and automatically pod and container will be created as per the configuration mention in the YAML file
```console
pankaj@kmaster:~$ kubectl get deployments
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
html-webserver   1/1     1            1           4h25m
pankaj@kmaster:~$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
html-webserver-5b459c9556-6bxxm   1/1     Running   1          4h25m

```
4.Now the container with index.html is running on the node.If you want to check details about the pod give below command
```console
pankaj@kmaster:~$ kubectl describe pod html-webserver-5b459c9556-6bxxm
Name:         html-webserver-5b459c9556-6bxxm
Namespace:    default
Priority:     0
Node:         knode/172.16.230.137
Start Time:   Mon, 07 Jun 2021 12:21:03 +0530
Labels:       app=html-webserver
              pod-template-hash=5b459c9556
              version=1.7.9
Annotations:  <none>
Status:       Running
IP:           10.200.1.12
IPs:
  IP:           10.200.1.12
Controlled By:  ReplicaSet/html-webserver-5b459c9556
Containers:
  html-webserver-cont:
    Container ID:   docker://76fd17e0f4cdd2168788f878bb8f3bacd7554152bde36c8c89947ea4205bfa11
    Image:          pankutech/html-webserver
    Image ID:       docker-pullable://pankutech/html-webserver@sha256:87e9ccec8bf88e791b28e0eaddacbf3bd9c0405bac930ebc5c23ca0d4ee3ff04
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Mon, 07 Jun 2021 12:28:10 +0530
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Mon, 07 Jun 2021 12:21:14 +0530
      Finished:     Mon, 07 Jun 2021 12:26:54 +0530
```


# Expose the Deployment using NodePort
1.Exposing the deployment will create a service for that application. So give below command to expose 
```console
pankaj@kmaster:~$ kubectl expose deployment html-webserver --port 80 --type NodePort
service/html-webserver exposed
pankaj@kmaster:~$ kubectl get svc
NAME             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
html-webserver   NodePort    10.32.16.12   <none>        80:32367/TCP   4s
kubernetes       ClusterIP   10.32.0.1     <none>        443/TCP        26h
```
2.Now the service is availble on the port 32367

# Access your Application

Note: 

You can access the container within the clusetr using IP address of the pod but if pod get deleted and re-created then new IP is get assign to that pod.


1.To access the application inside the cluster you can go to browser and type

  http://localhost:32367 or
  
  http://knode:32367/ or
  
  http://172.16.230.137:32367/

![WhatsApp Image 2021-06-07 at 6 03 13 PM](https://user-images.githubusercontent.com/76647860/121017926-7626d400-c7bb-11eb-893e-e8f2f1ede1b9.jpeg)

  
2.To access the application outside the cluster you can go to browser and type

  http://knode:32367/ or
  
  http://172.16.230.137:32367/
  
  ![WhatsApp Image 2021-06-07 at 6 11 39 PM](https://user-images.githubusercontent.com/76647860/121018266-dd448880-c7bb-11eb-8a50-3a004d94d7d2.jpeg)
  
  
  # Thanks for reading..
