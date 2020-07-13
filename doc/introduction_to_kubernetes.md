# Introduction to Kubernetes

## Chapter 1 From Monolithic to Microservices

### The Legacy Monolith

A **monolith** has a rather expensive taste in hardware.
Being a large, single piece of software which continuously grows, it has to run on a single system which has to satisfy its compute, memory, storage, and networking requirements.
The hardware of such capacity is both complex and pricey.

Since the entire monolith application runs as a single process, the scaling of individual features of the monolith is almost impossible.
It internally supports a hardcoded number of connections and operations.
However, scaling the entire application means to manually deploy a new instance of the monolith on another server, typically behind a load balancing appliance - another pricey solution.

During upgrades, patches or migrations of the monolith application - downtimes occur and maintenance windows have to be planned as disruptions in service are expected to impact clients.
While there are solutions to minimize downtimes to customers by setting up monolith applications in a highly available active/passive configuration, it may still be challenging for system engineers to keep all systems at the same patch level.

### The Modern Microservice

**Microservices** can be deployed individually on separate servers provisioned with fewer resources - only what is required by each service and the host system itself.

Microservices-based architecture is aligned with Event-driven Architecture and Service-Oriented Architecture (SOA) principles, where complex applications are composed of small independent processes which communicate with each other through APIs over a network.
APIs allow access by other internal services of the same application or external, third-party services and applications.

Each microservice is developed and written in a modern programming language, selected to be the best suitable for the type of service and its business function.
This offers a great deal of flexibility when matching microservices with specific hardware when required, allowing deployments on inexpensive commodity hardware.

Although the distributed nature of microservices adds complexity to the architecture, one of the greatest benefits of microservices is scalability.
With the overall application becoming modular, each microservice can be scaled individually, either manually or automated through demand-based autoscaling.

Seamless upgrades and patching processes are other benefits of microservices architecture.
There is virtually no downtime and no service disruption to clients because upgrades are rolled out seamlessly - one service at a time, rather than having to re-compile, re-build and re-start an entire monolithic application.
As a result, businesses are able to develop and roll-out new features and updates a lot faster, in an agile approach, having separate teams focusing on separate features, thus being more productive and cost-effective.

### Refactoring

### Challenges

The refactoring path from a monolith to microservices is not smooth and without challenges.
Not all monoliths are perfect candidates for refactoring, while some may not even "survive" such a modernization phase.
When deciding whether a monolith is a possible candidate for refactoring, there are many possible issues to consider.

When considering a legacy Mainframe based system, written in older programming languages - Cobol or Assembler, it may be more economical to just re-build it from the ground up as a cloud-native application.
A poorly designed legacy application should be re-designed and re-built from scratch following modern architectural patterns for microservices and even containers.
Applications tightly coupled with data stores are also poor candidates for refactoring.

Once the monolith survived the refactoring phase, the next challenge is to design mechanisms or find suitable tools to keep alive all the decoupled modules to ensure application resiliency as a whole. 

Choosing runtimes may be another challenge.
If deploying many modules on a single physical or virtual server, chances are that different libraries and runtime environment may conflict with one another causing errors and failures.
This forces deployments of single modules per servers in order to separate their dependencies - not an economical way of resource management, and no real segregation of libraries and runtimes, as each server also has an underlying Operating System running with its libraries, thus consuming server resources - at times the OS consuming more resources than the application module itself.

Ultimately application containers came along, providing encapsulated lightweight runtime environments for application modules.
Containers promised consistent software environments for developers, testers, all the way from Development to Production.
Wide support of containers ensured application portability from physical bare-metal to Virtual Machines, but this time with multiple applications deployed on the very same server, each running in their own execution environments isolated from one another, thus avoiding conflicts, errors, and failures.
Other features of containerized application environments are higher server utilization, individual module scalability, flexibility, interoperability and easy integration with automation tools.

### Success Stories

## Chapter 2 Container Orchestration

### What Are Containers?

Before we dive into container orchestration, let's review first what containers are.

Containers are application-centric methods to deliver high-performing, scalable applications on any infrastructure of your choice.
Containers are best suited to deliver microservices by providing portable, isolated virtual environments for applications to run without interference from other running applications.

**Microservices** are lightweight applications written in various modern programming languages, with specific dependencies, libraries and environmental requirements.
To ensure that an application has everything it needs to run successfully it is packaged together with its dependencies.

Containers encapsulate microservices and their dependencies but do not run them directly.
Containers run container images.

A **container image** bundles the application along with its runtime and dependencies, and a container is deployed from the container image offering an isolated executable environment for the application.
Containers can be deployed from a specific image on many platforms, such as workstations, Virtual Machines, public cloud, etc.

### What Is Container Orchestration?

In Development (Dev) environments, running containers on a single host for development and testing of applications may be an option.
However, when migrating to Quality Assurance (QA) and Production (Prod) environments, that is no longer a viable option because the applications and services need to meet specific requirements:
* Fault-tolerance
* On-demand scalability
* Optimal resource usage
* Auto-discovery to automatically discover and communicate with each other
* Accessibility from the outside world
* Seamless updates/rollbacks without any downtime.

**Container orchestrators** are tools which group systems together to form clusters where containers' deployment and management is automated at scale while meeting the requirements mentioned above.

### Container Orchestrators

With enterprises containerizing their applications and moving them to the cloud, there is a growing demand for container orchestration solutions.
While there are many solutions available, some are mere re-distributions of well-established container orchestration tools, enriched with features and, sometimes, with certain limitations in flexibility.

