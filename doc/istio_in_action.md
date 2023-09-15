# Istio in Action

## 1 Introducing the Istio service mesh

### 1.1 Challeges of going faster

The following things must be addressed when moving to a services-oriented architecture:
* Keeping faults from jumping isolation boundaries
* Building applications/services capable of responding to changes in their environment
* Building systems capable of running in partially failed conditions
* Understanding what's happening to the overall system as it constantly changes and evolves
* Inability to control the runtime behaviors of the system
* Implementing strong security as the attack surface grows
* Lowering the risk of making changes to the system
* Enforcing policies about who or what can use system components, and when

As we dig into Istio, we'll explore these in more detail and explain how to deal with them.
These are core challenges to building services-based architectures on any cloud infrastructure.
In the past, non-cloud architectures did have to contend with some of these problems; but in today's cloud environments, they are highly amplified and can take down your entire system if not taken into account correctly.
Let's look a little bit closer at the problems encountered with unreliable infrastructure.

#### 1.1.1 Our cloud infrastructure is not reliable

In the cloud, we have to build our apps assuming the infrastructure is ephemeral and will be unavailable at times.
This ephemerality must be considered upfront in our architectures.

#### 1.1.2 Making service interactions resilient

Some patterns have evolved to help mitigate these types of scenarios and help make applications more resilient to unplanned, unexpected failures:
* *Client-side load balancing* — Give the client the list of possible endpoints, and let it decide which one to call.
* *Service discovery* — A mechanism for finding the periodically updated list of healthy endpoints for a particular logical service.
* *Circuit breaking* — Shed load for a period of time to a service that appears to be misbehaving.
* *Bulkheading* — Limit client resource usage with explicit thresholds (connections, threads, sessions, and so on) when making calls to a service.
* *Timeouts* — Enforce time limitations on requests, sockets, liveness, and so on when making calls to a service.
* *Retries* — Retry a failed request.
* *Retry budgets* — Apply constraints to retries: that is, limit the number of retries in a given period (for example, only retry 50% of the calls in a 10-second window).
* *Deadlines* — Give requests context about how long a response may still be useful; if outside the deadline, disregard processing the request.

Collectively, these types of patterns can be thought of as *application networking*.
They have a lot of overlap with similar constructs at lower layers of the networking stack, except that they operate at the layer of messages instead of packets.

#### 1.1.3 Understanding what's happening in real time

### 1.2 Solving these challenges with application libraries

#### 1.2.1 Drawbacks to application-specific libraries

Although we've mitigated a concern about large-scale services architectures when we decentralize and distribute the implementation of application resiliency into the applications themselves, we've introduced some new challenges.
The first challenge is around the expected assumptions of any application.
If we wish to introduce a new service into our architecture, it will be constrained to implementation decisions made by other people and other teams.
For example, to use NetflixOSS Hystrix, you must use Java or a JVM-based technology.

### 1.3 Pushing these concerns to the infrastructure

These basic application-networking concerns are not specific to any particular application, language, or framework.
Retries, timeouts, client-side load balancing, circuit breaking, and so on are also not differentiating application features.
They are critical concerns to have as part of your service, but investing massive time and resources into language-specific implementations for each language you intend to use (including the other drawbacks from the previous section) is a waste of time.
What we really want is a technology-agnostic way to implement these concerns and relieve applications from having to do so themselves.

#### 1.3.1 The application-aware service proxy

Using a *proxy* is a way to move these horizontal concerns into the infrastructure.
A proxy is an intermediate infrastructure component that can handle connections and redirect them to appropriate backends.
We use proxies all the time (whether we know it or not) to handle network traffic, enforce security, and load balance work to backend servers.
For example, HAProxy is a simple but powerful reverse proxy for distributing connections across many backend servers.
`mod_proxy` is a module for the Apache HTTP server that also acts as a reverse proxy.
In our corporate IT systems, all outgoing internet traffic is typically routed through forwarding proxies in a firewall.
These proxies monitor traffic and block certain types of activities.

