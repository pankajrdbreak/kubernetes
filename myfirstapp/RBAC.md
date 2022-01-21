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
7.Add user "pankaj" to this kubeconfig file
```console
pankaj@pankajvare:~$ kubectl --kubeconfig pankaj.kubeconfig config set-credentials pankaj --client-certificate /home/pankaj/pankaj.crt --client-key /home/pankaj/pankaj.key 
User "pankaj" set.
```
8.Now set context for user
Note: you can set any name for context. Here we have named it as "pankaj-kubernetes"
```console
pankaj@pankajvare:~$ kubectl --kubeconfig pankaj.kubeconfig config set-context pankaj-kubernetes --cluster kubernetes --namespace infra --user pankaj
Context "pankaj-kubernetes" created.
```
Below is the kubeconfig file after above commands.
Note: By default current-context is empty "" we put our context name i.e pankaj-kubernetes
```console
pankaj@pankajvare:~$ cat pankaj.kubeconfig
apiVersion: v1
clusters:
- cluster:
    certificate-authority: ca.crt
    server: https://192.168.246.128:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    namespace: infra
    user: pankaj
  name: pankaj-kubernetes
current-context: pankaj-kubernetes
kind: Config
preferences: {}
users:
- name: pankaj
  user:
    client-certificate: pankaj.crt
    client-key: pankaj.key
```
Now kubeconfig file for user pankaj is created but you have to send this kubeconfig file along with pankaj's key and certificate to pankaj. Or you can just copy the master node kubeconfig file and make changes in the file like below
1. Change context user,context name,current-context,users name
2. Paste the base 64 value of user's client-certificate-data and client-key-data. To get base 64 values use following command

```console
pankaj@pankajvare:~$ cat pankaj.key | base64 -w0
pankaj@pankajvare:~$ cat ca.crt | base64 -w0
```

