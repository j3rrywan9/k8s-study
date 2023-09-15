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

### 4.1 Getting familiar with the Kubernetes API

In a Kubernetes cluster, both users and Kubernetes components interact with the cluster by manipulating objects through the Kubernetes API, as shown in figure 4.1.

These objects represent the configuration of the entire cluster.
They include the applications running in the cluster, their configuration, the load balancers through which they are exposed within the cluster or externally, the underlying servers and the storage used by these applications, the security privileges of users and applications, and many other details of the infrastructure.

#### 4.1.1 Introducing the API

#### 4.1.2 Understanding the structure of an object manifest

Before you are confronted with the complete manifest of a Kubernetes object, let me first explain its major parts, because this will help you to find your way through the sometimes hundreds of lines it is composed of.

### 4.2 Examining an object's individual properties

#### 4.2.1 Exploring the full manifest of a Node object

#### 4.2.2 Understanding individual object fields

To learn more about individual fields in the manifest, you can refer to the API reference documentation at http://kubernetes.io/docs/reference/ or use the `kubectl explain` command as described next.

##### USING KUBECTL EXPLAIN TO EXPLORE API OBJECT FIELDS

The kubectl tool has a nice feature that allows you to look up the explanation of each field for each object type (kind) from the command line.
Usually, you start by asking it to provide the basic description of the object kind by running `kubectl explain <kind>`, as shown here:
```sh
kubectl explain nodes
```

#### 4.2.3 Understanding an object's status conditions

The set of fields in both the `spec` and `status` sections is different for each object kind, but the `conditions` field is found in many of them.
It gives a list of conditions the object is currently in.
They are very useful when you need to troubleshoot an object, so let's examine them more closely.
Since the Node object is used as an example, this section also teaches you how to easily identify problems with a cluster node.

#### 4.2.4 Inspecting objects using the kubectl describe command

### 4.3 Observing cluster event via Event objects

## Chapter 5. Running workloads in Pods

### 5.1 Understanding pods

You've already learned that a pod is a co-located group of containers and the basic building block in Kubernetes.
Instead of deploying containers individually, you deploy and manage a group of containers as a single unit - a pod.
Although pods may contain several, it's not uncommon for a pod to contain just a single container.
When a pod has multiple containers, all of them run on the same worker node - a single pod instance never spans multiple nodes.

#### 5.1.1 Understanding why we need pods

Imagine an application that consists of several processes that communicate with each other via *IPC* (Inter-Process Communication) or shared files, which requires them to run on the same computer.
In chapter 2, you learned that each container is like an isolated computer or virtual machine.
A computer typically runs several processes; containers can also do this.
You can run all the processes that make up an application in just one container, but that makes the container very difficult to manage.

Containers are *designed* to run only a single process, not counting any child processes that it spawns.
Both container tooling and Kubernetes were developed around this fact.
For example, a process running in a container is expected to write its logs to standard output.
Docker and Kubernetes commands that you use to display the logs only show what has been captured from this output.
If a single process is running in the container, it's the only writer,

Another indication that containers should only run a single process is the fact that the container runtime only restarts the container when the container's root process dies.
It doesn't care about any child processes created by this root process.
If it spawns child processes, it alone is responsible for keeping all these processes running.

To take full advantage of the features provided by the container runtime, you should consider running only one process in each container.

##### UNDERSTANDING HOW A POD COMBINES MULTIPLE CONTAINERS

Since you shouldn't run multiple processes in a single container, it's evident you need another higher-level construct that allows you to run related processes together even when divided into multiple containers.
These processes must be able to communicate with each other like processes in a normal computer.
And that is why pods were introduced.

#### 5.1.2 Organizing containers into pods

You can think of each pod as a separate computer.
Unlike virtual machines, which typically host multiple applications, you typically run only one application in each pod.
You never need to combine multiple applications in a single pod, as pods have almost no resource overhead.
You can have as many pods as you need, so instead of stuffing all your applications into a single pod, you should divide them so that each pod runs only closely related application processes.

##### SPLITTING A MULTI-TIER APPLICATION STACK INTO MULTIPLE PODS

##### INTRODUCING SIDECAR CONTAINERS

Placing several containers in a single pod is only appropriate if the application consists of a primary process and one or more processes that complement the operation of the primary process.
The container in which the complementary process runs is called a *sidecar container* because it's analogous to a motorcycle sidecar, which makes the motorcycle more stable and offers the possibility of carrying an additional passenger.
But unlike motorcycles, a pod can have more than one sidecar, as shown in figure 5.5.

Other examples of sidecar containers are log rotators and collectors, data processors, communication adapters, and others.

Unlike changing the application's existing code, adding a sidecar increases the pod's resources requirements because an additional process must run in the pod.
But keep in mind that adding code to legacy applications can be very difficult.
This could be because its code is difficult to modify, it's difficult to set up the build environment, or the source code itself is no longer available.
Extending the application by adding an additional process is sometimes a cheaper and faster option.

### 5.2 Creating pods from YAML or JSON files

### 5.3 Interacting with the application and the pod

### 5.4 Running multiple containers in a pod

The Kiada application you deployed in section 5.2 only supports HTTP.
Let's add TLS support so it can also serve clients over HTTPS.
You could do this by adding code to the `app.js` file, but an easier option exists where you don't need to touch the code at all.

### 5.5 Running additional containers at pod startup

#### 5.5.1 Introducing init containers

## Chapter 6. Managing the Pod lifecycle

## Chapter 7. Attaching storage volumes to Pods

## Chapter 8. Persisting data in PersistentVolumes

## Chapter 9. Configuration via ConfigMaps, Secrets, and the Downward API

### 9.1 Setting the command, arguments, and environment variables

Like regular applications, containerized applications can be configured using command-line arguments, environment variables, and files.

Hardcoding the configuration into the container image is the same as hardcoding it into the application source code.
This is not ideal because you must rebuild the image every time you change the configuration.
Also, you should never include sensitive configuration data such as security credentials or encryption keys in the container image because anyone who has access to it can easily extract them.

### 9.2 Using a config map to decouple configuration from the pod

To reuse the same pod definition in multiple environments, it's better to decouple the configuration from the pod manifest.
One way to do this is to move the configuration into a ConfigMap object, which you then reference in the pod manifest.
This is what you'll do next.

#### 9.2.1 Introducing ConfigMaps

A ConfigMap is a Kubernetes API object that simply contains a list of key/value pairs.
The values can range from short strings to large blocks of structured text that you typically find in an application configuration file.
Pods can reference one or more of these key/value entries in the config map.
A pod can refer to multiple config maps, and multiple pods can use the same config map.

#### 9.2.2 Creating a ConfigMap object

#### 9.2.3 Injecting config map values into environment variables