Although not exhaustive, the list below provides a few different container orchestration tools and services available today:
* Amazon Elastic Container Service
Amazon Elastic Container Service (ECS) is a hosted service provided by Amazon Web Services (AWS) to run Docker containers at scale on its infrastructure.
* Azure Container Instances
Azure Container Instance (ACI) is a basic container orchestration service provided by Microsoft Azure.
* Azure Service Fabric
Azure Service Fabric is an open source container orchestrator provided by Microsoft Azure.
* Kubernetes
Kubernetes is an open source orchestration tool, started by Google, part of the Cloud Native Computing Foundation (CNCF) project.
* Marathon
Marathon is a framework to run containers at scale on Apache Mesos.
* Nomad
Nomad is the container orchestrator provided by HashiCorp.
* Docker Swarm
Docker Swarm is a container orchestrator provided by Docker, Inc.
It is part of Docker Engine.

### Why Use Container Orchestrators?

Although we can manually maintain a couple of containers or write scripts for dozens of containers, orchestrators make things much easier for operators especially when it comes to managing hundreds and thousands of containers running on a global infrastructure.

Most container orchestrators can:
* Group hosts together while creating a cluster
* Schedule containers to run on hosts in the cluster based on resources availability
* Enable containers in a cluster to communicate with each other regardless of the host they are deployed to in the cluster
* Bind containers and storage resources
* Group sets of similar containers and bind them to load-balancing constructs to simplify access to containerized applications by creating a level of abstraction between the containers and the user
* Manage and optimize resource usage
* Allow for implementation of policies to secure access to applications running inside containers.

With all these configurable yet flexible features, container orchestrators are an obvious choice when it comes to managing containerized applications at scale.
In this course, we will explore **Kubernetes**, one of the most in-demand container orchestration tools available today.

### Where to Deploy Container Orchestrators?

Most container orchestrators can be deployed on the infrastructure of our choice - on bare metal, Virtual Machines, on-premise, or the public cloud. Kubernetes, for example, can be deployed on a workstation, with or without a local hypervisor such as Oracle VirtualBox, inside a company's data center, in the cloud on AWS Elastic Compute Cloud (EC2) instances, Google Compute Engine (GCE) VMs, DigitalOcean Droplets, OpenStack, etc.

There are turnkey solutions which allow Kubernetes clusters to be installed, with only a few commands, on top of cloud Infrastructures-as-a-Service, such as GCE, AWS EC2, Docker Enterprise, IBM Cloud, Rancher, VMware, Pivotal, and multi-cloud solutions through IBM Cloud Private and StackPointCloud.

Last but not least, there is the managed container orchestration as-a-Service, more specifically the managed Kubernetes as-a-Service solution, offered and hosted by the major cloud providers, such as Google Kubernetes Engine (GKE), Amazon Elastic Container Service for Kubernetes (Amazon EKS), Azure Kubernetes Service (AKS), IBM Cloud Kubernetes Service, DigitalOcean Kubernetes, Oracle Container Engine for Kubernetes, etc.
These shall be explored in one of the later chapters.

## Chapter 3 Kubernetes

### What Is Kubernetes?

According to the Kubernetes website,
> "Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications."

**Kubernetes** comes from the Greek word **κυβερνήτης**, which means helmsman or ship pilot.
With this analogy in mind, we can think of Kubernetes as the pilot on a ship of containers.

Kubernetes is also referred to as **k8s**, as there are 8 characters between k and s.

Kubernetes is highly inspired by the Google Borg system, a container orchestrator for its global operations for more than a decade.
It is an open source project written in the Go language and licensed under the Apache License, Version 2.0.

Kubernetes was started by Google and, with its v1.0 release in July 2015, Google donated it to the Cloud Native Computing Foundation (CNCF).
We will talk more about CNCF later in this chapter.

New Kubernetes versions are released in 3 months cycles.
The current stable version is 1.14 (as of May 2019).

### From Borg to Kubernetes

### Kubernetes Features I

Kubernetes offers a very rich set of features for container orchestration.
Some of its fully supported features are:
* Automatic bin packing
Kubernetes automatically schedules containers based on resource needs and constraints, to maximize utilization without sacrificing availability.
* Self-healing
Kubernetes automatically replaces and reschedules containers from failed nodes.
It kills and restarts containers unresponsive to health checks, based on existing rules/policy. It also prevents traffic from being routed to unresponsive containers.
* Horizontal scaling
With Kubernetes applications are scaled manually or automatically based on CPU or custom metrics utilization.
* Service discovery and Load balancing
Containers receive their own IP addresses from Kubernetes, while it assigns a single Domain Name System (DNS) name to a set of containers to aid in load-balancing requests across the containers of the set.

### Kubernetes Features II

Some other fully supported Kubernetes features are:
* Automated rollouts and rollbacks
Kubernetes seamlessly rolls out and rolls back application updates and configuration changes, constantly monitoring the application's health to prevent any downtime.
* Secret and configuration management
Kubernetes manages secrets and configuration details for an application separately from the container image, in order to avoid a re-build of the respective image.
Secrets consist of confidential information passed to the application without revealing the sensitive content to the stack configuration, like on GitHub.
* Storage orchestration
Kubernetes automatically mounts software-defined storage (SDS) solutions to containers from local storage, external cloud providers, or network storage systems.
* Batch execution
Kubernetes supports batch execution, long-running jobs, and replaces failed containers.

There are many other features besides the ones we just mentioned, and they are currently in alpha/beta phase.
They will add great value to any Kubernetes deployment once they become stable features.
For example, support for role-based access control (RBAC) is stable as of the Kubernetes 1.8 release.

### Why Use Kubernetes?

In addition to its fully-supported features, Kubernetes is also portable and extensible.
It can be deployed in many environments such as local or remote Virtual Machines, bare metal, or in public/private/hybrid/multi-cloud setups.
It supports and it is supported by many 3rd party open source tools which enhance Kubernetes' capabilities and provide a feature-rich experience to its users.