```console
pankaj@pankajvare:~$ cat pankaj.kubeconfig

apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM1ekNDQWMrZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJeE1EZ3dNakEzTXpRMU1Gb1hEVE14TURjek1UQTNNelExTUZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTTBBCnpKd0YreVJnWkRKYi90M1RkUURlam1vY1lVendyaWd5cndlMnliMzhxWU95K0p3N05CZXNVSitMVnZSeGVYeUYKbXpzc1pFZzUvUCtSSDQ2dEluMlhjWEloTHZkZFVFOTZKdXNtUGxMTWQzdERka1g0WmQ3NFhkbFVkc3RCbG1DRQpjeDRtQkNGOG5hRFQ1SXNGZGtKR1I4OXFDZ1pvaElVRUpXS0NnaDcxcEZmMnAyc0hXcDRrbkl6RVA0QWZnMHY2Ck9jSCtObi9OV29uUmVZQjRJWTczamtONVphT3BsbUJZdDAzKzM1VUhwRzdka0MvNFhld3Viek5vYmhESVVEODQKaDl6OWpkNmxhZG50MkRDRVNtaC8vUytSVE5IUGJTTDhzb1ZoOXpSZEpGZmVlcDFOWWdWVDlxRE5xN1pJNVBSagpOOUZaSTdwWlZKMmw2TEdDNlJNQ0F3RUFBYU5DTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0hRWURWUjBPQkJZRUZJMEhkbWVpakV2WjlNald5aXc5Z1FPZUt3azBNQTBHQ1NxR1NJYjMKRFFFQkN3VUFBNElCQVFBdlJqdHNkUXFVSXFReWlCMktkWWRWQ3ZmaUFoVXdzWFBrckY4eEVwTlllM2J0YzJWbQppQ2ZTWWJSUEgwLzhNOHhYeWlaek1QcVdpQTBjVGV1UmFEVzBCTHNmWkVGbE5mYTdWYlQ0dGwvajlYRFJIa0FmCnhBTllBUm9ZVllNelEyTEdadEpOemlrdm9zQ0F0WkM2THZ2UWZHdWY5TmY5U3gvV2l2VU85VGhpZ0JiN0R2RmIKTk1JcTZLakk1MXFUOWxmaEhaVTBoZHJEK2tnVEtOTGF4KzVPSHFNdkZla3pvUWFGQVoweitUbkNEWkFwMGZxNgpFSHZXREZxLzkyR0EvUWRwR1ZlbGVPYVUzZEI4QjlUeGpFMy82ZVJtVnFkUVY0Z3J3cmNPTk1ySE9UNS91aGtoCndXUnc3M1FITU1TdEd0UEZBNVN5MGx3cThkY0ZLWkIxY1NWSQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    server: https://192.168.246.128:6443
  name: kubernetes
contexts:
- context: 
    cluster: kubernetes
    user: pankaj
    namespace: infra
  name: pankaj-kuebrnetes
current-context: pankaj-kuebrnetes
kind: Config
preferences: {}
users:
- name: pankaj
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN2VENDQWFVQ0ZGc245blRUaUxNcTdjRVIycWVQSlRabVF2RHhNQTBHQ1NxR1NJYjNEUUVCQ3dVQU1CVXgKRXpBUkJnTlZCQU1UQ210MVltVnlibVYwWlhNd0hoY05Nakl3TVRFNU1UY3hORE14V2hjTk1qTXdNVEU1TVRjeApORE14V2pBaE1ROHdEUVlEVlFRRERBWndZVzVyWVdveERqQU1CZ05WQkFvTUJXbHVabkpoTUlJQklqQU5CZ2txCmhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBdjRUT0VLYWRHS3hiT2IwcTBCczRyR3JxeklRR3RMYlEKQVB2aWliUmp5N0NFeDhqa3BKWkd5MGFCOXV4WnE1Wkp3ZzQ5Y1JnNDJudVBqaWtiVVU3VDR3UXA1dzg1S25BSQphZndURElGUHZhRjZrblFSb1hyN3RUdXZYMEh5cW53WHptaGFZeHgxZ09uL1FSWlJkMlVqeXgrdUhFajc2aDBwCmlKeFVmM01EK2tDQklLUG1taGVSMmJMRHZHaW9sK1MzV2l4eXg5ZXFGUFUvekxtSGw5N09BUStSTTFpNGY3ZE8KV25idHZCdFYvbWFXaExGU1J4eHFrRERZR21KN1BHVHIyMW9XZ2Y3VTIwRTNyVHJoSTJCanAyc3RUb21nbFVTVwpmdW9WWnlJVDVNUzRXY0s0bjhVTWd3QTRqclU4NXdUbGd4TG00aFdBSVA1cDVHOEFocHMxbHdJREFRQUJNQTBHCkNTcUdTSWIzRFFFQkN3VUFBNElCQVFDcnJtdExVV2lXUUgvOWd2UHB0RWhVTzNXUTVHa1phTW9UWkpLa3pqblcKM2FtRlJmaWFFTnVLOHNObUtOT2MvcXZ3RXpUazArSElPVkxWaWRoNHNaUnRzU25CZ2JoVFhwNlo2UTJwdjl1NgprU2NoekxIK0hRbEU0MWpvdmFQRVlLdWlrcS8zZXhkZmVTaktqTUhEOG9Rd2ZteGpJZGJYZXhnOUY5eE9mZVZ2CjJROTFBWkRTalhBYWdMcUY0c0NHWkFwYmpDTGt0a2IrVzk1b0xJSjFRSzJWNXRkMkgzMUF2d2t2K3UwN1hINmEKM21qMmFycjgycUNVQ2dZZ0ZaRWxRQjhyOHp0RGFyb3d2SytQc0dCME9rbU5STHdkQkdvT0hZUlNoSlJjSUpwSgpac1dJVlc1ZlJ1TllTWVpFSFpvVlU0cnhyaW5hZkpnTXpST2VrMUtGSnRBbgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBdjRUT0VLYWRHS3hiT2IwcTBCczRyR3JxeklRR3RMYlFBUHZpaWJSank3Q0V4OGprCnBKWkd5MGFCOXV4WnE1Wkp3ZzQ5Y1JnNDJudVBqaWtiVVU3VDR3UXA1dzg1S25BSWFmd1RESUZQdmFGNmtuUVIKb1hyN3RUdXZYMEh5cW53WHptaGFZeHgxZ09uL1FSWlJkMlVqeXgrdUhFajc2aDBwaUp4VWYzTUQra0NCSUtQbQptaGVSMmJMRHZHaW9sK1MzV2l4eXg5ZXFGUFUvekxtSGw5N09BUStSTTFpNGY3ZE9XbmJ0dkJ0Vi9tYVdoTEZTClJ4eHFrRERZR21KN1BHVHIyMW9XZ2Y3VTIwRTNyVHJoSTJCanAyc3RUb21nbFVTV2Z1b1ZaeUlUNU1TNFdjSzQKbjhVTWd3QTRqclU4NXdUbGd4TG00aFdBSVA1cDVHOEFocHMxbHdJREFRQUJBb0lCQUNxMzJxYklJeDVQdzRGYgplbU0wenkxV0k4MCtYYWlOWmxQeDJ4UFFFcnBxUXhPMDhacnNraG5oUGpmdmZBalQydHZZQlVobW1MZlMrN0EvCjV3SDV6VFZEeG56dFhsamk2RjJMaGt3eHc3R09oU0tKbFMwcG0xOVBVc3l5andnTDZkdDJWMExvSkNWL0RCcGUKZWdsaG05eEEwcnNvWkZoUjdPTzF5dnNxa21hTS9UQTNQc1lLMDMrbWRPZDdrWUUyc2dJMHUrak83WnpFc3NaaQo1UkhwQ3IyOTMrQVpyRFV6VFVGdGlHdk41VWV6cnJWR242RStKSXdoSEdhVUR4aTZTWURFbHA5S013aGc1NnN6CjhhZGZhS0RoUGtla2xLUG1qc3UxcC9CSHYwVWRSM0xZVVJ3TlluY0dxUnhHWEthMmw2aXNBQkZiNDl1MldtWU4KOUY1eUNlRUNnWUVBK3lTZVMyek41YVBOVEUwKzZiZzFhTkNnVlRBbGxMNFFZNFhKUDA2WXNDK0dML0U5WFRNYwowMHBhVFMrR2U1bmRLbmVRaDVJNXNWZG1HeXB2QURQMVZaSzVKbUQyTWs1VVNMMFUrTWZaZTFrWFZUb3RZZjlnCk5ldVF4UVdKMno5NlV3clNlWFg0TmZXcE5YQlNVSEpqU2g2THkxSzJOWkVaeEl6REdlR0ZzcWtDZ1lFQXd6aisKVGJhaENLdEVWeVJYT2Vkc0FITTVJM3dkdFhJSWZUd0VPS2FkU0hjeUVZcURHS0Y2ZGFnQXR2RWlNajVGdGdJSgphZ0VJM3FpOUk1RWhLTWVkWldlbzFGSXg0NTROMGtBZUhmVk45TTRSSzZ0ZjJOMFQzQUFHUm05bWJGUHp4SzRiCk9UelBhOEN6eXJZcEM2M3BNVFYxcitkSldCYThwU0FMakRQWERqOENnWUJadDNjbEVyVnJOOXo3U1EwVWlVM3IKSjd4Zk1sZjZqdnRqMGtOV2JrbDFoMFMwazhXTUtkbytVTzE1YldUcGVzbmJoZU1IeTJENHpYUVllRXczRWxpdQpQVUFQU2N3cHBIblBrbHlQa3pWS0wwVjZtTkhsbEVsV2VkUzV6WVMxNGpOY3Z4ejVidjlBcDRYUEpWVUNrQnFRCk8rRk12VHVDWDFlSk00L3ZDdldzSVFLQmdFMVVmaWQ2ZUQ5ZTJDdE1rZUMxOHVvYXVqOThJcWlGQ3lmUVpqdXEKaEJMNFpEVGVrUjlvbDRHVGt2VGtmNDgzYTVXMUtOVjhvMjdQbUZ4R1dNUTJqZnBsSFZNOVc5VzEvZk9Td2x0TAptQjJvb3RTUmhkMzVkS3hvdGhPZ2ZRbmNGMnVKSys5NFR4RjN1OEJJZCtuUWNkYTBQbkgzUSs1STAyRDFXSjJvCnl6OHhBb0dBTnNqSk9nYWY1b0JYMUZXYUQ5WDRtOTdpUUVxMjNqdWFRT2dkRXZCUk5WbU5OanhsamxvVndjeW8KRmlHVVh5Y0FEN1VwRXZBVno5bTBXL2RHQWF1bFF2bHVwdFlYM0tneGVNSkF6SWlERDRCb3VIWVZmam83V2w1UQpjSThQdUcwaE9oOGt1bnlPMWFYajVYbUhjall5aWJSNDJqK1NJUExWbTAra0hVdEhXRHM9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
```
```console
pankaj@pankajvare:~$ kubectl --kubeconfig pankaj.kubeconfig get pods
Error from server (Forbidden): pods is forbidden: User "pankaj" cannot list resource "pods" in API group "" in the namespace "default"
```
The error is saying you are able to connect the cluster but you dont have permissions/access to resources yet.So now we will give access to user "pankaj"
To provide access we have to create Role. Below we have given get and list access to the pods to the user.
```console
pankaj@pankajvare:~$ kubectl create role pankaj-infra --verb=get,list --resource=pods --namespace=infra
role.rbac.authorization.k8s.io/pankaj-infra created

pankaj@pankajvare:~$ kubectl get roles -n infra
NAME           CREATED AT
pankaj-infra   2022-01-21T05:36:01Z
pankaj@pankajvare:~$ kubectl get roles -n infra pankaj-infra -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2022-01-21T05:36:01Z"
  name: pankaj-infra
  namespace: infra
  resourceVersion: "277335"
  uid: 2cf4a921-7191-4095-ba9f-f7b2ac64226a
rules:
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  ```
Now you can attach this role to user using rolebinding
```console
pankaj@pankajvare:~$ kubectl create rolebinding pankaj-infra-rolebinding --role=pankaj-infra --user=pankaj --namespace=infra
rolebinding.rbac.authorization.k8s.io/pankaj-infra-rolebinding created
pankaj@pankajvare:~$ kubectl get rolebinding -n infra
NAME                       ROLE                AGE
pankaj-infra-rolebinding   Role/pankaj-infra   24s
pankaj@pankajvare:~$ kubectl get rolebinding -n infra pankaj-infra-rolebinding -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: "2022-01-21T05:39:09Z"
  name: pankaj-infra-rolebinding
  namespace: infra
  resourceVersion: "277573"
  uid: 68a139ba-0ff0-4c7e-b61c-29b2c3731abf
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pankaj-infra
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: pankaj
```
Now if you try to access the pods in inra ns you will get output asn not found because there is no pod in that namespace
```console
pankaj@pankajvare:~$ kubectl --kubeconfig pankaj.kubeconfig get pods
No resources found in infra namespace.
```
And if you try to access pods from any other ns then you will get error or if you try to access resource other than pods then also you will get error because for now you only have access to pods and to get and list the pods
```console
pankaj@pankajvare:~$ kubectl --kubeconfig pankaj.kubeconfig get pods -n kube-system
Error from server (Forbidden): pods is forbidden: User "pankaj" cannot list resource "pods" in API group "" in the namespace "kube-system"
```
Now if you want to give all permission to user then just edit role and under resource and verbs section put '*'
```console
pankaj@pankajvare:~$ kubectl -n infra edit role pankaj-infra
role.rbac.authorization.k8s.io/pankaj-infra edited


apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2022-01-21T05:36:01Z"
  name: pankaj-infra
  namespace: infra
  resourceVersion: "279639"
  uid: 2cf4a921-7191-4095-ba9f-f7b2ac64226a
rules:
- apiGroups:
  - '*'
  resources:
  - '*'
  verbs:
  - '*'
  
pankaj@pankajvare:~$ kubectl --kubeconfig pankaj.kubeconfig get deploy
No resources found in infra namespace.

pankaj@pankajvare:~$ kubectl --kubeconfig pankaj.kubeconfig get svc
No resources found in infra namespace.
```
Now you will be able to create delete deploy or service or pod
```console
pankaj@pankajvare:~$ kubectl --kubeconfig pankaj.kubeconfig create deploy nginx --image=nginx:alpine 
deployment.apps/nginx created
pankaj@pankajvare:~$ kubectl --kubeconfig pankaj.kubeconfig get all
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-565785f75c-9fpr5   1/1     Running   0          6s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           7s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-565785f75c   1         1         1       7s
```
#Now user pankaj has got the access
But what if you want to add more users and provide same namespace role and rolebinding?
Typically you can edit rolebinding and add users in subject section of file one by one but it is not the way for that the concept of group is came into picture
```console
pankaj@pankajvare:~$ kubectl -n infra get rolebinding pankaj-infra-rolebinding -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: "2022-01-21T05:39:09Z"
  name: pankaj-infra-rolebinding
  namespace: infra
  resourceVersion: "277573"
  uid: 68a139ba-0ff0-4c7e-b61c-29b2c3731abf
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pankaj-infra
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: pankaj
```
#Now i want to add user "ankit" 
For ankit user we have to follow the same steps like create certificate,key,csr after creating this follow the below stes

1.Set cluster
```console

```
2.Set credentials
