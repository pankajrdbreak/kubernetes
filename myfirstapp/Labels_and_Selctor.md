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