Kubernetes' architecture is modular and pluggable.
Not only that it orchestrates modular, decoupled microservices type applications, but also its architecture follows decoupled microservices patterns.
Kubernetes' functionality can be extended by writing custom resources, operators, custom APIs, scheduling rules or plugins.

For a successful open source project, the community is as important as having great code.
Kubernetes is supported by a thriving community across the world. It has more than 2,000 contributors, who, over time, have pushed over 77,000 commits.
There are meet-up groups in different cities and countries which meet regularly to discuss Kubernetes and its ecosystem.
There are Special Interest Groups (SIGs), which focus on special topics, such as scaling, bare metal, networking, etc.
We will talk more about them in our last chapter, Kubernetes Communities.

### Cloud Native Computing Foundation (CNCF)

### CNCF and Kubernetes

## Chapter 4 Kubernetes Architecture

### Kubernetes Architecture

At a very high level, Kubernetes has the following main components:
* One or more **master nodes**
* One or more **worker nodes**
* Distributed key-value store, such as **etcd**

### Master Node

The master node provides a running environment for the control plane responsible for managing the state of a Kubernetes cluster, and it is the brain behind all operations inside the cluster.
The control plane components are agents with very distinct roles in the cluster's management.
In order to communicate with the Kubernetes cluster, users send requests to the master node via a Command Line Interface (CLI) tool, a Web User-Interface (Web UI) Dashboard, or Application Programming Interface (API).

It is important to keep the control plane running at all costs.
Losing the control plane may introduce downtimes, causing service disruption to clients, with possible loss of business.
To ensure the control plane's fault tolerance, master node replicas are added to the cluster, configured in High-Availability (HA) mode.
While only one of the master node replicas actively manages the cluster, the control plane components stay in sync across the master node replicas.
This type of configuration adds resiliency to the cluster's control plane, should the active master node replica fail.

To persist the Kubernetes cluster's state, all cluster configuration data is saved to etcd.
However, etcd is a distributed key-value store which only holds cluster state related data, no client workload data.
etcd is configured on the master node (stacked) or on its dedicated host (external) to reduce the chances of data store loss by decoupling it from the control plane agents.

When stacked, HA master node replicas ensure etcd resiliency as well.
Unfortunately, that is not the case of external etcds, when the etcd hosts have to be separately replicated for HA mode configuration.

### Master Node Components

A master node has the following components:
* API Server
* Scheduler
* Controller Managers
* etcd

#### API Server

All the administrative tasks are coordinated by the **kube-apiserver**, a central control plane component running on the master node.
The API server intercepts RESTful calls from users, operators and external agents, then validates and processes them.
During processing the API server reads the Kubernetes cluster's current state from the etcd, and after a call's execution, the resulting state of the Kubernetes cluster is saved in the distributed key-value data store for persistence.
The API server is the only master plane component to talk to the etcd data store, both to read and to save Kubernetes cluster state information from/to it - acting as a middle-man interface for any other control plane agent requiring to access the cluster's data store.

The API server is highly configurable and customizable.
It also supports the addition of custom API servers, when the primary API server becomes a proxy to all secondary custom API servers and routes all incoming RESTful calls to them based on custom defined rules.

#### Scheduler

The role of the **kube-scheduler** is to assign new objects, such as pods, to nodes.
During the scheduling process, decisions are made based on current Kubernetes cluster state and new object's requirements.
The scheduler obtains from etcd, via the API server, resource usage data for each worker node in the cluster.
The scheduler also receives from the API server the new object's requirements which are part of its configuration data.
Requirements may include constraints that users and operators set, such as scheduling work on a node labeled with **disk==ssd** key/value pair.
The scheduler also takes into account Quality of Service (QoS) requirements, data locality, affinity, anti-affinity, taints, toleration, etc.

The scheduler is highly configurable and customizable.
Additional custom schedulers are supported, then the object's configuration data should include the name of the custom scheduler expected to make the scheduling decision for that particular object; if no such data is included, the default scheduler is selected instead.

A scheduler is extremely important and quite complex in a multi-node Kubernetes cluster.
In a single-node Kubernetes cluster, such as the one explored later in this course, the scheduler's job is quite simple.

#### Controller Managers

The **controller managers** are control plane components on the master node running controllers to regulate the state of the Kubernetes cluster.
Controllers are watch-loops continuously running and comparing the cluster's desired state (provided by objects' configuration data) with its current state (obtained from etcd data store via the API server).
In case of a mismatch corrective action is taken in the cluster until its current state matches the desired state.

The **kube-controller-manager** runs controllers responsible to act when nodes become unavailable, to ensure pod counts are as expected, to create endpoints, service accounts, and API access tokens.

The **cloud-controller-manager** runs controllers responsible to interact with the underlying infrastructure of a cloud provider when nodes become unavailable, to manage storage volumes when provided by a cloud service, and to manage load balancing and routing.

#### etcd

**etcd** is a distributed key-value data store used to persist a Kubernetes cluster's state.
New data is written to the data store only by appending to it, data is never replaced in the data store.
Obsolete data is compacted periodically to minimize the size of the data store.

Out of all the control plane components, only the API server is able to communicate with the etcd data store.

etcd's CLI management tool provides backup, snapshot, and restore capabilities which come in handy especially for a single etcd instance Kubernetes cluster - common in Development and learning environments.
However, in Stage and Production environments, it is extremely important to replicate the data stores in HA mode, for cluster configuration data resiliency.

