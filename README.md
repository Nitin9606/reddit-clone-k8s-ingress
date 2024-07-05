Deployed a Reddit Copy clone on Kubernetes with Ingress Enabled


launch two ec2 instance(t2-micro , t2-medium) 

Before we begin with the Project,following prerequisites installed:

- EC2 ( AMI- Ubuntu, Type- t2.medium )
- Docker
- Git
- Minikube
- kubectl

You can Install all this by doing the below steps one by one. and these steps are for Ubuntu AMI.


# Steps:-

# For Docker Installation
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker  ##(change permission for docker to add a user in group)

# For Git Installation
sudo apt-get install -y git

# For Minikube & Kubectl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube 

sudo snap install kubectl --classic
minikube start --driver=docker






Table of contents
Step 1: Clone the source code
Step 2: Containerize the Application using Docker
Step 3) Building Docker-Image
Step 4) Push the Image To DockerHub
Step 5) Write a Kubernetes Manifest File
Write Deployment.yml file
Write Service.yml file
Step 5) Deploy the app to Kubernetes & Create a Service For It
Step 6) Let's Configure Ingress
Ingress:
Step 8) Expose the app
Test Ingress



Step 1: Clone the source code
- The first step is to clone the source code for the app. You can do this by using this command 
    git clone https://github.com/Nitin9606/reddit-clone-k8s-ingress.git


Step 2: Containerize the Application using Docker

- Write a Dockerfile with the following code:

FROM node:19-alpine3.15

WORKDIR /reddit-clone

COPY . /reddit-clone

RUN npm install 

EXPOSE 3000

CMD ["npm","run","dev"]

Step 3) Building Docker-Image


- Now it's time to build Docker Image from this Dockerfile.


 docker build . -t nitinbijlwan/reddit-clone 

(use this command to build a docker image.)



Step 4) Push the Image To DockerHub

- First login to your DockerHub account using Command i.e 

   docker login and give your username & password.

- Then use for pushing to the DockerHub.
   
    docker push nitinbijlwan/reddit-clone 

- You can use an existing docker image i.e
   
     nitinbijlwan/reddit-clone for deployment.

###################


 

Step 5) Write a Kubernetes Manifest File

- Write Deployment.yml file

apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  labels:
    app: reddit-clone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: reddit-clone
  template:
    metadata:
      labels:
        app: reddit-clone
    spec:
      containers:
      - name: reddit-clone
        image: nitinbijlwan/reddit-clone
        ports:
        - containerPort: 3000
########################################################

- Write Service.yml file

apiVersion: v1
kind: Service
metadata:
  name: reddit-clone-service
  labels:
    app: reddit-clone
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 31000
  selector:
    app: reddit-clone


 kubectl apply -f Deployment.yml
 kubectl apply -f Service.yml

 minikube svc reddit-clone-service --url
 curl -L <copy url>
 kubectl expose deployment reddit-clone-deployment --type=NodePort
 kubectl port-forward svc/reddit-clone-service 3000:3000 --address 0.0.0.0 &




Step 6) Let's Configure Ingress- For expose the deployment


- Ingress:

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-reddit-app
spec:
  rules:
  - host: "domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
  - host: "*.domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000


Minikube doesn't enable ingress by default; we have to enable it first using 
  minikube addons enable ingress
  minikube addons list

Verify that the ingress resource is running correctly by using 
  kubectl get ingress ingress-reddit-app 


Step 8) Expose the app


- First We need to expose our deployment so use
    kubectl expose deployment reddit-clone-deployment --type=NodePort

- You can test your deployment using
    curl -L http://192.168.49.2:31000. 192.168.49.2 is a minikube ip & Port 31000 is defined in Service.yml

- Then We have to expose our app service 
    kubectl port-forward svc/reddit-clone-service 3000:3000 --address 0.0.0.0 &

- Test Ingress
   curl -L domain.com/test

You can also see the deployed application on Ec2_ip:3000

Note:- Make sure you open the 3000 port in a security group of your Ec2 Instance.
