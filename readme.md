# Kubernetes-Declaractively-Deploying-Infrastructure-with-Minikube

## Intro

Let’s say you have multiple houses that you want to manage, each with their own set of unique features and requirements. It would be a difficult and time-consuming task to manage all of them manually, making sure that everything is running smoothly and making changes as needed.

This is where Kubernetes comes in. Kubernetes is like a property manager that helps you manage multiple houses at once. Just like a property manager oversees the maintenance and operations of multiple properties, Kubernetes helps you manage and orchestrate multiple Docker containers at scale.

In this example, Kubernetes would be like the property manager who oversees multiple houses. Each house would be a Docker container that contains all the necessary components to run the application. Kubernetes would help you manage and automate the deployment, scaling, and monitoring of all these containers, making sure that everything is running smoothly and efficiently.

So if one of the houses needs some maintenance work done, like fixing a leaky faucet. With Kubernetes, you can automatically spin up a new container with the fixed faucet, while the old container is taken down for maintenance.

This ensures that the application remains available and operational, even while maintenance work is being done.

Now let’s get into the more technical concepts of Kubernetes.



# Kubernetes Info

## Kubernetes
Kubernetes is an open-source container orchestration tool at heart. It helps automate managing and deploying containerized applications, making it easier for developers to build, deploy and manage their applications.

## Minikube 
Minikube is very lightweight Kubernetes application that deploys a Kubernetes cluster on a virtual machine hosted on a local system. It supports all major operation systems (Mac, Windows and Linux).

## Pod 
A Pod is the smallest deployable unit in Kubernetes and represents a single instance of a running process in a cluster. It can contain one or more containers that share the same network namespace and are deployed and managed together.

## Service
A Service provides a stable IP address and DNS name for a set of Pods and allows for the expose of these Pods to be accessed by other parts of the application or by external clients.

## Deployment
A Deployment manages a set of replicas of a Pod and allows you to declaratively define and manage the desired state of a group of Pods. It can also automatically create or delete replicas to match the desired state.

## ConfigMap
A ConfigMap stores configuration data as key-value pairs or as plain text files in a data volume. It allows you to separate configuration data from your application code and makes it easier to manage configuration changes without redeploying your application.

# Kubernetes Commands

## kubctl get
Kubectl get retrieves information about Kubernetes resources, such as Pods, Services, Deployments, ConfigMaps etc.

## kubctl apply
kubctl apply is used to apply configuration changes to Kubernetes resources. It takes a YAML or JSON file that defines the desired state of a resource and applies the changes to the cluster.

## Kubectl deploy
Kubectl deploy is used to deploy a new version of an application to a Kubernetes cluster. It creates a new Deployment resource with the desired number of replicas and updates the running Pods to match the new version of the application.

## Kubctl create
Kubectl create is used to create new Kubernetes resources, such as Pods, Services, Deployments, ConfigMaps, etc. It also takes a YAML or JSON file that defines the desired state of the resource and creates the resource in the cluster.

## Kubctl describe
Kubectl describe is used to display detailed information about a Kubernetes resource, such as the resource’s labels, annotations, events and status. It is also useful for troubleshooting and debugging issues with Kubernetes resources.

## Prerequisites
1.Basic knowledge and understanding of Containerization and Docker

2.Basic Linux command line knowledge

3.Basic knowledge and use of an Interactive Development Environment (IDE).

## Objectives
1.Spin up a multi-container Pod in Kubernetes is a Pod that runs multiple containers sharing the same execution environment, network, and storage. This pattern is useful when containers need to work closely together.

## Step 0: Setup Kubernetes environment with miniKube
Head to this link `https://minikube.sigs.k8s.io/docs/start/?arch=%2Fwindows%2Fx86-64%2Fstable%2F.exe+download` to learn how to get miniKube up and running on your local system. For this demonstration, I will be installing and running miniKube on my local Windows system using Chocolatey.

Run the following command to install miniKube
```language
choco install minikube
```

Verify the installation: Open PowerShell or Command Prompt and check the Minikube version:
```language
minikube version
```

Start Minikube: To start Minikube, run:
```language
minikube start
```

You should observe miniKube running through a series of steps to create the Kubernetes Cluster and node, as seen below.

![image_alt]()

We can now verify that Kubernetes Cluster is ready by running the following command which displays the running Client and Server

```language
kubectl version --short
```

The output will include the version numbers of both the Client and Server you’re connected to.

![image_alt]()


Run the followings commands in succession to verify the a node and default ClusterIP service was created when deploying the Kubernetes cluster 

```language
kubectl get all
```


```language
kubectl get nodes
```

You will see similar results as shown below. One default Service of type `ClusterIP` and one `minikube` Node that is the master/control plane.

![image_alt]()

Now that we’ve set up our environment, let’s proceed to Step 1 — Creating the Deploying Pods from a manifest file using YAML to leverage IaC.

## Step 1: Create Kubernetes Multi-Container Pod with Sidecar Pattern
A multi-container Pod in Kubernetes is a Pod that runs multiple containers sharing the same execution environment, network, and storage. This pattern is useful when containers need to work closely together.
A common implementation is the sidecar pattern, where a secondary container supports the main application without modifying its core functionality.

In this example, we deploy a Kubernetes Pod where:
`The main container (ctr-web)` runs an Nginx web server serving static content.

`The sidecar container (ctr-sync)` automatically syncs content from a GitHub repository.

This setup ensures the web server always has the latest content without requiring manual updates.

## Architecture
1.Web Container (ctr-web): Runs Nginx and serves static content from a shared volume.

2.Sidecar Container (ctr-sync): Uses Git-Sync to pull content from a remote Git repository into the shared volume.

3.Shared Volume (emptyDir): Allows both containers to access and update the same files.

![image_alt]()


## Kubernetes YAML Configuration

Here I will use the nano text editor to create a Kubernetes sidecarpod.yml  YAML file by running the following command


```language
nano sidecarpod.yml
```

A text editor window should open. Copy and paste the code below into the text editor

```language

apiVersion: v1
kind: Pod
metadata:
  name: git-sync
  labels:
    app: sidecar
spec:
  containers:
  - name: ctr-web
    image: nginx
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/
  - name: ctr-sync
    image: k8s.gcr.io/git-sync:v3.1.6
    volumeMounts:
    - name: html
      mountPath: /tmp/git
    env:
    - name: GIT_SYNC_REPO
      value: https://github.com/Tatenda-Prince/ps-sidecar
    - name: GIT_SYNC_BRANCH
      value: master
    - name: GIT_SYNC_DEPTH
      value: "1"
    - name: GIT_SYNC_DEST
      value: "html"
  volumes:
  - name: html
    emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: svc-sidecar
spec:
  selector:
    app: sidecar
  ports:
  - port: 80
  type: LoadBalancer
```