Some Kubernetes cluster bootstrapping tools, by default, provision stacked etcd master nodes, where the data store runs alongside and shares resources with the other control plane components on the same master node.
For data store isolation from the control plane components, the bootstrapping process can be configured for an external etcd, where the data store is provisioned on a dedicated separate host, thus reducing the chances of an etcd failure.
Both stacked and external etcd configurations support HA configurations.
etcd is based on the Raft Consensus Algorithm which allows a collection of machines to work as a coherent group that can survive the failures of some of its members.
At any given time, one of the nodes in the group will be the master, and the rest of them will be the followers.
Any node can be treated as a master.

etcd is written in the Go programming language.
In Kubernetes, besides storing the cluster state, etcd is also used to store configuration details such as subnets, ConfigMaps, Secrets, etc.

### Worker Node

A **worker node** provides a running environment for client applications.
Though containerized microservices, these applications are encapsulated in Pods, controlled by the cluster control plane agents running on the master node.
Pods are scheduled on worker nodes, where they find required compute, memory and storage resources to run, and networking to talk to each other and the outside world.
A Pod is the smallest scheduling unit in Kubernetes.
It is a logical collection of one or more containers scheduled together.
We will explore them further in later chapters.

Also, to access the applications from the external world, we connect to worker nodes and not to the master node.
We will dive deeper into this in future chapters.

### Worker Node Components

A worker node has the following components:
* Container Runtime
* kubelet
* kube-proxy
* Addons for DNS, Dashboard, cluster-level monitoring and logging

#### Container Runtime

Although Kubernetes is described as a "container orchestration engine", it does not have the capability to directly handle containers.
In order to run and manage a container's lifecycle, Kubernetes requires a **container runtime** on the node where a Pod and its containers are to be scheduled.
Kubernetes supports many container runtimes:

* Docker - although a container platform which uses containerd as a container runtime, it is the most widely used container runtime with Kubernetes
* CRI-O - a lightweight container runtime for Kubernetes, it also supports Docker image registries
* containerd - a simple and portable container runtime providing robustness
* rkt - a pod-native container engine, it also runs Docker images
* rktlet - a Kubernetes Container Runtime Interface (CRI) implementation using rkt.

#### kubelet

The kubelet is an agent running on each node and communicates with the control plane components from the master node.
It receives Pod definitions, primarily from the API server, and interacts with the container runtime on the node to run containers associated with the Pod.
It also monitors the health of the Pod's running containers.

The kubelet connects to the container runtime using Container Runtime Interface (CRI).
CRI consists of protocol buffers, gRPC API, and libraries.

As shown above, the kubelet acting as grpc client connects to the CRI shim acting as grpc server to perform container and image operations.
CRI implements two services: **ImageService** and **RuntimeService**.
The **ImageService** is responsible for all the image-related operations, while the **RuntimeService** is responsible for all the Pod and container-related operations.

Container runtimes used to be hard-coded in Kubernetes, but with the development of CRI, Kubernetes is more flexible now and uses different container runtimes without the need to recompile.
Any container runtime that implements CRI can be used by Kubernetes to manage Pods, containers, and container images.

#### kubelet - CRI shims

Below you will find some examples of CRI shims:
* dockershim
With dockershim, containers are created using Docker installed on the worker nodes.
Internally, Docker uses containerd to create and manage containers.

#### kube-proxy

The **kube-proxy** is the network agent which runs on each node responsible for dynamic updates and maintenance of all networking rules on the node.
It abstracts the details of Pods networking and forwards connection requests to Pods.

#### Addons

**Addons** are cluster features and functionality not yet available in Kubernetes, therefore implemented through 3rd-party pods and services.
* DNS - cluster DNS is a DNS server required to assign DNS records to Kubernetes objects and resources
* Dashboard - a general purposed web-based user interface for cluster management
* Monitoring - collects cluster-level container metrics and saves them to a central data store
* Logging - collects cluster-level container logs and saves them to a central log store for analysis.

### Networking Challenges

Decoupled microservices based applications rely heavily on networking in order to mimic the tight-coupling once available in the monolithic era.
Networking, in general, is not the easiest to understand and implement.
Kubernetes is no exception - as a containerized microservices orchestrator is needs to address 4 distinct networking challenges:
* Container-to-container communication inside Pods
* Pod-to-Pod communication on the same node and across cluster nodes
* Pod-to-Service communication within the same namespace and across cluster namespaces
* External-to-Service communication for clients to access applications in a cluster.

All these networking challenges must be addressed before deploying a Kubernetes cluster.
Next, we will see how we solve these challenges.

#### Container-to-Container Communication Inside Pods

Making use of the underlying host operating system's kernel features, a container runtime creates an isolated network space for each container it starts.
On Linux, that isolated network space is referred to as a **network namespace**.
A network namespace is shared across containers, or with the host operating system.

When a Pod is started, a network namespace is created inside the Pod, and all containers running inside the Pod will share that network namespace so that they can talk to each other via localhost.

#### Pod-to-Pod Communication Across Nodes

In a Kubernetes cluster Pods are scheduled on nodes randomly.
Regardless of their host node, Pods are expected to be able to communicate with all other Pods in the cluster, all this without the implementation of Network Address Translation (NAT).
This is a fundamental requirement of any networking implementation in Kubernetes.

The Kubernetes network model aims to reduce complexity, and it treats Pods as VMs on a network, where each VM receives an IP address - thus each Pod receiving an IP address.
This model is called "IP-per-Pod" and ensures Pod-to-Pod communication, just as VMs are able to communicate with each other.

