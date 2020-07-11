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

### Why Use Container Orchestrators?

### Where to Deploy Container Orchestrators?

## Chapter 3 Kubernetes

### What Is Kubernetes?

### From Borg to Kubernetes

### Kubernetes Features I

### Kubernetes Features II

### Why Use Kubernetes?

### Cloud Native Computing Foundation (CNCF)

### CNCF and Kubernetes

## Chapter 4 Kubernetes Architecture

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
