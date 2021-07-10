# ConfigMap

It is not safe to create container image with all server configuration files as we may upload docker images on remote. That's why we use configmaps to pass the config files to the container after creating it.

What we need to start
As we are going to deploy a nginx container which will use config file from configmap

1) We need external config file --> nginx-connector.conf
2) Will create configmap named "nginx-config" using this config file
3) At last we will create container which will use this nginx-config configmap 

nginx-connector.conf
```console
pankaj@kmaster:~/Desktop$ cat nginx-connector.conf 
server {
    listen       8080;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

Creating ConfigMap
```console
pankaj@kmaster:~/Desktop$ kubectl create configmap nginx-config --from-file=nginx-connector.conf 
configmap/nginx-config created
pankaj@kmaster:~/Desktop$ kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      34d
nginx-config       1      4s
pankaj@kmaster:~/Desktop$ kubectl describe cm nginx-config
Name:         nginx-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
nginx-connector.conf:
----
server {
    listen       8080;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

Events:  <none>
```
Now we will mount this configmap on containers file system.
Lets create container with the following YAML
```console
pankaj@kmaster:~/Desktop$ cat configmapmdemo.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
      - name: config-volume
        configMap:
          name: nginx-config
      containers:
      - name: nginx
        image: nginx:alpine
        volumeMounts:
        - mountPath: /etc/nginx/conf.d
          name: config-volume
```
Now you can check deployments and containers are running
```console
pankaj@kmaster:~/Desktop$ kubectl create -f configmapmdemo.yaml
deployment.apps/nginx created
pankaj@kmaster:~/Desktop$ kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   0/1     1            0           8s
pankaj@kmaster:~/Desktop$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-656df74c4c-vmh9w   0/1     Pending   0          12s
pankaj@kmaster:~/Desktop$ kubectl get pods -o wide
NAME                     READY   STATUS              RESTARTS   AGE   IP       NODE    NOMINATED NODE   READINESS GATES
nginx-656df74c4c-vmh9w   0/1     ContainerCreating   0          20s   <none>   knode   <none>           <none>
pankaj@kmaster:~/Desktop$ kubectl get pods -o wide
NAME                     READY   STATUS    RESTARTS   AGE    IP             NODE    NOMINATED NODE   READINESS GATES
nginx-656df74c4c-vmh9w   1/1     Running   0          105s   10.200.1.142   knode   <none>           <none>
```
Now you can see the page is serving on port 8080 as we mentioned in configmap and not on default port