Let's not forget about containers though.
They share the Pod's network namespace and must coordinate ports assignment inside the Pod just as applications would on a VM, all while being able to communicate with each other on localhost - inside the Pod.
However, containers are integrated with the overall Kubernetes networking model through the use of the Container Network Interface (CNI) supported by CNI plugins.
CNI is a set of a specification and libraries which allow plugins to configure the networking for containers.
While there are a few core plugins, most CNI plugins are 3rd-party Software Defined Networking (SDN) solutions implementing the Kubernetes networking model.
In addition to addressing the fundamental requirement of the networking model, some networking solutions offer support for Network Policies.
Flannel, Weave, Calico are only a few of the SDN solutions available for Kubernetes clusters.

The container runtime offloads the IP assignment to CNI, which connects to the underlying configured plugin, such as Bridge or MACvlan, to get the IP address.
Once the IP address is given by the respective plugin, CNI forwards it back to the requested container runtime.

#### Pod-to-External World Communication

For a successfully deployed containerized applications running in Pods inside a Kubernetes cluster, it requires accessibility from the outside world.
Kubernetes enables external accessibility through **services**, complex constructs which encapsulate networking rules definitions on cluster nodes.
By exposing services to the external world with **kube-proxy**, applications become accessible from outside the cluster over a virtual IP.

We will have a complete chapter dedicated to this, so we will dive into this later.

## Chapter 5 Installing Kubernetes

### Kubernetes Configuration

Kubernetes can be installed using different configurations. The four major installation types are briefly presented below:

* All-in-One Single-Node Installation
In this setup, all the master and worker components are installed and running on a single-node.
While it is useful for learning, development, and testing, it should not be used in production.
Minikube is one such example, and we are going to explore it in future chapters.
* Single-Node etcd, Single-Master and Multi-Worker Installation
In this setup, we have a single-master node, which also runs a single-node etcd instance.
Multiple worker nodes are connected to the master node.
* Single-Node etcd, Multi-Master and Multi-Worker Installation
In this setup, we have multiple-master nodes configured in HA mode, but we have a single-node etcd instance.
Multiple worker nodes are connected to the master nodes.
* Multi-Node etcd, Multi-Master and Multi-Worker Installation
In this mode, etcd is configured in clustered HA mode, the master nodes are all configured in HA mode, connecting to multiple worker nodes.
This is the most advanced and recommended production setup.

### Infrastructure for Kubernetes Installation

Once we decide on the installation type, we also need to make some infrastructure-related decisions, such as:

* Should we set up Kubernetes on bare metal, public cloud, or private cloud?
* Which underlying OS should we use? Should we choose RHEL, CoreOS, CentOS, or something else?
* Which networking solution should we use?
* And so on

