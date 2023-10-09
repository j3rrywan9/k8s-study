# Kubernetes: Up and Running, 3rd Edition

## Chapter 1. Introduction

Kubernetes is an open source orchestrator for deploying containerized applications.

Kubernetes is a proven infrastructure for distribute dsystems that is suitable for cloud-native developers of all scales, from a cluster of Raspberry Pi computers to a warehouse full of the latest machines.
It provides the software necessary to successfully build and deploy reliable, scalable distributed systems.

Depending on when and why you have come to hold this book in your hands, you may have varying degrees of experience with containers, distributed systems, and Kubernetes.
You may be planning on building your application on top of public cloud infrastructure, in private data centers, or in some hybrid environment.
Regardless of what your experience is, we believe this book will enable you to make the most of your use of Kubernetes.

There are many reasons why people come to use containers and container APIs like Kubernetes, but we believe they can all be traced back to one of these benefits:
* Velocity
* Scaling (of both software and teams)
* Abstracting your infrastructure
* Efficiency

In the following sections, we describe how Kubernetes can help provide each of these features.

### Velocity

This changing landscape means that the difference between you and your competitors is often the speed with which you can develop and deploy new components and features, or the speed with which you can respond to innovations developed by others.

It is important to note, however, that velocity is not defined in terms of simply raw speed.
While your users are always looking for iterative improvement, they are more interested in a highly reliable service.
Once upon a time, it was OK for a service to be down for maintenance at midnight every night.
But today, all users expect constant uptime, even if the software they are running is changing constantly.

Consequently, velocity is measured not in terms of the raw number of features you can ship per hour or day, but rather in terms of the number of things you can ship while maintaining a highly available service.

In this way, containers and Kubernetes can provide the tools that you need to move quickly, while staying available.
The core concepts that enable this are:
* Immutability
* Declarative configuration
* Online self-healing systems
These ideas all interrelate to radically improve the speed with which you can reliably deploy software.

#### The Value of Immutability

Containers and Kubernetes encourage developers to build distributed systems that adhere to the principles of immutable infrastructure.
With immutable infrastructure, once an artifact is created in the system it does not change via user modifications.

#### Declarative Configuration

Immutability extends beyond containers running in your cluster to the way you describe your application to Kubernetes.
Everything in Kubernetes is a *declarative configuration object* that represents the desired state of the system.
It is the job of Kubernetes to ensure that the actual state of the world matches this desired state.

Much like mutable versus immutable infrastructure, declarative configuration is an alternative to *imperative* configuration, where the state of the world is defined by the execution of a series of instructions rather than a declaration of the desired state of the world.
While imperative commands define actions, declarative configurations define state.

#### Self-Healing Systems

Kubernetes is an online, self-healing system.
When it receives a desired state configuration, it does not simply take a set of actions to make the current state match the desired state a single time.
It *continuously* takes actions to ensure that the current state matches the desired state.
This means that not only will Kubernetes initialize your system, but it will guard it against any failures or perturbations that might destabilize the system and affect reliability.

Online self-healing systems improve developer velocity because the time and energy you might otherwise have spent on operations and maintenance can instead be spent on developing and testing new features.

In a more advanced form of self-healing, there has been significant recent work in the *operator* paradigm for Kubernetes.
With operators, more advanced logic needed to maintain, scale, and heal a specific piece of software (MySQL, for example) is encoded into an operator application that runs as a container in the cluster.
The code in the operator is responsible for more targeted and advanced health detection and healing than can be achieved via Kubernetes's generic self-healing.
Often this is packaged up as "operators," which are discussed in a later section.

### Scaling Your Service and Your Teams

As your product grows, it's inevitable that you will need to scale both your software and the teams that develop it.
Fortunately, Kubernetes can help with both of these goals.
Kubernetes achieves scalability by favoring *decoupled* architectures.

## Chapter 2. Creating and Running Containers

### Container Images

## Chapter 3. Deploying a Kubernetes Cluster

## Chapter 4. Common `kubectl` Commands

### Namespaces

