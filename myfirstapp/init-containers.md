# Init Containers


Init Containers are containers that run before the main container runs with your containerized application. They normally contain setup scripts that prepares an environment for you containerized application. Init Containers also ensure the wider server environment is ready for your application to start to run.

Basically if you want to check whether particular services which will be used by main containers are running fine after that only main container will start.If init container is failed to satisfy the requirements of main container then the main container won't be start.With init containers you can check health of host,availabe cpu,memory,disk etc.Also if you want to verify any network configuration then also init containers are used.

init containers automatically terminated after completing its task.


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