Explore the [Kubernetes documentation](https://kubernetes.io/docs/setup/) for details on choosing the right solution.

### Localhost Installation

These are only a few localhost installation options available to deploy single- or multi-node Kubernetes clusters on our workstation/laptop:

* Minikube - single-node local Kubernetes cluster
* Docker Desktop - single-node local Kubernetes cluster for Windows and Mac
* CDK on LXD - multi-node local cluster with LXD containers.

Minikube is the preferred and recommended way to create an all-in-one Kubernetes setup locally.
We will be using it extensively in this course.

## Chapter 6 Minikube - A Local Single-Node Kubernetes Cluster

### Introduction

As we mentioned in the previous chapter, Minikube is the easiest and most recommended way to run an all-in-one Kubernetes cluster locally on our workstations.
In this chapter, we will explore the requirements to install Minikube locally on our workstation, together with the installation instructions to set it up on local Linux, macOS, and Windows operating systems.

### Requirements for Running Minikube

Minikube is installed and runs directly on a local Linux, macOS, or Windows workstation.
However, in order to fully take advantage of all the features Minikube has to offer, a Type-2 Hypervisor should be installed on the local workstation, to run in conjunction with Minikube.
This does not mean that we need to create any VMs with guest operating systems with this Hypervisor.

Minikube builds all its infrastructure as long as the Type-2 Hypervisor is installed on our workstation.
Minikube invokes the Hypervisor to create a single VM which then hosts a single-node Kubernetes cluster.
Thus we need to make sure that we have the necessary hardware and software required by Minikube to build its environment.
Below we outline the requirements to run Minikube on our local workstation:

* kubectl
**kubectl** is a binary used to access and manage any Kubernetes cluster.
It is installed separately from Minikube.
Since we will install kubectl after the Minikube installation, we may see warnings during the Minikube initialization - safe to disregard for the time being, but do keep in mind that we will have to install kubectl to be able to manage the Kubernetes cluster.
We will explore kubectl in more detail in future chapters.
* Type-2 Hypervisor
  * On Linux VirtualBox or KVM
  * On macOS VirtualBox, HyperKit, or VMware Fusion
  * On Windows VirtualBox or Hyper-V

NOTE: Minikube supports a --vm-driver=none option that runs the Kubernetes components directly on the host OS and not inside a VM. With this option a Docker installation is required and a Linux OS on the local workstation, but no hypervisor installation. If you use --vm-driver=none, be sure to specify a bridge network for Docker. Otherwise, it might change between network restarts, causing loss of connectivity to your cluster.
* VT-x/AMD-v virtualization must be enabled on the local workstation in BIOS
* Internet connection on first Minikube run - to download packages, dependencies, updates and pull images needed to initialize the Minikube Kubernetes cluster. Subsequent runs will require an internet connection only when new Docker images need to be pulled from a container repository or when deployed containerized applications need it. Once an image has been pulled it can be reused without an internet connection.

In this chapter, we use VirtualBox as hypervisor on all three operating systems - Linux, macOS, and Windows, to allow Minikube to provision the VM which hosts the single-node Kubernetes cluster.

### Installing Minikube on Linux

### Installing Minikube on macOS

#### Install the VirtualBox hypervisor for OS X hosts

Download and install the **.dmg** package.

#### Install Minikube

```bash
brew install minikube
```

#### Start Minikube

```bash
minikube start --driver=virtualbox
```

#### Check the status

```bash
minikube status
```

#### Stop Minikube

```bash
minikube stop
```

### Installing Minikube on Windows

### Minikube CRI-O

## Chapter 7 Accessing Minikube

### Accessing Minikube

Any healthy running Kubernetes cluster can be accessed via any one of the following methods:

* Command Line Interface (CLI) tools and scripts
* Web-based User Interface (Web UI) from a web browser
* APIs from CLI or programmatically

These methods are applicable to all Kubernetes clusters.

#### CLI

`kubectl` is the **Kubernetes Command Line Interface (CLI) client** to manage cluster resources and applications.
It can be used standalone, or part of scripts and automation tools.
Once all required credentials and cluster access points have been configured for `kubectl` it can be used remotely from anywhere to access a cluster. 

In later chapters, we will be using kubectl to deploy applications, manage and configure Kubernetes resources.

#### Web UI

#### APIs

As we know, Kubernetes has the API server, and operators/users connect to it from the external world to interact with the cluster.
Using both CLI and Web UI, we can connect to the API server running on the master node to perform different operations.
We can directly connect to the API server using its API endpoints and send commands to it, as long as we can access the master node and have the right credentials.

HTTP API space of Kubernetes can be divided into three independent groups:

* Core Group (/api/v1)
This group includes objects such as Pods, Services, nodes, namespaces, configmaps, secrets, etc.
* Named Group
This group includes objects in /apis/$NAME/$VERSION format. These different API versions imply different levels of stability and support:
Alpha level - it may be dropped at any point in time, without notice. For example, /apis/batch/v2alpha1.
Beta level - it is well-tested, but the semantics of objects may change in incompatible ways in a subsequent beta or stable release. For example, /apis/certificates.k8s.io/v1beta1.
Stable level - appears in released software for many subsequent versions. For example, /apis/networking.k8s.io/v1.
* System-wide
This group consists of system-wide API endpoints, like /healthz, /logs, /metrics, /ui, etc.
We can either connect to an API server directly via calling the respective API endpoints or via the CLI/Web UI.

Next, we will see how we can access the Minikube environment we set up in the previous chapter.

#### kubectl

##### Installing kubectl on Linux

##### Installing kubectl on macOS

```bash
brew install kubernetes-cli
```

##### Installing kubectl on Windows

##### kubectl Configuration File

To access the Kubernetes cluster, the kubectl client needs the master node endpoint and appropriate credentials to be able to interact with the API server running on the master node.
While starting Minikube, the startup process creates, by default, a configuration file, config, inside the `.kube` directory (often referred to as the `dot-kube-config` file), which resides in the user's home directory.
The configuration file has all the connection details required by `kubectl`.
By default, the `kubectl` binary parses this file to find the master node's connection endpoint, along with credentials.
To look at the connection details, we can either see the content of the `~/.kube/config` file (on Linux) or run the following command:
```bash
kubectl config view
```

Once kubectl is installed, we can get information about the Minikube cluster with the `kubectl cluster-info` command:
```bash
kubectl cluster-info
```

You can find more details about the kubectl command line options here.

Although for the Kubernetes cluster installed by Minikube the `~/.kube/config` file gets created automatically, this is not the case for Kubernetes clusters installed by other tools.
In other cases, the config file has to be created manually and sometimes re-configured to suit various networking and client/server setups.

#### Kubernetes Dashboard

As mentioned earlier, the Kubernetes Dashboard provides a web-based user interface for Kubernetes cluster management.
To access the dashboard from Minikube, we can use the `minikube dashboard` command, which opens a new tab on our web browser, displaying the Kubernetes Dashboard:
```bash
minikube dashboard
```

#### The `kubectl proxy` Command

Issuing the `kubectl proxy` command, `kubectl` authenticates with the API server on the master node and makes the Dashboard available on a slightly different URL than the one earlier, this time through the proxy port 8001.

First, we issue the `kubectl proxy` command:
```bash
kubectl proxy
```

It locks the terminal for as long as the proxy is running.
With the proxy running we can access the Dashboard over the new URL (just click on it below - it should work on your workstation).
Once we stop the proxy (with CTRL + C) the Dashboard is no longer accessible.

[http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/](http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/)

#### APIs - with `kubectl proxy`

When `kubectl proxy` is running, we can send requests to the API over the localhost on the proxy port 8001 (from another terminal, since the proxy locks the first terminal):
```bash
curl http://localhost:8001/
```

With the above curl request, we requested all the API endpoints from the API server.
Clicking on the link above (in the curl command), it will open the same listing output in a browser tab.

We can explore every single path combination with curl or in a browser, such as:
* http://localhost:8001/api/v1
* http://localhost:8001/apis/apps/v1
* http://localhost:8001/healthz
* http://localhost:8001/metrics

#### APIs - without `kubectl proxy`

When not using the `kubectl proxy`, we need to authenticate to the API server when sending API requests.
We can authenticate by providing a **Bearer Token** when issuing a curl, or by providing a set of **keys** and **certificates**.

A **Bearer Token** is an **access token** which is generated by the authentication server (the API server on the master node) and given back to the client.
Using that token, the client can connect back to the Kubernetes API server without providing further authentication details, and then, access resources.

Get the token:
```bash
TOKEN=$(kubectl describe secret -n kube-system $(kubectl get secrets -n kube-system | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t' | tr -d " ")
```

Get the API server endpoint:
```bash
APISERVER=$(kubectl config view | grep https | cut -f 2- -d ":" | tr -d " ")
```

Confirm that the APISERVER stored the same IP as the Kubernetes master IP by issuing the following 2 commands and comparing their outputs:
```bash
echo $APISERVER

kubectl cluster-info
```

Access the API server using the `curl` command, as shown below:
```bash
curl $APISERVER --header "Authorization: Bearer $TOKEN" --insecure
```

Instead of the access token, we can extract the client certificate, client key, and certificate authority data from the `.kube/config` file.
Once extracted, they are encoded and then passed with a curl command for authentication.
The new `curl` command looks similar to:
```bash
curl $APISERVER --cert encoded-cert --key encoded-key --cacert encoded-ca
```

## Chapter 8 Kubernetes Building Blocks

### Pods

A [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) is the smallest and simplest Kubernetes object.
It is the unit of deployment in Kubernetes, which represents a single instance of the application.
A Pod is a logical collection of one or more containers, which:
* Are scheduled together on the same host with the Pod
* Share the same network namespace
* Have access to mount the same external storage (volumes)

Pods are ephemeral in nature, and they do not have the capability to self-heal by themselves.
That is the reason they are used with controllers which handle Pods' replication, fault tolerance, self-healing, etc.
Examples of controllers are Deployments, ReplicaSets, ReplicationControllers, etc.
We attach a nested Pod's specification to a controller object using the Pod Template, as we have seen in the previous section.

Below is an example of a Pod object's configuration in YAML format:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.15.11
    ports:
    - containerPort: 80
```

The `apiVersion` field must specify **v1** for the **Pod** object definition.
The second required field is `kind` specifying the **Pod** object type.
The third required field `metadata`, holds the object's name and label.
The fourth required field `spec` marks the beginning of the block defining the desired state of the Pod object - also named the `PodSpec`.
Our Pod creates a single container running the `nginx:1.15.11` image from Docker Hub.

### Labels

Labels are key-value pairs attached to Kubernetes objects (e.g. Pods, ReplicaSets).
Labels are used to organize and select a subset of objects, based on the requirements in place.
Many objects can have the same Label(s).
Labels do not provide uniqueness to objects.
Controllers use Labels to logically group together decoupled objects, rather than using objects' names or IDs.

### Label Selectors

### ReplicationControllers

### ReplicaSets I

A ReplicaSet is the next-generation ReplicationController.
ReplicaSets support both equality- and set-based selectors, whereas ReplicationControllers only support equality-based Selectors.
Currently, this is the only difference.

With the help of the ReplicaSet, we can scale the number of Pods running a specific container application image.
Scaling can be accomplished manually or through the use of an autoscaler.

Next, you can see a graphical representation of a ReplicaSet, where we have set the replica count to 3 for a Pod.

### ReplicaSets II

### ReplicaSets III

The ReplicaSet will detect that the current state is no longer matching the desired state.
The ReplicaSet will create an additional Pod, thus ensuring that the current state matches the desired state.

ReplicaSets can be used independently as Pod controllers but they only offer a limited set of features.
A set of complementary features are provided by Deployments, the recommended controllers for the orchestration of Pods.
Deployments manage the creation, deletion, and updates of Pods.
A Deployment automatically creates a ReplicaSet, which then creates a Pod.
There is no need to manage ReplicaSets and Pods separately, the Deployment will manage them on our behalf.

### Deployments I

Deployment objects provide declarative updates to Pods and ReplicaSets.
The DeploymentController is part of the master node's controller manager, and it ensures that the current state always matches the desired state.
It allows for seamless application updates and downgrades through `rollouts` and `rollbacks`, and it directly manages its ReplicaSets for application scaling. 

### Deployments II

### Deployments III

### Namespaces

If multiple users and teams use the same Kubernetes cluster we can partition the cluster into virtual sub-clusters using Namespaces.
The names of the resources/objects created inside a Namespace are unique, but not across Namespaces in the cluster.

To list all the Namespaces, we can run the following command:
```bash
kubectl get namespaces
```

Generally, Kubernetes creates four default Namespaces: kube-system, kube-public, kube-node-lease, and default.
The kube-system Namespace contains the objects created by the Kubernetes system, mostly the control plane agents.
The default Namespace contains the objects and resources created by administrators and developers.
By default, we connect to the default Namespace.
kube-public is a special Namespace, which is unsecured and readable by anyone, used for special purposes such as exposing public (non-sensitive) information about the cluster.
The newest Namespace is kube-node-lease which holds node lease objects used for node heartbeat data.
Good practice, however, is to create more Namespaces to virtualize the cluster for users and developer teams.

With Resource Quotas, we can divide the cluster resources within Namespaces.
We will briefly cover **resource** quotas in one of the future chapters.

## Chapter 9 Authentication, Authorization, Admission Control

### Authentication, Authorization, and Admission Control - Overview

To access and manage any Kubernetes resource or object in the cluster, we need to access a specific API endpoint on the API server.
Each access request goes through the following three stages:
* Authentication
Logs in a user.
* Authorization
Authorizes the API requests added by the logged-in user.
* Admission Control
Software modules that can modify or reject the requests based on some additional checks, like a pre-set **Quota**.

### Authentication I

## Chapter 10 Services

### Service Discovery

As Services are the primary mode of communication in Kubernetes, we need a way to discover them at runtime.
Kubernetes supports two methods for discovering Services:
* Environment Variables
As soon as the Pod starts on any worker node, the kubelet daemon running on that node adds a set of environment variables in the Pod for all active Services.
* DNS
Kubernetes has an add-on for DNS, which creates a DNS record for each Service and its format is **my-svc.my-namespace.svc.cluster.local**.
Services within the same Namespace find other Services just by their name.

This is the most common and highly recommended solution.

### ServiceType

## Chapter 11 Deploying a Standalone Application

### Liveness

If a container in the Pod is running, but the application running inside this container is not responding to our requests, then that container is of no use to us.
This kind of situation can occur, for example, due to application deadlock or memory pressure.
In such a case, it is recommended to restart the container to make the application available.

Rather than restarting it manually, we can use a **Liveness Probe**.
Liveness probe checks on an application's health, and if the health check fails, `kubelet` restarts the affected container automatically.

Liveness Probes can be set by defining:
* Liveness command
* Liveness HTTP request
* TCP Liveness Probe

We will discuss these three approaches in the next few sections.

### Liveness Command

### Readiness Probes

## Chapter 12 Kubernetes Volume Management

### Volumes

As we know, containers running in Pods are ephemeral in nature.
All data stored inside a container is deleted if the container crashes.
However, the `kubelet` will restart it with a clean slate, which means that it will not have any of the old data.

To overcome this problem, Kubernetes uses Volumes.
A Volume is essentially a directory backed by a storage medium.
The storage medium, content and access mode are determined by the Volume Type.

In Kubernetes, a Volume is attached to a Pod and can be shared among the containers of that Pod.
The Volume has the same life span as the Pod, and it outlives the containers of the Pod - this allows data to be preserved across container restarts.

### Volume Types

A directory which is mounted inside a Pod is backed by the underlying Volume Type.
A Volume Type decides the properties of the directory, like size, content, default access modes, etc.
Some examples of Volume Types are:

* emptyDir
An empty Volume is created for the Pod as soon as it is scheduled on the worker node.
The Volume's life is tightly coupled with the Pod.
If the Pod is terminated, the content of `emptyDir` is deleted forever.  
* hostPath
With the `hostPath` Volume Type, we can share a directory from the host to the Pod.
If the Pod is terminated, the content of the Volume is still available on the host.
* gcePersistentDisk
With the `gcePersistentDisk` Volume Type, we can mount a Google Compute Engine (GCE) persistent disk into a Pod.
* awsElasticBlockStore
With the `awsElasticBlockStore` Volume Type, we can mount an AWS EBS Volume into a Pod. 
* azureDisk
With azureDisk we can mount a Microsoft Azure Data Disk into a Pod.
* azureFile
With azureFile we can mount a Microsoft Azure File Volume into a Pod.
* cephfs
With `cephfs`, an existing CephFS volume can be mounted into a Pod.
When a Pod terminates, the volume is unmounted and the contents of the volume are preserved.
* nfs
With nfs, we can mount an NFS share into a Pod.
* iscsi
With iscsi, we can mount an iSCSI share into a Pod.
* secret
With the `secret` Volume Type, we can pass sensitive information, such as passwords, to Pods.
We will take a look at an example in a later chapter.
* configMap
With `configMap` objects, we can provide configuration data, or shell commands and arguments into a Pod.
* persistentVolumeClaim
We can attach a `PersistentVolume` to a Pod using a `persistentVolumeClaim`.
We will cover this in our next section. 

You can learn more details about Volume Types in the [Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/volumes/).

### PersistentVolumes

In a typical IT environment, storage is managed by the storage/system administrators.
The end user will just receive instructions to use the storage but is not involved with the underlying storage management.

In the containerized world, we would like to follow similar rules, but it becomes challenging, given the many Volume Types we have seen earlier.
Kubernetes resolves this problem with the **PersistentVolume (PV)** subsystem, which provides APIs for users and administrators to manage and consume persistent storage.
To manage the Volume, it uses the PersistentVolume API resource type, and to consume it, it uses the PersistentVolumeClaim API resource type.

A Persistent Volume is a network-attached storage in the cluster, which is provisioned by the administrator.

PersistentVolumes can be dynamically provisioned based on the StorageClass resource.
A StorageClass contains pre-defined provisioners and parameters to create a PersistentVolume.
Using PersistentVolumeClaims, a user sends the request for dynamic PV creation, which gets wired to the StorageClass resource.

Some of the Volume Types that support managing storage using PersistentVolumes are:
* GCEPersistentDisk
* AWSElasticBlockStore
* AzureFile
* AzureDisk
* CephFS
* NFS
* iSCSI

For a complete list, as well as more details, you can check out the Kubernetes documentation.

### PersistentVolumeClaims

A PersistentVolumeClaim (PVC) is a request for storage by a user.
Users request for PersistentVolume resources based on type, access mode, and size.
There are three access modes: ReadWriteOnce (read-write by a single node), ReadOnlyMany (read-only by many nodes), and ReadWriteMany (read-write by many nodes).
Once a suitable PersistentVolume is found, it is bound to a PersistentVolumeClaim.

After a successful bound, the PersistentVolumeClaim resource can be used in a Pod.

Once a user finishes its work, the attached PersistentVolumes can be released.
The underlying PersistentVolumes can then be reclaimed (for an admin to verify and/or aggregate data), deleted (both data and volume are deleted), or recycled for future usage (only data is deleted). 

To learn more, you can check out the [Kubernetes documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims).

### Container Storage Interface (CSI)

Container orchestrators like Kubernetes, Mesos, Docker or Cloud Foundry used to have their own methods of managing external storage using Volumes.
For storage vendors, it was challenging to manage different Volume plugins for different orchestrators.
Storage vendors and community members from different orchestrators started working together to standardize the Volume interface;
a volume plugin built using a standardized CSI designed to work on different container orchestrators.
You can find CSI specifications here.

Between Kubernetes releases v1.9 and v1.13 CSI matured from alpha to stable support, which makes installing new CSI-compliant Volume plugins very easy.
With CSI, third-party storage providers can develop solutions without the need to add them into the core Kubernetes codebase. 