Kubernetes uses *namespaces* to organize objects in the cluster.
You can think of each namespace as a folder that holds a set of objects.
By default, the kubectl command-line tool interacts with the default namespace.
If you want to use a different namespace, you can pass `kubectl` the `--namespace` flag.

### Contexts

### Viewing Kubernetes API Objects

Everything contained in Kubernetes is represented by a RESTful resource.
Throughout this book, we refer to these resources as *Kubernetes objects*.
Each Kubernetes object exists at a unique HTTP path;
for example, https://your-k8s.com/api/v1/namespaces/default/pods/my-pod leads to the representation of a Pod in the default namespace named `my-pod`.
The `kubectl` command makes HTTP requests to these URLs to access the Kubernetes objects that reside at these paths.

The most basic command for viewing Kubernetes objects via `kubectl` is `get`.
If you run `kubectl get <resource-name>` you will get a listing of all resources in the current namespace.
If you want to get a specific resource, you can use `kubectl get <resource-name> <obj-name>`.

## Chapter 5. Pods

Consequently, Kubernetes groups multiple containers into a single atomic unit called a *Pod*.

### Pods in Kubernetes

A Pod is a collection of application containers and volumes running in the same execution environment.
Pods, not containers, are the smallest deployable artifact in a Kubernetes cluster.
This means all of the containers in a Pod always land on the same machine.

Each container within a Pod runs in its own cgroup, but they share a number of Linux namespaces.

Applications running in the same Pod share the same IP address and port space (network namespace), have the same hostname (UTS namespace), and can communicate using native interprocess communication channels over System V IPC or POSIX message queues (IPC namespace).
However, applications in different Pods are isolated from each other;
they have different IP addresses, hostnames, and more.
Containers in different Pods running on the same node might as well be on different servers.

### Thinking with Pods

In general, the right question to ask yourself when designing Pods is "Will these containers work correctly if they land on different machines?"
If the answer is no, a Pod is the correct grouping for the containers.
If the answer is yes, using multiple Pods is probably the correct solution.
In the example at the beginning of this chapter, the two containers interact via a local filesystem.
It would be impossible for them to operate correctly if the containers were scheduled on different machines.

In the remaining sections of this chapter, we will describe how to create, introspect, manage, and delete Pods in Kubernetes.

### The Pod Manifest

Pods are described in a Pod *manifest*, which is just a text-file representation of the Kubernetes API object.
Kubernetes strongly believes in *declarative configuration*, which means that you write down the desired state of the world in a configuration and then submit that configuration to a service that takes actions to ensure the desired state becomes the actual state.

The Kubernetes API server accepts and processes Pod manifests before storing them in persistent storage (`etcd`).
The scheduler also uses the Kubernetes API to find Pods that haven't been scheduled to a node.
It then places the Pods onto nodes depending on the resources and other constraints expressed in the Pod manifests.
The scheduler can place multiple Pods on the same machine as long as there are sufficient resources.
However, scheduling multiple replicas of the same application onto the same machine is worse for reliability, since the machine is a single failure domain.
Consequently, the Kubernetes scheduler tries to ensure that Pods from the same application are distributed onto different machines for reliability in the presence of such failures.
Once scheduled to a node, Pods don't move and must be explicitly destroyed and rescheduled.

#### Creating a Pod

#### Creating a Pod Manifest

### Running Pods

#### Listing Pods

#### Pod Details

To find out more information about a Pod (or any Kubernetes object), you can use the `kubectl describe` command.
For example, to describe the Pod we previously created, you can run:
```sh
kubectl describe pods kuard
```

#### Deleting a Pod

### Accessing Your Pod

Now that your Pod is running, you're going to want to access it for a variety of reasons.
You may want to load the web service that is running in the Pod.
You may want to view its logs to debug a problem that you are seeing, or even execute other commands in the context of the Pod to help debug.
The following sections detail various ways that you can interact with the code and data running inside your Pod.

#### Getting More Information with Logs

#### Running Commands in Your Container with `exec`

