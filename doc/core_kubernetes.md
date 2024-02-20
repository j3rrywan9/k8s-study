# Core Kubernetes

## 1 Why Kubernetes Exists

### 1.5 Kubernetes features

*Container orchestration platforms* allow developers to automate the process of running instances, provisioning hosts, linking containers to optimize orchestration procedures, and extending application life cycles.
It's time to dive into the core features within a container orchestration platform because, essentially, containers need Pods and Pods need Kubernetes to
* Allow for service discovery via a domain name service (DNS), implemented previously by KubeDNS and, most recently, by CoreDNS, which integrates with the API server

### 1.6 Kubernetes components and architecture

#### 1.6.1 The Kubernetes API

If there's one important thing to take away from this chapter that will enable you to go forth on a deep journey through this book, it's that administering microservices and other containerized software applications on a Kubernetes platform is just a matter of declaring Kubernetes API objects.
For the most part, everything else is done for you.

There are around 70 different API types that you can play with, create, edit, and delete in a standard Kubernetes cluster.
You can view these by running `kubectl api-resources`.

## 2 Why the Pod?

The Pod is the smallest atomic unit that can be deployed to a Kubernetes cluster, and Kubernetes is built around the Pod definition.

### 2.2 What is a Pod?

Roughly, a *Pod* is one or more OCI images that run as containers on a Kubernetes cluster node.
The Kubernetes *node* is a single piece of computing power (a server) that runs a kubelet.
Like everything else in Kubernetes, a Node is also an API object.

Pods aren't deployed directly in most cases.
Instead, they are automatically created for us by other API objects such as Deployments, Jobs, StatefulSets, and DaemonSets that we define:
* *Deployments*
* *Jobs*
* *StatefulSets*
* *DaemonSets*

#### 2.2.1 A bunch of Linux namespaces

#### 2.2.2 Kubernetes, infrastructure, and the Pod

The *kubelet* is a binary program that runs as an agent, communicating with the Kubernetes API server via multiple control loops.
It runs on every node; without it, a Kubernetes node is not schedulable or considered to be a part of a cluster.

### 2.3 Creating a web application with `kubectl`

#### 2.3.1 The Kubernetes API server: `kube-apiserver`

The Kubernetes API server, `kube-apiserver`, is an HTTP-based REST server that exposes the various API objects for a Kubernetes cluster; these objects range from Pods to nodes to the Horizontal Pod Autoscaler.
The API server validates and provides the web frontend to perform CRUD services on the cluster's shared state.

The components on the control plane communicate with the API server as well.

The API server is the only component on the control plane that communicates with etcd, the database for Kubernetes.
In the past, some other components like CNI network providers have communicated with etcd, but currently, most do not.
In essence, the API server provides a stateful interface for all operations modifying a Kubernetes cluster.
Security for all control plane components is essential, but securing the API server and its HTTPS endpoints is crucial.

*Admission controllers* that run as part of the API server provide both authentication and authorization when a client communicates with the API server.

#### 2.3.2 The Kubernetes scheduler: `kube-scheduler`

#### 2.3.3 Infrastructure controllers

Creating a LoadBalancer Service instead of the ClusterIP Service involves a Kubernetes cloud provider "watching" for a user load-balancer request and then fulfilling it (for example, by making calls to a cloud API to provision an external IP address and binding it to internal Kubernetes service endpoints).

### 2.4 Scaling, highly available applications, and the control plane

Executing *kubectl scale* can increase and decrease the amount of Pods that are running in a cluster.
It operates directly on the ReplicaSets, StatefulSets, or other API objects that use Pods, depending on the input you provide to the command.

#### 2.4.1 Autoscaling

#### 2.4.2 Cost management

## 3. Let's build a Pod
