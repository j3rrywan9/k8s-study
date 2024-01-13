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

When a pod contains more than one container, all the containers are started in parallel.
Kubernetes doesn't yet provide a mechanism to specify whether a container depends on
another container, which would allow you to ensure that one is started before the other.
However, Kubernetes allows you to run a sequence of containers to initialize the pod before its main containers start.
This special type of container is explained in this section.

#### 5.5.1 Introducing init containers

A pod manifest can specify a list of containers to run when the pod starts and before the pod's normal containers are started.
These containers are intended to initialize the pod and are appropriately called *init containers*.
As the following figure shows, they run one after the other and must all finish successfully before the main containers of the pod are started.

#### 5.5.2 Adding init containers to a pod

In a pod manifest, init containers are defined in the `initContainers` field in the spec
section, just as regular containers are defined in its `containers` field.

## Chapter 6. Managing the Pod lifecycle

### 6.1 Understanding the pod's status

After you create a pod object and it runs, you can see what's going on with the pod by
reading the pod object back from the API.
As you've learned in chapter 4, the pod object manifest, as well as the manifests of most other kinds of objects, contain a section, which provides the status of the object.
A pod's status section contains the following information:

### 6.2 Keeping containers healthy

The pods you created in the previous chapter ran without any problems.
But what if one of the containers dies?
What if all the containers in a pod die?
How do you keep the pods healthy and their containers running?
That's the focus of this section.

#### 6.2.1 Understanding container auto-restart

When a pod is scheduled to a node, the Kubelet on that node starts its containers and from then on keeps them running for as long as the pod object exists.
If the main process in the container terminates for any reason, the Kubelet restarts the container.
If an error in your application causes it to crash, Kubernetes automatically restarts it, so even without doing anything special in the application itself, running it in Kubernetes automatically gives it the ability to heal itself.
Let's see this in action.

#### 6.2.2 Checking the container's health using liveness probes

In the previous section, you learned that Kubernetes keeps your application healthy by
restarting it when its process terminates.
But applications can also become unresponsive without terminating.
For example, a Java application with a memory leak eventually starts spewing out OutOfMemoryErrors, but its JVM process continues to run.
Ideally, Kubernetes should detect this kind of error and restart the container.

#### 6.2.3 Creating an HTTP GET liveness probe

### 6.3 Executing actions at container start-up and shutdown

### 6.4 Understanding the pod lifecycle

## Chapter 7. Attaching storage volumes to Pods

The previous two chapters focused on the pod's containers, but they are only half of what a pod typically contains.
They are typically accompanied by storage volumes that allow a pod's containers to store data for the lifetime of the pod or beyond, or to share files with the other
containers of the pod.
This is the focus of this chapter.

### 7.1 Introducing volumes

A pod is like a small logical computer that runs a single application.
This application can consist of one or more containers that run the application processes.
These processes share computing resources such as CPU, RAM, network interfaces, and others.
In a typical computer, the processes use the same filesystem, but this isn't the case with containers.
Instead, each container has its own isolated filesystem provided by the container image.

When a container starts, the files in its filesystem are those that were added to its
container image during build time.
The process running in the container can then modify those files or create new ones.
When the container is terminated and restarted, all changes it made to its files are lost, because the previous container is not really restarted, but completely replaced, as explained in the previous chapter.
Therefore, when a containerized application is restarted, it can't continue from the point where it was when it stopped.
Although this may be okay for some types of applications, others may need the entire filesystem or at least part of it to be preserved on restart.

This is achieved by adding a *volume* to the pod and *mounting* it into the container.

#### 7.1.1 Demonstrating the need for volumes

In this chapter, you'll build a new service that requires its data to be persisted.
To do this,
the pod that runs the service will need to contain a volume.
But before we get to that, let me tell you about this service, and allow you to experience first-hand why it can't work without a volume.

##### INTRODUCING THE QUIZ SERVICE

The Quiz service consists of a RESTful API frontend and a MongoDB database as the
backend.
Initially, you'll run these two components in separate containers of the same pod, as shown in the following figure.

As I explained in the pod introduction in chapter 5, creating pods like this is not the best
idea, as it doesn't allow for the containers to be scaled individually.
The reason we'll use a single pod is because you haven't yet learned the correct way to make pods communicate with each other.
You'll learn this in chapter 11.
That's when you'll split the two containers into separate pods.

##### BUILDING THE QUIZ API CONTAINER

#### 7.1.2 Understanding how volumes fit into pods

Like containers, volumes aren't top-level resources like pods or nodes, but are a component
within the pod and thus share its lifecycle.
As the following figure shows, a volume is defined at the pod level and then mounted at the desired location in the container.

The lifecycle of a volume is tied to the lifecycle of the entire pod and is independent of the
lifecycle of the container in which it is mounted.
Due to this fact, volumes are also used to persist data across container restarts.