Sometimes logs are insufficient, and to truly determine what's going on, you need to execute commands in the context of the container itself.
To do this, you can use:
```sh
kubectl exec kuard -- date
```
You can also get an interactive session by adding the `-it` flag:
```sh
kubectl exec -it kuard -- ash
```

#### Copying Files to and from Containers

### Health Checks

#### Using Port Forwarding

Later in the book, we'll show how to expose a service to the world or other containers using load balancers - but oftentimes you simply want to access a specific Pod, even if it's not serving traffic on the internet.

To achieve this, you can use the port-forwarding support built into the Kubernetes API and command-line tools.

When you run:
```sh
# Listen on port 8080 locally, forwarding to 8080 in the pod
kubectl port-forward kuard 8080:8080
```

## Chapter 6. Labels and Annotations

Kubernetes was made to grow with you as your application scales in both size and complexity.
Labels and annotations are fundamental concepts in Kubernetes that let you work in sets of things that map to how you think about your application.
You can organize, mark, and cross-index all of your resources to represent the groups that make the most sense for your application.

*Labels* are key/value pairs that can be attached to Kubernetes objects such as Pods and ReplicaSets.
They can be arbitrary and are useful for attaching identifying information to Kubernetes objects.
Labels provide the foundation for grouping objects.

*Annotations*, on the other hand, provide a storage mechanism that resembles labels: key/value pairs designed to hold nonidentifying information that tools and libraries can leverage.
Unlike labels, annotations are not meant for querying, filtering, or otherwise differentiating Pods from each other.

### Labels

## Chapter 7. Service Discovery

While the dynamic nature of Kubernetes makes it easy to run a lot of things, it creates problems when it comes to finding those things.
Most of the traditional network infrastructure wasn't built for the level of dynamism that Kubernetes presents.

### What Is Service Discovery?

The general name for this class of problems and solutions is *service discovery*.
Service-discovery tools help solve the problem of finding which processes are listening at which addresses for which services.
A good service-discovery system will enable users to resolve this information quickly and reliably.
A good system is also low-latency; clients are updated soon after the information associated with a service changes.
Finally, a good service-discovery system can store a richer definition of what that service is.
For example, perhaps there are multiple ports associated with the service.

## Chapter 8. HTTP Load Balancing with Ingress

A critical part of any application is getting network traffic to and from that application.
As described in Chapter 7, Kubernetes has a set of capabilities to enable services to be exposed outside of the cluster.
For many users and simple use cases, these capabilities are sufficient.

But the Service object operates at Layer 4 (according to the OSI model).
This means that it only forwards TCP and UDP connections and doesn't look inside of those connections.
Because of this, hosting many applications on a cluster uses many different exposed services.
In the case where these services are `type: NodePort`, you'll have to have clients connect to a unique port per service.
In the case where these services are `type: LoadBalancer`, you'll be allocating (often expensive or scarce) cloud resources for each service.
But for HTTP (Layer 7)-based services, we can do better.

When solving a similar problem in non-Kubernetes situations, users often turn to the idea of "virtual hosting."
This is a mechanism to host many HTTP sites on a single IP address.
Typically, the user uses a load balancer or reverse proxy to accept incoming connections on HTTP (80) and HTTPS (443) ports.
That program then parses the HTTP connection and, based on the `Host` header and the URL path that is requested, proxies the HTTP call to some other program.
In this way, that load balancer or reverse proxy directs traffic for decoding and directing incoming connections to the right "upstream" server.

Kubernetes calls its HTTP-based load-balancing system *Ingress*.
Ingress is a Kubernetes-native way to implement the "virtual hosting" pattern we just discussed.
One of the more complex aspects of the pattern is that the user has to manage the load balancer configuration file.
In a dynamic environment and as the set of virtual hosts expands, this can be very complex.
The Kubernetes Ingress system works to simplify this by (a) standardizing that configuration, (b) moving it to a standard Kubernetes object, and (c) merging multiple Ingress objects into a single config for the load balancer.

### Ingress Spec Versus Ingress Controller

