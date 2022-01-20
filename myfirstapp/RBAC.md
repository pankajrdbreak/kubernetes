# Role Based Access Control

RBAC is used to provide access to particular namespace to the users.Also we can specify what the user can do within that namespace like user can get,list,create,delete cluster resources like pod,service,namespace,deploy.

In this tutorial we will understand how to create Role and provide access to new user to the cluster namespace using that role

![image](https://user-images.githubusercontent.com/76647860/150096829-b189498b-a37c-41e0-9d3a-cfbabf7894ed.png) 

![image](https://user-images.githubusercontent.com/76647860/150098708-d05cbfdf-771a-4c0b-ba75-7b7c641d6e87.png)

![image](https://user-images.githubusercontent.com/76647860/150099561-83e1e910-643e-4b43-8617-da25ded245d1.png)

Lets start 

1.Create an namespace "infra"
```console
pankaj@pankajvare:~$ kubectl create ns infra
namespace/infra created
pankaj@pankajvare:~$ kubectl get ns
NAME              STATUS   AGE
default           Active   170d
dev               Active   166d
infra             Active   2s
kube-node-lease   Active   170d
kube-public       Active   170d
kube-system       Active   170d
```
2.Generate private key for the user "pankaj"
```console
pankaj@pankajvare:~$ openssl genrsa -out pankaj.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
..............................................................+++++
...............................................................................+++++
e is 65537 (0x010001)
```
3.Generate certificate signing request (csr) "pankaj.csr"
```console
pankaj@pankajvare:~$ openssl req -new -key pankaj.key -out pankaj.csr -subj "/CN=pankaj/O=infra"
```
4.Copy the k8s cluster certificate and key to your users machine
```console
pankaj@pankajvare:~$ sudo scp kmaster@kmaster:/etc/kubernetes/pki/ca.{crt,key} .
```
5.Sign the certificate using certification authority
```console
pankaj@pankajvare:~$ openssl x509 -req -in pankaj.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out pankaj.crt -days 365
Signature ok
subject=CN = pankaj, O = infra
Getting CA Private Key
```
6.Create kubeconfig file for the user
```console
pankaj@pankajvare:~$ kubectl --kubeconfig pankaj.kubeconfig config set-cluster kubernetes --server https://192.168.246.128:6443 --certificate-authority=ca.crt
Cluster "kubernetes" set.
```

```console
pankaj@pankajvare:~$ cat pankaj.kubeconfig 
apiVersion: v1
clusters:
- cluster:
    certificate-authority: ca.crt
    server: https://192.168.246.128:6443
  name: kubernetes
contexts: null
current-context: ""
kind: Config
preferences: {}
users: null
```