##### PERSISTING FILES ACROSS CONTAINER RESTARTS

All volumes in a pod are created when the pod is set up - before any of its containers are
started.
They are torn down when the pod is shut down.

Each time a container is (re)started, the volumes that the container is configured to use
are mounted in the container's filesystem.
The application running in the container can read from the volume and write to it if the volume and mount are configured to be writable.

A typical reason for adding a volume to a pod is to persist data across container restarts.
If no volume is mounted in the container, the entire filesystem of the container is ephemeral.
Since a container restart replaces the entire container, its filesystem is also re-created from
the container image.
As a result, all files written by the application are lost.

If, on the other hand, the application writes data to a volume mounted inside the container, as shown in the following figure, the application process in the new container can access the same data after the container is restarted.

It is up to the author of the application to determine which files must be retained on restart.
Normally you want to preserve data representing the application's state, but you may not
want to preserve files that contain the application's locally cached data, as this prevents the
container from starting fresh when it's restarted.
Starting fresh every time may allow the application to heal itself when corruption of the local cache causes it to crash.
Just restarting the container and using the same corrupted files could result in an endless crash loop.

##### MOUNTING MULTIPLE VOLUMES IN A CONTAINER

A pod can have multiple volumes and each container can mount zero or more of these volumes in different locations, as shown in the following figure.

The reason why you might want to mount multiple volumes in one container is that these
volumes may serve different purposes and can be of different types with different performance characteristics.

In pods with more than one container, some volumes can be mounted in some containers but not in others.
This is especially useful when a volume contains sensitive information that should only be accessible to some containers.

##### SHARING FILES BETWEEN MULTIPLE CONTAINERS

A volume can be mounted in more than one container so that applications running in these
containers can share files.
As discussed in chapter 5, a pod can combine a main application container with sidecar containers that extend the behavior of the main application.
In some cases, the containers must read or write the same files.

For example, you could create a pod that combines a web server running in one container with a content-producing agent running in another container.
The content agent container generates the static content that the web server then delivers to its clients.
Each of the two containers performs a single task that has no real value on its own.
However, as the next figure shows, if you add a volume to the pod and mount it in both containers, you enable these containers to become a complete system that provides a valuable service and is more than the sum of its parts.

In the figure you'll also notice that the volume mount in each container can be configured
either as read/write or as read-only.
Because the content agent needs to write to the volume whereas the web server only reads from it, the two mounts are configured differently.
In the interest of security, it's advisable to prevent the web server from writing to the volume, since this could allow an attacker to compromise the system if the web server software has a
vulnerability that allows attackers to write arbitrary files to the filesystem and execute them.

Other examples of using a single volume in two containers are cases where a sidecar container runs a tool that processes or rotates the web server logs or when an init container creates configuration files for the main application container.

### 7.2 Using an `emptyDir` volume

The simplest volume type is `emptyDir`.
As its name suggests, a volume of this type starts as an empty directory.
When this type of volume is mounted in a container, files written by the application to the path where the volume is mounted are preserved for the duration of the pod's existence.

This volume type is used in single-container pods when data must be preserved even if the container is restarted.
It's also used when the container's filesystem is marked read-only, and you want part of it to be writable.
In pods with two or more containers, an `emptyDir` volume is used to share data between them.

#### 7.2.1 Persisting files across container restarts

##### ADDING AN EMPTYDIR VOLUME TO A POD

Two changes to the pod manifest are required to achieve this:
1. An `emptyDir` volume must be added to the pod.
1. The volume must be mounted into the container.

The following listing shows the new pod manifest with these two changes highlighted in bold.
You'll find the manifest in the file `pod.quiz.emptydir.yaml`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: quiz
spec:
  volumes:
  - name: quiz-data
    emptyDir: {}
containers:
  - name: quiz-api
    image: luksa/quiz-api:0.1
    ports:
    - name: http
      containerPort: 8080
  - name: mongo
    image: mongo
    volumeMounts:
    - name: quiz-data
      mountPath: /data/db