While conceptually simple, at an implementation level, Ingress is very different from pretty much every other regular resource object in Kubernetes.
Specifically, it is split into a common resource specification and a controller implementation.
There is no "standard" Ingress controller that is built into Kubernetes, so the user must install one of many optional implementations.

### Installing Contour

### Using Ingress

### Advanced Ingress Topics and Gotchas

#### Running Multiple Ingress Controllers

#### Serving TLS

When serving websites, it is becoming increasingly necessary to do so securely using TLS and HTTPS.
Ingress supports this (as do most Ingress controllers).

## Chapter 9. ReplicaSets

Logically, a user managing a replicated set of Pods considers them as a single entity to be defined and managed—and that's precisely what a ReplicaSet is.
A ReplicaSet acts as a cluster-wide Pod manager, ensuring that the right types and numbers of Pods are running at all times.

## Chapter 10. Deployments

So far, you have seen how to package your applications as containers, create replicated sets of containers, and use Ingress controllers to load balance traffic to your services.
You can use all of these objects (Pods, ReplicaSets, and Services) to build a single instance of your application.
However, they do little to help you manage the daily or weekly cadence of releasing new versions of your application.
Indeed, both Pods and ReplicaSets are expected to be tied to specific container images that don't change.

The Deployment object exists to manage the release of new versions.
Deployments represent deployed applications in a way that transcends any particular version.
Additionally, Deployments enable you to easily move from one version of your code to the next.
This "rollout" process is specifiable and careful.
It waits for a user-configurable amount of time between upgrading individual Pods.
It also uses health checks to ensure that the new version of the application is operating correctly and stops the deployment if too many failures occur.

Using Deployments, you can simply and reliably roll out new software versions without downtime or errors.
The actual mechanics of the software rollout performed by a Deployment are controlled by a Deployment controller that runs in the Kubernetes cluster itself.
This means you can let a Deployment proceed unattended and it will still operate correctly and safely.
This makes it easy to integrate Deployments with numerous continuous delivery tools and services.
Further, running server-side makes it safe to perform a rollout from places with poor or intermittent internet connectivity.
Imagine rolling out a new version of your software from your phone while riding on the subway.
Deployments make this possible and safe!

### Your First Deployment

Like all objects in Kubernetes, a Deployment can be represented as a declarative YAML object that provides the details about what you want to run.
In the following case, the Deployment is requesting a single instance of the `kuard` application:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuard
  labels:
    run: kuard
spec:
  selector:
    matchLabels:
      run: kuard
  replicas: 1
  template:
    metadata:
      labels:
        run: kuard
    spec:
      containers:
      - name: kuard
        image: gcr.io/kuar-demo/kuard-amd64:blue
