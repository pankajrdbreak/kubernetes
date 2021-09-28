# Rancher setup to monitor kubernetes cluster
Setting up rancher is very easy task follow the below steps 

1. Install Rancher
> You can install Rancher on machine from which you want to monitor your kubernetes cluster
> sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher/rancher
> Run the above command and Rancher will be install and run on ports 80 and 443

2. Log in to Rancher 
> In browser type http://localhost/ and you will redirect to Rancer login page.
> Here you have to enter password for default user 'admin'.
> You can follow steps give there to get your password or else
> Give the below command to reset the password.
> sudo docker exec -ti 054cb7348655 reset-password
> 054cb7348655  is Rancher container id.

3. Now you can add your kubernetes cluster to Rancher
> If you have VM based k8s cluster then go to 'Import Cluster' option.
> Give name to cluster.
> Now you will get some commands to run on k8s master node like below.
> Run the kubectl command below on an existing Kubernetes cluster running a supported Kubernetes version to import it into Rancher:

kubectl apply -f https://192.168.36.18/v3/import/z8k8qlg67dggfjzpc26j8lqzphgr64sw95fb6pmr2cbjd4gcrcqp78_c-m-gwcx6855.yaml

If you get a "certificate signed by unknown authority" error, your Rancher installation has a self-signed or untrusted SSL certificate. Run the command below instead to bypass the certificate verification:

curl --insecure -sfL https://192.168.36.18/v3/import/z8k8qlg67dggfjzpc26j8lqzphgr64sw95fb6pmr2cbjd4gcrcqp78_c-m-gwcx6855.yaml | kubectl apply -f -

If you get permission errors creating some of the resources, your user may not have the cluster-admin role. Use this command to apply it:

kubectl create clusterrolebinding cluster-admin-binding --clusterrole cluster-admin --user <your username from your kubeconfig>
  
  4. Once done you will see your cluster is active in Rancher

  ![Screenshot from 2021-09-28 12-02-40](https://user-images.githubusercontent.com/76647860/135061617-ee0653c8-d994-45a7-a43d-11b8adfae668.png)
  ![Screenshot from 2021-09-28 12-02-52](https://user-images.githubusercontent.com/76647860/135061882-b2e36e43-35f3-47d6-88bc-93ed6e2067c6.png)
  ![Screenshot from 2021-09-28 12-03-14](https://user-images.githubusercontent.com/76647860/135061910-59663b4c-dd22-469c-95fc-fb9c587bdf3e.png)
  ![Screenshot from 2021-09-28 12-03-24](https://user-images.githubusercontent.com/76647860/135061924-4cc89c4e-307a-4a06-a3cd-6b4dfb246ff1.png)
  ![Screenshot from 2021-09-28 12-06-17](https://user-images.githubusercontent.com/76647860/135061949-1fb15450-7603-4104-8b1d-813cb6793e27.png)