```
The listing shows that an `emptyDir` volume named `quiz-data` is defined in the `spec.volumes` array of the pod manifest and that it is mounted into the mongo container's filesystem at the location `/data/db`.
The following two sections explain more about the volume and the volume mount definitions.

##### CONFIGURING THE EMPTYDIR VOLUME

In general, each volume definition must include a name and a type, which is indicated by the
name of the nested field (for example: `emptyDir`, `gcePersistentDisk`, `nfs`, and so on).
This field typically contains several sub-fields that allow you to configure the volume.
The set of sub-fields that you set depends on the volume type.

##### MOUNTING THE VOLUME IN A CONTAINER

Defining a volume in the pod is only half of what you need to do to make it available in a
container.
The volume must also be mounted in the container.
This is done by referencing the volume by name in the `volumeMounts` array in the container definition.

In addition to the name, a volume mount definition must also include the `mountPath` - the
path within the container where the volume should be mounted.
In listing 7.2, the volume is mounted at `/data/db` because that's where MongoDB stores its files.
You want these files to be written to the volume instead of the container's filesystem, which is ephemeral.

The full list of supported fields in a volume mount definition is presented in the following
table.

##### UNDERSTANDING THE LIFESPAN OF AN EMPTYDIR VOLUME

If you replace the quiz pod with the one in listing 7.2 and insert questions into the database,
you'll notice that the questions you add to the database remain intact, regardless of how
often the container is restarted.
This is because the volume's lifecycle is tied to that of the pod.

The directory is typically located at the following location in the node's filesystem:
```
/var/lib/kubelet/pods/<pod_UID>/volumes/kubernetes.io~empty-dir/<volume_name>
```
The `pod_UID` is the unique ID of the pod, which you'll find the Pod object's `metadata` section.
If you want to see the directory for yourself, run the following command to get the `pod_UID`:
```sh
kubectl get po quiz -o json | jq .metadata.uid
```

##### UNDERSTANDING WHERE THE FILES IN AN EMPTYDIR VOLUME ARE STORED

As you can see in the following figure, the files in an `emptyDir` volume are stored in a directory in the host node's filesystem.
It's nothing but a normal file directory.
This directory is mounted into the container at the desired location.

#### 7.2.2 Populating an `emptyDir` volume with data using an init container

Every time you create the quiz pod from the previous section, the MongoDB database is empty, and you have to insert the questions manually.
Let's improve the pod by automatically populating the database when the pod starts.

Many ways of doing this exist.
You could run the MongoDB container locally, insert the data, commit the container state into a new image and use that image in your pod.
But then you'd have to repeat the process every time a new version of the MongoDB container image
is released.

Fortunately, the MongoDB container image provides a mechanism to populate the database the first time it's started.
On start-up, if the database is empty, it invokes any `.js` and `.sh` files that it finds in the `/docker-entrypoint-initdb.d` directory.
All you need to do is get the file into that location.
Again, you could build a new MongoDB image with the file in that location, but you'd run into the same problem as described previously.
An alternative solution is to use a volume to inject the file into that location of the MongoDB container's filesystem.
But how do you get the file into the volume in the first place?

Kubernetes provides a special type of volume that is initialized by cloning a Git repository - the `gitRepo` volume.
However, this type of volume is now deprecated.
The proposed alternative is to use an `emptyDir` volume that you initialize with an init container that executes the `git clone` command.
You could use this approach, but this would mean that the pod must make a network call to fetch the data.

Another, more generic way of populating an `emptyDir` volume, is to package the data into a container image and copy the data files from the container to the volume when the container starts.
This removes the dependency on any external systems and allows the pod to run regardless of the network connectivity status.

To help you visualize the pod, look at the following figure.

When the pod starts, first the volumes and then the init container is created.
The `initdb` volume is mounted into this init container.
The container image contains the `insert-questions.js` file, which the container copies to the volume when it runs.
Then the copy operation is complete, the init container finishes and the pod's main containers are started.
The `initdb` volume is mounted into the `mongo` container at the location where MongoDB looks for database initialization scripts.
On first start-up, MongoDB executes the `insert-questions.js` script.
This inserts the questions into the database.
As in the previous version of the pod, the database files are stored in the `quiz-data` volume to allow the data to survive container restarts.

#### 7.2.3 Sharing files between containers

## Chapter 8. Persisting data in PersistentVolumes

## Chapter 9. Configuration via ConfigMaps, Secrets, and the Downward API

You've now learned how to use Kubernetes to run an application process and attach file
volumes to it.
In this chapter, you'll learn how to configure the application - either in the pod
manifest itself, or by referencing other API objects within it.
You'll also learn how to inject information about the pod itself into the application running inside it.

### 9.1 Setting the command, arguments, and environment variables

Like regular applications, containerized applications can be configured using command-line arguments, environment variables, and files.

You learned that the command that is executed when a container starts is typically
defined in the container image.
The command is configured in the container's Dockerfile using the `ENTRYPOINT` directive, while the arguments are typically specified using the `CMD` directive.
Environment variables can also be specified using the the `ENV` directive in the Dockerfile.
If the application is configured using configuration files, these can be added to the container image using the `COPY` directive.
You’ve seen several examples of this in the previous chapters.

Hardcoding the configuration into the container image is the same as hardcoding it into the application source code.
This is not ideal because you must rebuild the image every time you change the configuration.
Also, you should never include sensitive configuration data such as security credentials or encryption keys in the container image because anyone who has access to it can easily extract them.

Instead, it's much safer to store these files in a volume that you mount in the container.
As you learned in the previous chapter, one way to do this is to store the files in a persistent volume.
Another way is to use an `emptyDir` volume and an init container that fetches the files from secure storage and writes them to the volume.
You should know how to do this if you've read the previous chapters, but there's a better way.
In this chapter, you'll learn how to use special volume types to achieve the same result without using init containers.
But first, let's learn how to change the command, arguments, and environment variables without recreating the container image.

#### 9.1.1 Setting the command and arguments

### 9.2 Using a config map to decouple configuration from the pod

In the previous section, you learned how to hardcode configuration directly into your pod manifests.
While this is much better than hard-coding in the container image, it's still not ideal because it means you might need a separate version of the pod manifest for each environment you deploy the pod to, such as your development, staging, or production cluster.

To reuse the same pod definition in multiple environments, it's better to decouple the configuration from the pod manifest.
One way to do this is to move the configuration into a ConfigMap object, which you then reference in the pod manifest.
This is what you'll do next.

#### 9.2.1 Introducing ConfigMaps

A `ConfigMap` is a Kubernetes API object that simply contains a list of key/value pairs.
The values can range from short strings to large blocks of structured text that you typically find in an application configuration file.
Pods can reference one or more of these key/value entries in the config map.
A pod can refer to multiple config maps, and multiple pods can use the same config map.

To keep applications Kubernetes-agnostic, they typically don't read the `ConfigMap` object via the Kubernetes REST API.
Instead, the key/value pairs in the config map are passed to containers as environment variables or mounted as files in the container's filesystem via a `configMap` volume, as shown in the following figure.

#### 9.2.2 Creating a ConfigMap object

#### 9.2.3 Injecting config map values into environment variables

## Chapter 10. Organizing objects using Namespaces and Labels

A Kubernetes cluster is usually used by many teams.
How should these teams deploy objects to the same cluster and organize them so that one team doesn't accidentally modify the objects created by other teams?

And how can a large team deploying hundreds of microservices organize them so that each team member, even if new to the team, can quickly see where each object belongs and what its role in the system is?
For example, to which application does a config map or a secret belong.

These are two different problems.
Kubernetes solves the first with object namespaces, and the other with object labels.
In this chapter, you will learn about both.

### 10.1 Organizing objects into Namespaces

#### 10.1.1 Listing namespaces and the objects they contain

#### 10.1.2 Creating namespaces

#### 10.1.3 Managing objects in other namespaces

### 10.2 Organizing pods with labels

#### 10.2.1 Introducing labels

Labels are an incredibly powerful yet simple feature for organizing Kubernetes API objects.
A label is a key-value pair you attach to an object that allows any user of the cluster to identify
the object's role in the system.
Both the key and the value are simple strings that you can specify as you wish.
An object can have more than one label, but the label keys must be unique within that object.
You normally add labels to objects when you create them, but you can also change an object's labels later.

### 10.3 Filtering objects with label selectors

### 10.4 Annotating objects

## Chapter 11. Exposing Pods with Services

Instead of running a single pod to provide a particular service, people nowadays typically run
several replicas of the pod so that the load can be distributed across multiple cluster nodes.
But that means all pod replicas providing the same service should be reachable at a single
address so clients can use that single address, rather than having to keep track of and
connect directly to individual pod instances.
In Kubernetes, you do that with Service objects.

Each pod has its own network interface with its own IP address.

### 11.1 Exposing pods via services

#### 11.1.1 Introducing services

A Kubernetes Service is an object you create to provide a single, stable access point to a set
of pods that provide the same service.
Each service has a stable IP address that doesn't change for as long as the service exists.
Clients open connections to that IP address on one of the exposed network ports, and those connections are then forwarded to one of the pods that back that service.
In this way, clients don't need to know the addresses of the individual pods providing the service, so those pods can be scaled out or in and moved from one cluster node to the other at will.
A service acts as a load balancer in front of those pods.

##### UNDERSTANDING WHY YOU NEED SERVICES

By creating a service for the Kiada pods and configuring it to be reachable from outside the
cluster, you create a single, constant IP address through which external clients can connect
to the pods.
Each connection is forwarded to one of the kiada pods.

By creating a service for the Quote pods, you create a stable IP address through which
the Kiada pods can reach the Quote pods, regardless of the number of pod instances behind
the service and their location at any given time.

Although there's only one instance of the Quiz pod, it too must be exposed through a
service, since the pod's IP address changes every time the pod is deleted and recreated.
Without a service, you'd have to reconfigure the Kiada pods each time or make the pods get
the Quiz pod's IP from the Kubernetes API.
If you use a service, you don't have to do that because its IP address never changes.