```
Save this YAML file as *kuard-deployment.yaml*, then you can create it using:
```sh
kubectl create -f kuard-deployment.yaml
```
Let's explore how Deployments actually work.
Just as we learned that ReplicaSets manage Pods, Deployments manage ReplicaSets.
As with all relationships in Kubernetes, this relationship is defined by labels and a label selector.
You can see the label selector by looking at the Deployment object:
```sh
kubectl get deployments kuard -o jsonpath --template {.spec.selector.matchLabels}
```
From this you can see that the Deployment is managing a ReplicaSet with the label `run=kuard`.
You can use this in a label selector query across ReplicaSets to find that specific ReplicaSet:
```sh
kubectl get replicasets --selector=run=kuard
```
Now let's look at the relationship between a Deployment and a ReplicaSet in action.
We can resize the Deployment using the imperative `scale` command:
```sh
kubectl scale deployments kuard --replicas=2
```

## Chapter 11. DaemonSets

Deployments and ReplicaSets are generally about creating a service (such as a web server) with multiple replicas for redundancy.
But that is not the only reason to replicate a set of Pods within a cluster.
Another reason is to schedule a single Pod on every node within the cluster.
Generally, the motivation for replicating a Pod to every node is to land some sort of agent or daemon on each node, and the Kubernetes object for achieving this is the DaemonSet.

A DaemonSet ensures that a copy of a Pod is running across a set of nodes in a Kubernetes cluster.
DaemonSets are used to deploy system daemons such as log collectors and monitoring agents, which typically must run on every node.
DaemonSets share similar functionality with ReplicaSets;
both create Pods that are expected to be long-running services and ensure that the desired state and the observed state of the cluster match.

Given the similarities between DaemonSets and ReplicaSets, it's important to understand when to use one over the other.
ReplicaSets should be used when your application is completely decoupled from the node and you can run multiple copies on a given node without special consideration.
DaemonSets should be used when a single copy of your application must run on all or a subset of the nodes in the cluster.

You should generally not use scheduling restrictions or other parameters to ensure that Pods do not colocate on the same node.
If you find yourself wanting a single Pod per node, then a DaemonSet is the correct Kubernetes resource to use.
Likewise, if you find yourself building a homogeneous replicated service to serve user traffic, then a ReplicaSet is probably the right Kubernetes resource to use.

You can use labels to run DaemonSet Pods on specific nodes;
for example, you may want to run specialized intrusion-detection software on nodes that are exposed to the edge network.

You can also use DaemonSets to install software on nodes in a cloud-based cluster.
For many cloud services, an upgrade or scaling of a cluster can delete and/or re-create new virtual machines.
This dynamic immutable infrastructure approach can cause problems if you want (or are required by central IT) to have specific software on every node.
To ensure that specific software is installed on every machine despite upgrades and scale events, a DaemonSet is the right approach.
You can even mount the host filesystem and run scripts that install RPM/DEB packages onto the host operating system.
In this way, you can have a cloud native cluster that still meets the enterprise requirements of your IT department.

### DaemonSet Scheduler

By default, a DaemonSet will create a copy of a Pod on every node unless a node selector is used, which will limit eligible nodes to those with a matching set of labels.
DaemonSets determine which node a Pod will run on at Pod creation time by specifying the `nodeName` field in the Pod spec.
As a result, Pods created by DaemonSets are ignored by the Kubernetes scheduler.

Like ReplicaSets, DaemonSets are managed by a reconciliation control loop that measures the desired state (a Pod is present on all nodes) with the observed state (is the Pod present on a particular node?).
Given this information, the DaemonSet controller creates a Pod on each node that doesn't currently have a matching Pod.

If a new node is added to the cluster, then the DaemonSet controller notices that it is missing a Pod and adds the Pod to the new node.

### Creating DaemonSets

DaemonSets are created by submitting a DaemonSet configuration to the Kubernetes API server.
The DaemonSet in Example 11-1 will create a `fluentd` logging agent on every node in the target cluster.
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
  labels:
    app: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v0.14.10
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

## Chapter 12. Jobs

So far we have focused on long-running processes, such as databases and web applications.
These types of workloads run until they are either upgraded or the service is no longer needed.
While long-running processes make up the large majority of workloads that run on a Kubernetes cluster, there is often a need to run short-lived, one-off tasks.
The Job object is made for handling these types of tasks.

A Job creates Pods that run until successful termination (for instance, exit with 0).
In contrast, a regular Pod will continually restart regardless of its exit code.
Jobs are useful for things you only want to do once, such as database migrations or batch jobs.
If run as a regular Pod, your database migration task would run in a loop, continually repopulating the database after every exit.

In this chapter, we'll explore the most common job patterns Kubernetes affords.
We will also show you how to leverage these patterns in real-life scenarios.

### The Job Object

The Job object is responsible for creating and managing Pods defined in a template in the job specification.
These Pods generally run until successful completion.
The Job object coordinates running a number of Pods in parallel.

If the Pod fails before a successful termination, the job controller will create a new Pod based on the Pod template in the job specification.
Given that Pods have to be scheduled, there is a chance that your job will not execute if the scheduler does not find the required resources.
Also, due to the nature of distributed systems, there is a small chance that duplicate Pods will be created for a specific task during certain failure scenarios.

### Job Patterns

### CronJobs

Sometimes you want to schedule a job to be run at a certain interval.
To achieve this, you can declare a CronJob in Kubernetes, which is responsible for creating a new Job object at a particular interval.

## Chapter 13. ConfigMaps and Secrets

It's good practice to make container images as reusable as possible.
The same image should be able to be used for development, staging, and production.
It's even better if the same image is general-purpose enough to be used across applications and services.
Testing and versioning are more risky and complicated if images need to be re-created for each new environment.
How then do we specialize the use of that image at runtime?

This is where ConfigMaps and Secrets come into play.
ConfigMaps are used to provide configuration information for workloads.
This can be either fine-grained information like a string or a composite value in the form of a file.
Secrets are similar to ConfigMaps but focus on making sensitive information available to the workload.
They can be used for things like credentials or TLS certificates.

### ConfigMaps

One way to think of a ConfigMap is as a Kubernetes object that defines a small filesystem.
Another way is as a set of variables that can be used when defining the environment or command line for your containers.
The key thing to note is that the ConfigMap is combined with the Pod right before it is run.
This means that the container image and the Pod definition can be reused by many workloads just by changing the ConfigMap that is used.

#### Creating ConfigMaps

Let's jump right in and create a ConfigMap.
Like many objects in Kubernetes, you can create these in an immediate, imperative way, or you can create them from a manifest on disk.
We'll start with the imperative method.

#### Using a ConfigMap

### Secrets

While ConfigMaps are great for most configuration data, there is certain data that is extra sensitive.
This includes passwords, security tokens, or other types of private keys.
Collectively, we call this type of data "Secrets."
Kubernetes has native support for storing and handling this data with care.

Secrets enable container images to be created without bundling sensitive data.
This allows containers to remain portable across environments.
Secrets are exposed to Pods via explicit declaration in Pod manifests and the Kubernetes API.
In this way, the Kubernetes Secrets API provides an application-centric mechanism for exposing sensitive configuration information to applications in a way that's easy to audit and leverages native OS isolation primitives.

The remainder of this section will explore how to create and manage Kubernetes Secrets, and also lay out best practices for exposing Secrets to Pods that require them.

#### Creating Secrets

Secrets are created using the Kubernetes API or the `kubectl` command-line tool.
Secrets hold one or more data elements as a collection of key/value pairs.

In this section, we will create a Secret to store a TLS key and certificate for the `kuard` application that meets the storage requirements listed previously.

#### Consuming Secrets

## Chapter 14. Role-Based Access Control for Kubernetes

At this point, nearly every Kubernetes cluster you encounter has role-based access control (RBAC) enabled.
So you have likely encountered RBAC before.
Perhaps you initially couldn't access your cluster until you used some magical command to add a RoleBinding to map a user to a role.
Even though you may have had some exposure to RBAC, you may not have had a great deal of experience understanding RBAC in Kubernetes, including what it is for and how to use it.

Role-based access control provides a mechanism for restricting both access to and actions on Kubernetes APIs to ensure that only authorized users have access.
RBAC is a critical component to both harden access to the Kubernetes cluster where you are deploying your application and (possibly more importantly) prevent unexpected accidents where one person in the wrong namespace mistakenly takes down production when they think they are destroying their test cluster.

Before we dive into the details of RBAC in Kubernetes, it's valuable to have a high-level understanding of RBAC as a concept, as well as authentication and authorization more generally.

Every request to Kubernetes is first authenticated.
Authentication provides the identity of the caller issuing the request.
It could be as simple as saying that the request is unauthenticated, or it could integrate deeply with a pluggable authentication provider (e.g., Azure Active Directory) to establish an identity within that third-party system.
Interestingly enough, Kubernetes does not have a built-in identity store, focusing instead on integrating other identity sources within itself.

Once users have been authenticated, the authorization phase determines whether they are authorized to perform the request.
Authorization is a combination of the identity of the user, the resource (effectively the HTTP path), and the verb or action the user is attempting to perform.
If the particular user is authorized to perform that action on that resource, then the request is allowed to proceed.
Otherwise, an HTTP 403 error is returned.
Let's dive into this process.

### Role-Based Access Control



## Chapter 15. Service Meshes


