# Kubernetes: Up and Running, 2nd Edition

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

A Pod represents a collection of application containers and volumes running in the same execution environment.
Pods, not containers, are the smallest deployable artifact in a Kubernetes cluster.
This means all of the containers in a Pod always land on the same machine.

### Thinking with Pods

In general, the right question to ask yourself when designing Pods is, "Will these containers work correctly if they land on different machines?"
If the answer is "no," a Pod is the correct grouping for the containers.
If the answer is "yes," multiple Pods is probably the correct solution.
In the example at the beginning of this chapter, the two containers interact via a local filesystem.
It would be impossible for them to operate correctly if the containers were scheduled on different machines.

### The Pod Manifest

Pods are described in a Pod manifest.
The Pod manifest is just a text-file representation of the Kubernetes API object.
Kubernetes strongly believes in *declarative configuration*.
Declarative configuration means that you write down the desired state of the world in a configuration and then submit that configuration to a service that takes actions to ensure the desired state becomes the actual state.

The Kubernetes API server accepts and processes Pod manifests before storing them in persistent storage (`etcd`).
The scheduler also uses the Kubernetes API to find Pods that haven't been scheduled to a node.
The scheduler then places the Pods onto nodes depending on the resources and other constraints expressed in the Pod manifests.
Multiple Pods can be placed on the same machine as long as there are sufficient resources.

#### Creating a Pod

#### Creating a Pod Manifest

### Accessing Your Pod

Now that your Pod is running, you're going to want to access it for a variety of reasons.
You may want to load the web service that is running in the Pod. You may want to view its logs to debug a problem that you are seeing, or even execute other commands in the context of the Pod to help debug.
The following sections detail various ways that you can interact with the code and data running inside your Pod.

#### Using Port Forwarding

Later in the book, we'll show how to expose a service to the world or other containers using load balancers - but oftentimes you simply want to access a specific Pod, even if it's not serving traffic on the internet.

To achieve this, you can use the port-forwarding support built into the Kubernetes API and command-line tools.

When you run:
```
# Listen on port 8080 locally, forwarding to 8080 in the pod
kubectl port-forward kuard 8080:8080
```
