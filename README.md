# k8s-demo
Kubernetes Pods – Hands-on Practical
Objective
In this practical, you will learn how to:

Create and manage a Kubernetes Pod.
Inspect Pod details and status.
Access a container using kubectl exec.
View container logs.
Access an application using port forwarding.
Create a multi-container Pod.
Understand shared storage and networking between containers.
Understand Init Containers.
Observe how a Deployment replaces deleted Pods.
Prerequisites
Make sure you have:

A running Kubernetes cluster (Minikube, Kind, Docker Desktop Kubernetes, etc.).
kubectl installed and configured.
Basic knowledge of Docker containers and images.
Verify your setup:

kubectl version --client
kubectl cluster-info
kubectl get nodes
Exercise 1: Create Your First Pod
We will create an Nginx Pod using a YAML manifest.

Step 1: Create the YAML file
Create a file named nginx-pod.yaml.

Step 2: Create the Pod
kubectl apply -f nginx-pod.yaml
Step 3: Verify the Pod
kubectl get pods
kubectl get pods -o wide
Step 4: Inspect the Pod
kubectl describe pod nginx-pod
Observe:

Pod name
Pod IP address
Worker Node
Container image
Container status
Events
Exercise 2: Access the Container
Step 1: View container logs
kubectl logs nginx-pod
Step 2: Enter the container
kubectl exec -it nginx-pod -- sh
Execute the following commands inside the container:

whoami
hostname
pwd
ls
cat /etc/os-release
wget -qO- http://localhost
Exit the container:

exit
Step 3: Access Nginx from your browser
Run this command in a separate terminal:

kubectl port-forward pod/nginx-pod 8080:80
Open the following URL:

http://localhost:8080

You should see the Nginx welcome page.

Press Ctrl + C to stop port forwarding.

Exercise 3: Create a Multi-Container Pod
Objective
Create a Pod containing two containers:

Nginx: Serves a web page.
BusyBox: Updates the web page content every five seconds.
Both containers will share an emptyDir volume.

Step 1: Create the YAML file
Create multi-container-pod.yaml.

apiVersion: v1
kind: Pod
metadata:
  name: shared-web-pod
spec:
  volumes:
    - name: shared-data
      emptyDir: {}

  containers:
    - name: web
      image: nginx:alpine
      ports:
        - containerPort: 80
      volumeMounts:
        - name: shared-data
          mountPath: /usr/share/nginx/html

    - name: content-writer
      image: busybox:1.36
      command: ["/bin/sh", "-c"]
      args:
        - |
          while true; do
            date > /data/index.html
            sleep 5
          done
      volumeMounts:
        - name: shared-data
          mountPath: /data
Step 2: Create the Pod
kubectl apply -f multi-container-pod.yaml
Step 3: Check Pod status
kubectl get pods
kubectl describe pod shared-web-pod
Expected result:

NAME             READY   STATUS
shared-web-pod   2/2     Running
Step 4: View individual container logs
kubectl logs shared-web-pod -c web
kubectl logs shared-web-pod -c content-writer
