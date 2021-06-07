# Deploy an Application on Kubernetes Cluster
Here we will see how to run a simple html application using kubernetes.

Prequisites
1. Running kubernetes cluster
2. A Docker image with nginx & html application on Docker Hub so that we can pull that image 
 
Note: In this example my application and nginx web server both are available in the image pankutech/html-webserver

In the below screenshot you can see my kubernetes cluster.

![WhatsApp Image 2021-06-07 at 3 11 12 PM](https://user-images.githubusercontent.com/76647860/120997531-e9711b80-c7a4-11eb-8739-b2aba4562fc7.jpeg)

Steps we are going to follow :
1. Create a YAML file to create deployment named "html-webserver"
2. This will create a pod and one container will run in that pod
3. Then to access this html page we have to expose the deployment as a NodePort service
4. Finally we can access the pod from the cluster and outside the cluster 


1) Create YAML file for Deployment


