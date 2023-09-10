# Kubernetes in Action

## Chapter 1. Introducing Kubernetes

## Chapter 2. Understanding containers

### 2.1 Introducing containers

When a system consists of a small number of applications, it's okay to assign a dedicated virtual machine to each application and run each in its own operating system.
But as the microservices become smaller and their numbers start to grow, you may not be able to afford to give each one its own VM if you want to keep your hardware costs low and not waste resources.
It's not just a matter of wasting hardware resources - each VM typically needs to be individually configured and managed, which means that running higher numbers of VMs also results in higher staffing requirements and the need for a better, often more complicated automation system.
Due to the shift to microservice architectures, where systems consist of hundreds of deployed application instances, an alternative to VMs was needed.
Containers are that alternative.

## Chapter 3. Deploying your first application

### 3.1 Deploying a Kubernetes cluster

#### 3.1.1 Using the built-in Kubernetes cluster in Docker Desktop

If you use macOS or Windows, you've most likely installed Docker Desktop to run the exercises in the previous chapter.
It contains a single-node Kubernetes cluster that you can enable via its Settings dialog box.
This may be the easiest way for you to start your Kubernetes journey, but keep in mind that the version of Kubernetes may not be as recent as when using the alternative options described in the next sections.

### 3.2 Interacting with Kubernetes

You've now learned about several possible methods to deploy a Kubernetes cluster.
Now's the time to learn how to use the cluster.
To interact with Kubernetes, you use a commandline tool called `kubectl`, pronounced *kube-control*, *kube-C-T-L* or *kube-cuddle*.

As the next figure shows, the tool communicates with the Kubernetes API server, which is part of the Kubernetes Control Plane.
The control plane then triggers the other components to do whatever needs to be done based on the changes you made via the API.

#### 3.2.1 Setting up kubectl - the Kubernetes command-line client

#### 3.2.2 Configuring kubectl to use a specific Kubernetes cluster

#### 3.2.3 Using kubectl

Assuming you've installed and configured kubectl, you can now use it to talk to your cluster.

#### 3.2.4 Interacting with Kubernetes through web dashboards

### 3.3 Running your first application on Kubernetes

#### 3.3.1 Deploying your application

The imperative way to deploy an application is to use the `kubectl create deployment` command.
As the command itself suggests, it creates a *Deployment* object, which represents an application deployed in the cluster.
By using the imperative command, you avoid the need to know the structure of Deployment objects as when you write YAML or JSON manifests.

##### CREATING A DEPLOYMENT

##### LISTING DEPLOYMENTS

The interaction with Kubernetes consists mainly of the creation and manipulation of objects via its API.
Kubernetes stores these objects and then performs operations to bring them to life.
For example, when you create a Deployment object, Kubernetes runs an application.
Kubernetes then keeps you informed about the current state of the application by writing the status to the same Deployment object.
You can view the status by reading back the object.
One way to do this is to list all Deployment objects as follows:
```sh
kubectl get deployments
```

##### INTRODUCING PODS

#### 3.3.2 Exposing your application to the world

Your application is now running, so the next question to answer is how to access it.
I mentioned that each pod gets its own IP address, but this address is internal to the cluster
and not accessible from the outside.
To make the pod accessible externally, you'll *expose* it by creating a Service object.

#### 3.3.3 Horizontally scaling the application

## Chapter 4. Introducing Kubernetes API objects

## Chapter 5. Running workloads in Pods