What we want for this problem, however, is a proxy that's *application aware* and able to perform application networking on behalf of our services (see figure 1.4).
To do so, this service proxy will need to understand application constructs like messages and requests, unlike more traditional infrastructure proxies, which understand connections and packets.
In other words, we need a layer 7 proxy.

#### 1.3.2 Meet the Envoy proxy

Envoy (http://envoyproxy.io) is a service proxy that has emerged in the open source community as a versatile, performant, and capable application-layer proxy.
Envoy was developed at Lyft as part of the company's SOA infrastructure and is capable of implementing networking concerns like retries, timeouts, circuit breaking, client-side load balancing, service discovery, security, and metrics collection without any explicit language or framework dependencies.
Envoy implements all of that out-of-process from the application, as shown in figure 1.5.

The power of Envoy is not limited to these application-layer resilience aspects.
Envoy also captures many application-networking metrics like requests per second, number of failures, circuit-breaking events, and more.
By using Envoy, we can automatically get visibility into what’s happening between our services, which is where we start to see a lot of unanticipated complexity.
The Envoy proxy forms the foundation for solving cross-cutting, horizontal reliability and observability concerns for a services architecture and allows us to push these concerns outside of the applications and into the infrastructure.
We'll cover more of Envoy in ensuing sections and chapters.

We can deploy service proxies alongside our applications to get these features (resilience and observability) out-of-process from the application, but at a fidelity that is very application specific.
Figure 1.6 shows how in this model, applications that wish to communicate with the rest of the system do so by passing their requests to Envoy first, which then handles the communication upstream.

Service proxies can also do things like collect distributed tracing spans so we can stitch together all the steps taken by a particular request.
We can see how long each step took and look for potential bottlenecks or bugs in our system.
If all applications talk through their own proxy to the outside world, and all incoming traffic to an application goes through our proxy, we gain some important capabilities for our application without changing any application code.
This proxy + application combination forms the foundation of a communication bus known as a *service mesh*.

### 1.4 What's a service mesh?

Service proxies like Envoy help add important capabilities to our services architecture running in a cloud environment.
Each application can have its own requirements or configurations for how a proxy should behave, given its workload goals.
With an increasing number of applications and services, it can be difficult to configure and manage a large fleet of proxies.
Moreover, having these proxies in place at each application instance opens opportunities for building interesting higher-order capabilities that we would otherwise have to do in the applications themselves.

A *service mesh* is a distributed application infrastructure that is responsible for handling network traffic on behalf of the application in a transparent, out-of-process manner.
Figure 1.8 shows how service proxies form the *data plane* through which all traffic is handled and observed.
The data plane is responsible for establishing, securing, and controlling the traffic through the mesh.
The data plane behavior is configured by the *control plane*.
The control plane is the brains of the mesh and exposes an API for operators to manipulate network behaviors.
Together, the data plane and the control plane provide important capabilities necessary in any cloud-native architecture:
* Service resilience
* Observability signals
* Traffic control capabilities
* Security
* Policy enforcement

The service mesh takes on the responsibility of making service communication resilient to failures by implementing capabilities like retries, timeouts, and circuit breakers.
It's also capable of handling evolving infrastructure topologies by handling things like service discovery, adaptive and zone-aware load balancing, and health checking.
Since all the traffic flows through the mesh, operators can control and direct traffic explicitly.
For example, if we want to deploy a new version of our application, we may want to expose it to only a small fraction, say 1%, of live traffic.
With the service mesh in place, we have the power to do that. Of course, the converse of control in the service mesh is understanding its current behavior.
Since traffic flows through the mesh, we’re able to capture detailed signals about the behavior of the network by tracking metrics like request spikes, latency, throughput, failures, and so on.
We can use this telemetry to paint a picture of what's happening in our system.
Finally, since the service mesh controls both ends of the network communication between applications, it can enforce strong security like transport-layer encryption with mutual authentication: specifically, using the mutual Transport Layer Security (mTLS) protocol.

The service mesh provides all of these capabilities to service operators with very few or no application code changes, dependencies, or intrusions.
Some capabilities require minor cooperation with the application code, but we can avoid large, complicated library dependencies.
With a service mesh, it doesn't matter what application framework or programming language you've used to build your application; these capabilities are implemented consistently and correctly and allow service teams to move quickly, safely, and confidently when implementing and delivering changes to systems to test their hypotheses and deliver value.

### 1.5 Introducing the Istio service mesh

Istio is an open source implementation of a service mesh founded by Google, IBM, and Lyft.
It helps you add resilience and observability to your services architecture in a transparent way.
With Istio, applications don't have to know that they're part of the service mesh: whenever they interact with the outside world, Istio handles the networking on their behalf.
It doesn't matter if you're using microservices, monoliths, or anything in between — Istio can bring many benefits.
Istio's data plane uses the Envoy proxy and helps you configure your application to have an instance of the service proxy (Envoy) deployed alongside it.
Istio's control plane is made up of a few components that provide APIs for end users/operators, configuration APIs for the proxies, security settings, policy declarations, and more.
We'll cover these control-plane components in future sections of this book.

Istio was originally built to run on Kubernetes but was written from the perspective of being deployment-platform agnostic.
This means you can use an Istio-based service mesh across deployment platforms like Kubernetes, OpenShift, and even traditional deployment environments like virtual machines (VMs).
In later chapters, we'll take a look at how powerful this can be for hybrid deployments across combinations of clouds, including private data centers.

#### 1.5.5 What are the drawbacks to using a service mesh?

We've talked a lot about the problems of building a distributed architecture and how a service mesh can help, but we don't want to give the impression that a service mesh is the one and only way to solve these problems or that a service mesh doesn't have drawbacks.
Using a service mesh does have a few drawbacks you must be aware of.

First, using a service mesh puts another piece of middleware, specifically a proxy, in the request path.
This proxy can deliver a lot of value; but for those unfamiliar with the proxy, it can end up being a black box and make it harder to debug an application's behavior.
The Envoy proxy is specifically built to be very debuggable by exposing a lot about what's happening on the network—more so than if it wasn't there — but for someone unfamiliar with operating Envoy, it could look very complex and inhibit existing debugging practices.

## 2 First steps with Istio

Istio solves some of the difficult challenges of service communication in cloud environments and provides a lot of capabilities to both developers and operators.
We'll cover these capabilities and how it all works in subsequent chapters; but to help you get a feel for some of the features of Istio, in this chapter we do a basic installation (more advanced installation options can be found in appendix A) and deploy a few services.
The services and examples come from the book's source code, which you can find at https://github.com/istioinaction/book-source-code.
From there, we explore the components that make up Istio and what functionality we can provide to our example services.
Finally, we look at how to do basic traffic routing, metrics collection, and resilience.
Further chapters will dive deeper into the functionality.

### 2.1 Deploying Istio on Kubernetes

We're going to deploy Istio and our example applications using containers, and we'll use the Kubernetes container platform to do that.
Kubernetes is a very powerful container platform capable of scheduling and orchestrating containers over a fleet of host machines known as Kubernetes *nodes*.
These nodes are host machines capable of running containers, but Kubernetes handles those mechanisms.
As we'll see, Kubernetes is a great place to initially kick the tires with Istio — although we should be clear that Istio is intended to support multiple types of workloads, including those running on virtual machines (VMs).

#### 2.1.1 Using Docker Desktop for the examples

#### 2.1.2 Getting the Istio distribution

#### 2.1.3 Installing the Istio components into Kubernetes

In the distribution you just downloaded and unpacked, the manifests directory contains a collection of charts and resource files for installing Istio into the platform of your choice.
The official method for any real installation of Istio is to use `istioctl`, `istio-operator`, or Helm.
Appendix A guides you through installing and customizing Istio using `istioctl` and `istio-operator`.

For this book, we use istioctl and various pre-curated profiles to take a step-by-step, incremental approach to adopting Istio.
To perform the demo install, use the `istioctl` CLI tool as shown next:
```sh
istioctl install --set profile=demo -y
```
After running this command, you may have to wait a few moments for the Docker images to properly download and the deployments to succeed.
Once things have settled in, you can run the `kubectl` command to list all of the Pods in the `istio-system` namespace.

The `istio-system` namespace is special in that the control plane is deployed into it and can act as a cluster-wide control plane for Istio.
Let's see what components are installed into the `istio-system` namespace:

What exactly did we install?
In chapter 1, we introduced the concept of a service mesh and said that Istio is an open source implementation of a service mesh.
We also said that a service mesh comprises data-plane (that is, service proxies) and control-plane components.
After installing Istio into a cluster, you should see the control plane and the ingress and egress gateways.
As soon as we install applications and inject the service proxies into them, we will have a data plane as well.

The astute reader may notice that for each component of the Istio control plane, there is only a single replica or instance. You may also be thinking, "This appears to be a single point of failure. What happens if these components fail or go down?"
That's a great question and one we'll cover throughout the book.
For now, know that the Istio control plane is intended to be deployed in a highly available architecture (with multiple replicas of each component).
In the event of failures of the control-plane components or even the entire control plane, the data plane is resilient enough to continue for periods of disconnection from the control plane.
Istio is implemented to be highly resilient to the myriad of failures that can occur in a distributed system.

The last thing we want to do is verify the installation.
We can run the `verify-install` command post-install to verify that it has completed successfully:
```sh
istioctl verify-install
```
Finally, we need to install the control-plane supporting components.
These components are not strictly required but should be installed for any real deployment of Istio.
The versions of the supporting components we install here are recommended for demo purposes only, not production usage.
From the root of the Istio distribution you downloaded, run the following to install the example supporting components:
```sh
kubectl apply -f ./samples/addons
```
Now, if we check the `istio-system` namespace, we see the supporting components installed:
```sh
kubectl get pod -n istio-system
```

### 2.2 Getting to know the Istio control plane

In the previous section, we did a demo installation of Istio that deployed all of the control-plane components and supporting components to Kubernetes.
The control plane provides a way for users of the service mesh to control, observe, manage, and configure the mesh.
For Istio, the control plane provides the following functions:
* APIs for operators to specify desired routing/resilience behavior
* APIs for the data plane to consume their configuration
* A service discovery abstraction for the data plane
* APIs for specifying usage policies
* Certificate issuance and rotation
* Workload identity assignment
* Unified telemetry collection
* Service-proxy sidecar injection
* Specification of network boundaries and how to access them

The bulk of these responsibilities is implemented in a single control-plane component called `istiod`.
Figure 2.1 shows `istiod` along with gateways responsible for ingress traffic and egress traffic.
We also see supporting components that are typically integrated with a service mesh to support observability or security use cases.
We'll take a closer look at all of these components in the forthcoming chapters.
Now, let's examine the control-plane components.

#### 2.2.1 Istiod

Istio's control-plane responsibilities are implemented in the `istiod` component.
`istiod`, sometimes referred to as Istio Pilot, is responsible for taking higher-level Istio configurations specified by the user/operator and turning them into proxy-specific configurations for each data-plane service proxy (see figure 2.2).

For example, through configuration resources, we can specify how traffic is allowed into the cluster, how it is routed to specific versions of services, how to shift traffic when doing a new deployment, and how callers of a service should treat resiliency aspects like timeouts, retries, and circuit breaking.
`istiod` takes these configurations, interprets them, and exposes them as service-proxy-specific configurations.
Istio uses Envoy as its service proxy, so these configurations are translated to Envoy configurations.
For example, for a service trying to talk to a `catalog` service, we may wish to send traffic to v2 of the service if it has the header `x-dark-launch` in its request.
We can express that for Istio with the following configuration:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: catalog-service
spec:
  hosts:
  - catalog.prod.svc.cluster.local
  http:
  - match:
    - headers:
        x-dark-launch:
          exact: "v2"
    route:
    - destination:
        host: catalog.prod.svc.cluster.local
        subset: v2
  - route:
    - destination:
        host: catalog.prod.svc.cluster.local
        subset: v1
```
For the moment, don't worry about the specifics, as this example is just to illustrate that this YAML configuration is translated to the data plane as a proxy-specific configuration.
The configuration specifies that, based on header matching, we would like to route a request to the `v2` deployment of the `catalog` service when there is a header `x-dark-launch` that equals `v2`;
and that for all other requests, we will route to `v1` of the `catalog` service.
As an operator of Istio running on Kubernetes, we would create this configuration using a tool like `kubectl`.
For example, if this configuration is stored in a file named `catalog-service.yaml`, we can create it as follows:
```sh
kubectl apply -f catalog-service.yaml
```
We'll dig deeper into what this configuration does later in the chapter.
For now, just know that configuring Istio traffic routing rules will use a similar pattern: describe intent in Istio resource files (YAML) and pass it to the Kubernetes API.

Istio’s configuration resources are implemented as Kubernetes custom resource definitions (CRDs).
CRDs are used to extend the native Kubernetes API to add new functionality to a Kubernetes cluster without having to modify any Kubernetes code.
In the case of Istio, we can use Istio's custom resources (CRs) to add Istio functionality to a Kubernetes cluster and use native Kubernetes tools to apply, create, and delete the resources.
Istio implements a controller that watches for these new CRs to be added and reacts to them accordingly.

Istio reads Istio-specific configuration objects, like `VirtualService` in the previous configuration, and translates them into Envoy's native configuration.
`istiod` exposes this configuration intent to the service proxies as Envoy configuration through its data-plane API:
```json
"domains": [
  "catalog.prod.svc.cluster.local"
],
"name": "catalog.prod.svc.cluster.local:80",
"routes": [
  {
    "match": {
      "headers": [
        {
          "name": "x-dark-launch",
          "value": "v2"
        }
      ],
      "prefix": "/"
    },
    "route": {
        "cluster":
        "outbound|80|v2|catalog.prod.svc.cluster.local",
        "use_websocket": false
    }
  },
  {
    "match": {
      "prefix": "/"
    },
    "route": {
      "cluster":
      "outbound|80|v1|catalog.prod.svc.cluster.local",
      "use_websocket": false
    }
  }
]
```
This data-plane API exposed by `istiod` implements Envoy's *discovery APIs*.
These discovery APIs, like those for service discovery (listener discovery service \[LDS\]), endpoints (endpoint discovery service \[EDS\]), and routing rules (route discovery service \[RDS\]) are known as the *xDS* APIs.
These APIs allow the data plane to separate how it is configured and dynamically adapt its behavior without having to stop and reload.
We'll cover these xDS APIs from the perspective of the Envoy proxy in chapter 3.

#### 2.2.2 Ingress and egress gateway

For our applications and services to provide anything meaningful, they need to interact with applications that live outside of our cluster.
Those could be existing monolith applications, off-the-shelf software, messaging queues, databases, and third-party partner systems.
To do this, operators need to configure Istio to allow traffic into the cluster and be very specific about what traffic is allowed to leave the cluster.
Modeling and understanding what traffic is allowed into and out of the cluster is good practice and improves our security posture.

Figure 2.4 shows the Istio components that provide this functionality: `istio-ingressgateway` and `istio-egressgateway`.
We saw those when we printed out the control plane components.

These components are really Envoy proxies that can understand Istio configurations.
Although they are not technically part of the control plane, they are instrumental in any real-world usage of a service mesh.
These components reside in the data plane and are configured very similarly to Istio service proxies that live with the applications.
The only actual difference is that they're independent of any application workload and are just to let traffic into and out of the cluster.
In future chapters, we'll see how these components play a role in combining clusters and even clouds.

### 2.3 Deploying your first application in the service mesh

To get the source code for this example, download it from http://istioinaction.io or clone it from https://github.com/istioinaction/book-source-code.
In the services directory, you should see the Kubernetes resource files that describe the deployment of our components.
The first thing to do is create a namespace in Kubernetes in which we'll deploy our services:
```sh
kubectl create namespace istioinaction
kubectl config set-context $(kubectl config current-context) --namespace=istioinaction
```
