# Kubernetes Labels & Selectors

Labels are used to label the kubernetes objects pods,nodes etc
Selectors are used to select the pods,nodes on basis of labels

Lets create a POD with label using below YAML
```console
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels:
    environment: production
    app: nginx
    server: webserver
spec:
  containers:
  - name: nginxserver
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

```console
kmaster@kmaster:~$ kubectl get pods --show-labels
NAME                READY   STATUS    RESTARTS   AGE     LABELS
label-demo          1/1     Running   0          2m14s   app=nginx,environment=production
nginx-app           1/1     Running   10         40d     run=nginx-app
pod-multi           2/2     Running   38         40d     run=pod-multi
staticpod-kmaster   1/1     Running   10         40d     run=staticpod
```
```console
kmaster@kmaster:~$ kubectl get pods -l environment=production --show-labels
NAME         READY   STATUS    RESTARTS   AGE     LABELS
label-demo   1/1     Running   0          3m41s   app=nginx,environment=production,server=webserver
```
Here app=nginx,environment=production,server=webserver are the labels given to the pod and you can serach the pod using any of the label
