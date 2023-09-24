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
```sh
kubectl get pod -n istio-system
```
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

Istio's configuration resources are implemented as Kubernetes custom resource definitions (CRDs).
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
Now that we're in the `istioinaction` namespace, let's take a look at what we're going to deploy.
The Kubernetes resource files for `catalog-service` can be found in the $SRC_BASE/services/catalog/kubernetes/catalog.yaml file and looks similar to this:
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: catalog
  name: catalog
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 3000
  selector:
    app: catalog
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: catalog
    version: v1
  name: catalog
spec:
  replicas: 1
  selector:
    matchLabels:
      app: catalog
      version: v1
  template:
    metadata:
      labels:
        app: catalog
        version: v1
    spec:
      containers:
      - env:
        - name: KUBERNETES_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        image: istioinaction/catalog:latest
        imagePullPolicy: IfNotPresent
        name: catalog
        ports:
        - containerPort: 3000
          name: http
          protocol: TCP
        securityContext:
          privileged: false
```
Before we deploy this, however, we want to inject the Istio service proxy so that this service can participate in the service mesh.
From the root of the source code, run the `istioctl` command we introduced earlier:
```sh
istioctl kube-inject -f services/catalog/kubernetes/catalog.yaml
```
The `istioctl kube-inject` command takes a Kubernetes resource file and enriches it with the sidecar deployment of the Istio service proxy and a few additional components (elaborated on in appendix B).
Recall from chapter 1 that a *sidecar* deployment packages a complementing container alongside the main application container: they work together to deliver some functionality.
In the case of Istio, the sidecar is the service proxy, and the main application container is your application code.
If you look through the previous command's output, the YAML now includes a few extra containers as part of the deployment.
Most notably, you should see the following:
```yaml
      - args:
        - proxy
        - sidecar
        - --domain
        - $(POD_NAMESPACE).svc.cluster.local
        - --serviceCluster
        - catalog.$(POD_NAMESPACE)
        - --proxyLogLevel=warning
        - --proxyComponentLogLevel=misc:error
        - --trust-domain=cluster.local
        - --concurrency
        - "2"
        env:
        - name: JWT_POLICY
          value: first-party-jwt
        - name: PILOT_CERT_PROVIDER
          value: istiod
        - name: CA_ADDR
          value: istiod.istio-system.svc:15012
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
...
        image: docker.io/istio/proxyv2:{1.13.0}
        imagePullPolicy: Always
        name: istio-proxy
```
In Kubernetes, the smallest unit of deployment is called a *Pod*.
A Pod can be one or more containers deployed atomically together.
When we run `kube-inject`, we add another container named `istio-proxy` to the Pod template in the `Deployment` object, although we haven't actually deployed anything yet.
We could deploy the YAML file created by the `kube-inject` command directly;
however, we are going to take advantage of Istio's ability to automatically inject the sidecar proxy.

To enable automatic injection, we label the `istioinaction` namespace with `istio-injection=enabled`:
```sh
kubectl label namespace istioinaction istio-injection=enabled
```
Now let's create the `catalog` deployment:
```sh
kubectl apply -f services/catalog/kubernetes/catalog.yaml
```

After things come to a steady state, you should see the Pod with `Running` in the `Status` column, as in the previous snippet.
Also note the `2/2` in the `Ready` column: this means there are two containers in the Pod, and two of them are in the `Ready` state.
One of those containers is the application container, `catalog` in this case.
The other container is the `istio-proxy` sidecar.

At this point, we can query the `catalog` service *from within* the Kubernetes cluster with the hostname `catalog.istioinaction`.
Run the following command to verify everything is up and running properly.
If you see the following JSON output, the service is up and running correctly:
```sh
kubectl run -i -n default --rm --restart=Never dummy --image=curlimages/curl --command -- sh -c 'curl -s http://catalog.istioinaction/items/1'
```

So far, all we've done is deploy the `catalog` and `webapp` services with the Istio service proxies.
Each service has its own sidecar proxy, and all traffic to or from the individual services goes through the respective sidecar proxy (see figure 2.7).

### 2.4 Exploring the power of Istio with resilience, observability, and traffic control

In the previous example, we had to port-forward the `webapp` service locally because, so far, we have no way of getting traffic into the cluster.
With Kubernetes, we typically use an ingress controller like Nginx or a dedicated API gateway like Solo.io's Gloo Edge to do that.
With Istio, we can use an Istio ingress gateway to get traffic into the cluster, so we can call our web application.
In chapter 4, we'll look at why the out-of-the-box Kubernetes ingress resource is not sufficient for typical enterprise workloads and how Istio has the concepts of `Gateway` and `VirtualService` resources to solve those challenges.
For now, we'll use the Istio ingress gateway to expose our webapp service:
```sh
kubectl apply -f ch2/ingress-gateway.yaml
```
At this point, we've made Istio aware of the `webapp` service at the edge of the Kubernetes cluster, and we can call into it.
Let's see whether we can reach our service.
First we need to get the endpoint on which the Istio gateway is listening.
On Docker Desktop, it defaults to http://localhost:80:
```sh
curl http://localhost:80/api/catalog/items/1
```

If you have encountered any errors up to this point, go back and make sure you successfully complete all of the steps.
If you still encounter errors, ensure that the Istio ingress gateway has a route to our `webapp` service set up properly.
To do that, you can use Istio's debugging tools to check the configuration of the ingress gateway proxy.
You can use the same technique to check any Istio proxy deployed with any application, but we'll come back to that.
For now, check whether your gateway has a route:
```sh
istioctl proxy-config routes deploy/istio-ingressgateway.istio-system
```
You should see something similar to this:
```
NAME          VHOST NAME     DOMAINS     MATCH                  VIRTUAL SERVICE
http.8080     *:80           *           /*                     webapp-virtualservice.istioinaction
              backend        *           /stats/prometheus*
              backend        *           /healthz/ready*
```
If you don't, your best bet is to double-check that the gateway and virtual service resources were installed:
```sh
kubectl get gateway
kubectl get virtualservice
```
Additionally, make sure they are applied in the `istioinaction` namespace: in the virtual service definition, we use the abbreviated hostname (`webapp`), which lacks the namespace and defaults to the namespace the virtual service is applied to.
You can also add the namespace by updating the virtual service to route traffic to the host
`webapp.istioinaction`.

#### 2.4.1 Istio observability

Since the Istio service proxy sits in the call path on both sides of the connection (each service has its own service proxy), Istio can collect a lot of telemetry and insight into what's happening between applications.
Istio's service proxy is deployed as a sidecar alongside each application, so the insight it collects is from "out of process" of the application.
For the most part, this means applications do not need library- or framework-specific implementations to accomplish this level of observability.
The application is a black box to the proxy, and telemetry is focused on the application's behavior as observed through the network.

Istio creates telemetry for two major categories of observability.
The first is top-line metrics or things like requests per second, number of failures, and tail-latency percentiles.
Knowing these values can provide great insight into where problems are starting to arise in a system.
Second, Istio can facilitate distributed tracing like OpenTracing.io.
Istio can send spans to distributed-tracing backends without applications having to worry about it.
This way, we can dig into what happened during a particular service interaction, see where latency occurred, and get information about overall call latency.
Let's explore these capabilities hands-on with our example application.

We'll first look at some Istio observability features we can get out of the box.
In the previous section, we added two Kubernetes deployments and injected them with the Istio sidecar proxies.
Then we added an Istio ingress gateway, so we could reach our service from outside the cluster.
To get metrics, we will use Prometheus and Grafana.

Istio by default comes with some sample add-ons or supporting components that we installed earlier.
As noted in the previous sections, these components from the Istio installation are intended for demo purposes only.
For a production setup, you should install each supporting component following its respective documentation.
Referring again to the diagram of the control plane (figure 2.8), we can see how these components fit in.

Let's use `istioctl` to port-forward Grafana to our local machine, so we can see the dashboards:
```sh
istioctl dashboard grafana
```
This should automatically open your default browser; if it doesn't, open a browser and go to http://localhost:3000.
You should arrive at the Grafana home screen, as shown in figure 2.9.
In the upper-left corner, select the Home dashboard to expose a drop-down list of other dashboards we can switch to.

#### 2.4.2 Istio for resiliency

As we've discussed, applications that communicate over the network to help complete their business logic must be aware of and account for the fallacies of distributed computing: they need to deal with network unpredictability.
In the past, we tried to include a lot of this networking workaround code in our applications by doing things like retries, timeouts, circuit-breaking, and so on.
Istio can save us from having to write this networking code directly into our applications and provide a consistent, default expectation of resilience for all the applications in the service mesh.

#### 2.4.3 Istio for traffic routing

The last Istio capability we'll look at in this chapter is the ability to have very fine-grained control over requests in the service mesh no matter how deep they are in a call graph.

## Chapter 3. Istio's data plane: The Envoy proxy

When we introduced the idea of a service mesh in chapter 1, we established the concept of a service proxy and how this proxy understands application-level constructs (for example, application protocols like HTTP and gRPC) and enhances an application's business logic with non-differentiating application-networking logic.
A service proxy runs collocated and out of process with the application, and the application talks through the service proxy whenever it wants to communicate with other services.

With Istio, the Envoy proxies are deployed collocated with all application instances participating in the service mesh, thus forming the service-mesh data plane.
Since Envoy is such a critical component in the data plane and in the overall service-mesh architecture, we spend this chapter getting familiar with it.
This should give you a better understanding of Istio and how to debug or troubleshoot your deployments.

### 3.1 What is the Envoy proxy?

Envoy was developed at Lyft to solve some of the difficult application networking problems that crop up when building distributed systems.
It was contributed as an open source project in September 2016, and a year later (September 2017) it joined the Cloud Native Computing Foundation (CNCF).
Envoy is written in C++ in an effort to increase performance and, more importantly, to make it more stable and deterministic at higher load echelons.

Envoy is a proxy, so before we go any further, we should make very clear what a proxy is.
We already mentioned that a *proxy* is an intermediary component in a network architecture that is positioned in the middle of the communication between a client and a server (see figure 3.1).
Being in the middle enables it to provide additional features like security, privacy, and policy.

Proxies can simplify what a client needs to know when talking to a service.
For example, a service may be implemented as a set of identical instances (a cluster), each of which can handle a certain amount of load.
How should the client know which instance or IP address to use when making requests to that service?
A proxy can stand in the middle with a single identifier or IP address, and clients can use that to talk to the instances of the service.
Figure 3.2 shows how the proxy handles load balancing across the instances of the service without the client knowing any details of how things are actually deployed.
Another common function of this type of reverse proxy is checking the health of instances in the cluster and routing traffic around failing or misbehaving backend instances.
This way, the proxy can protect the client from having to know and understand which backends are overloaded or failing.

The Envoy proxy is specifically an application-level proxy that we can insert into the request path of our applications to provide things like service discovery, load balancing, and health checking, but Envoy can do more than that.
We've hinted at some of these enhanced capabilities in earlier chapters, and we'll cover them more in this chapter.
Envoy can understand layer 7 protocols that an application may speak when communicating with other services.
For example, out of the box, Envoy understands HTTP 1.1, HTTP 2, gRPC, and other protocols and can add behavior like request-level timeouts, retries, per-retry timeouts, circuit breaking, and other resilience features.
Something like this cannot be accomplished with basic connection-level (L3/L4) proxies that only understand connections.

Envoy can be extended to understand protocols in addition to the out-of-the-box defaults.
Filters have been written for databases like MongoDB, DynamoDB, and even asynchronous protocols like Advanced Message Queuing Protocol (AMQP).
Reliability and the goal of network transparency for applications are worthwhile endeavors, but just as important is the ability to quickly understand what's happening in a distributed architecture, especially when things are not working as expected.
Since Envoy understands application-level protocols and application traffic flows through Envoy, the proxy can collect lots of telemetry about the requests flowing through the system, such as how long they're taking, how many requests certain services are seeing (throughput), and what error rates the services are experiencing.
We will cover Envoy's telemetry-collection capabilities in chapter 7 and its extensibility in chapter 14.

As a proxy, Envoy is designed to shield developers from networking concerns by running out-of-process from applications.
This means any application written in any programming language or with any framework can take advantage of these features.
Moreover, although services architectures (SOA, microservices, and so on) are the architecture de jour, Envoy doesn't care if you're doing microservices or if you have monoliths and legacy applications written in any language.
As long as they speak protocols that Envoy can understand (like HTTP), Envoy can provide benefits.

Envoy is a very versatile proxy and can be used in different roles: as a proxy at the edge of your cluster (as an ingress point), as a shared proxy for a single host or group of services, and even as a per-service proxy as we see with Istio.
With Istio, a single Envoy proxy is deployed per service instance to achieve the most flexibility, performance, and control.
Just because you use one type of deployment pattern (a sidecar service proxy) doesn't mean you cannot also have the edge served with Envoy.
In fact, having the proxy be the same implementation at the edge as well as located within the application traffic can make your infrastructure easier to operate and reason about.
As we'll see in chapter 4, Envoy can be used at the edge for ingress and to tie into the service mesh to give full control and observe traffic from the point it enters the cluster all the way to the individual services in a call graph for a particular request.

#### 3.1.1 Envoy's core features

#### 3.1.2 Comparing Envoy to other proxies

### 3.2 Configuring Envoy

Envoy is driven by a configuration file in either JSON or YAML format.
The configuration file specifies listeners, routes, and clusters as well as server-specific settings like whether to enable the Admin API, where access logs should go, tracing engine configuration, and so on.
If you are already familiar with Envoy or Envoy configuration, you may know that there are different versions of the Envoy config.
The initial versions, v1 and v2, have been deprecated in favor of v3.
We look only at v3 configuration in this book, as that's the go-forward version and is what Istio uses.

### 3.3 Envoy in action

Envoy is written in C++ and compiled to a native/specific platform.
The best way to get started with Envoy is to use Docker and run a Docker container with it.
We've been using Docker Desktop for this book, but access to any Docker daemon can be used for this section.
For example, on a Linux machine, you can directly install Docker.

### 3.4 How Envoy fits with Istio

Envoy provides the bulk of the heavy lifting for most of the Istio features we covered in chapter 2 and throughout this book.
As a proxy, Envoy is a great fit for the service-mesh use case;
however, to get the most value out of Envoy, it needs supporting infrastructure or components.
The supporting components that allow for user configuration, security policies, and runtime settings, which Istio provides, create the control plane.
Envoy also does not do all the work in the data plane and needs support.
To learn more about that, see appendix B.

Let's illustrate the need for supporting components with a few examples.
We saw that due to Envoy's capabilities, we can configure a fleet of service proxies using static configuration files or a set of *xDS discovery services* for discovering listeners, endpoints, and clusters at run time. Istio implements these xDS APIs in the `istiod` control-plane component.

## Chapter 4. Istio gateways: Getting traffic into a cluster

We usually run interesting services and applications inside our cluster.
And as we'll see throughout the book, Istio allows us to solve some difficult challenges in service-to-service communication.
It is this intra-service communication where Istio shines (within a cluster or across clusters).

Before services communicate with each other, something must trigger the interactions.
For example, an end user purchasing an item, a client querying our API, and so on.
What each of these triggers have in common is that they originate outside of the cluster.
This raises the question: how do we get traffic from the outside of the cluster and into it (see figure 4.1)?
In this chapter, we will answer the question by opening an entry point for clients that live outside the cluster to connect securely to services running inside the cluster.

### 4.1 Traffic ingress concepts

The networking community has a term for connecting networks via well-established entry points: *ingress points*.
*Ingress* refers to traffic that originates outside the network and is intended for an endpoint within the network.
The traffic is first routed to an ingress point that acts as a gatekeeper for traffic coming into the network.
The ingress point enforces rules and policies about what traffic is allowed into the local network.
If the ingress point allows the traffic, it proxies the traffic to the correct endpoint in the local network.
If the traffic is not allowed, the ingress point rejects the traffic.

#### 4.1.1 Virtual IPs: Simplifying service access

At this point, it's useful to dig a little further into how traffic is routed to a network's ingress points - at least, how it relates to the type of clusters we look at in this book.
Let's say we have a service that we wish to expose at api.istioinaction.io/v1/products for external systems to get a list of products in our catalog.
When our client tries to query that endpoint, the client's networking stack first tries to resolve the api.istioinaction.io domain name to an IP address.
This is done with DNS servers.
The networking stack queries the DNS servers for the IP addresses for a particular hostname.
So the first step in getting traffic into our network is to map our service's IP to a hostname in DNS.
For a public address, we could use a service like Amazon Route 53 or Google Cloud DNS and map a domain name to an IP address.
In our own datacenters, we'd use internal DNS servers to do the same thing.
But to what IP address should we map the name?

Figure 4.2 visualizes why we should not map the name directly to a single instance or endpoint of our service (single IP), as that approach can be very fragile.
What would happen if that one specific service instance went down?
Clients would see many errors until we changed the DNS mapping to a new IP address with a working endpoint.
But doing this any time a service goes down is slow, error-prone, and low availability.

Figure 4.3 shows how mapping the domain name to a virtual IP address that represents our service and forwards traffic to our actual service instances, provides us with higher-availability and flexibility.
The virtual IP is bound to a type of ingress point known as a *reverse proxy*.
The reverse proxy is an intermediary component that's responsible for distributing requests to backend services and does not correspond to any specific service.
The reverse proxy can also provide capabilities like load balancing so requests don't overwhelm any one backend.

#### 4.1.2 Virtual hosting: Multiple services from single access point

In the previous section, we saw how a single virtual IP can be used to address a service that may consist of many service instances with their own IPs;
however, the client only uses the virtual IP.
We can also represent multiple different hostnames using a single virtual IP.
For example, we could have both prod.istioinaction.io and api.istioinaction.io resolve to the same virtual IP address.
This would mean requests for both hostnames would end up going to the same virtual IP, and thus the same ingress reverse proxy would route the request.
If the reverse proxy was smart enough, it could use the `Host` HTTP header to further delineate which requests should go to which group of services (see figure 4.4).

Hosting multiple different services at a single entry point is known as *virtual hosting*.
We need a way to decide which virtual host group a particular request should be routed to.
With HTTP/1.1, we can use the `Host` header;
with HTTP/2, we can use the `:authority` header;
and with TCP connections, we can rely on Server Name Indication (SNI) with TLS.
We'll take a closer look at SNI later in this chapter.
The important thing to note is that the edge ingress functionality we see in Istio uses virtual IP routing and virtual hosting to route service traffic into the cluster.

### 4.2 Istio ingress gateways

Istio has the concept of an *ingress gateway* that plays the role of the network ingress point and is responsible for guarding and controlling access to the cluster from traffic that originates outside of the cluster.
Additionally, Istio's ingress gateway handles load balancing and virtual-host routing.

Figure 4.5 shows Istio's ingress gateway component allowing traffic into the cluster and performing reverse proxy functionality.
Istio uses a single Envoy proxy as the ingress gateway.
We saw in chapter 3 that Envoy is a capable service-to-service proxy, but it can also be used to load balance and route traffic from outside the service mesh to services running inside it.
All the features of Envoy that we discussed in the previous chapter are also available in an ingress gateway.

Let’s take a closer look at how Istio uses Envoy to implement its ingress gateway component.
As we saw when we installed Istio in chapter 2, figure 4.6 shows the list of components that make up the control plane and additional components that support the control plane.

If you'd like to verify that the Istio service proxy (Envoy proxy) is indeed running in the Istio ingress gateway, you can run something like this from the root directory of the book's source code:
```sh
kubectl -n istio-system exec deploy/istio-ingressgateway -- ps
```
You should see a process listing as the output, showing the Istio service proxy command line with both `pilot-agent` and `envoy` as the running processes.
The `pilot-agent` process initially configures and bootstraps the Envoy proxy;
and as we'll see in chapter 13, it implements a DNS proxy as well.

To configure Istio's ingress gateway to allow traffic into the cluster and through the service mesh, we'll start by exploring two Istio resources: `Gateway` and `VirtualService`.
Both are fundamental for getting traffic to flow in Istio, but we'll look at them only within the context of allowing traffic into the cluster.
We will cover VirtualService more fully in chapter 5.

#### 4.2.1 Specifying Gateway resources

To configure an ingress gateway in Istio, we use the `Gateway` resource and specify which ports we wish to open and what virtual hosts to allow for those ports.
The example `Gateway` resource we'll explore is quite simple and exposes an HTTP port on port 80 that accepts traffic destined for virtual host `webapp.istioinaction.io`:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: coolstore-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "webapp.istioinaction.io"
```
Our `Gateway` resource configures Envoy to listen on port 80 and expect HTTP traffic.
Let's create that resource and see what it does.
In the root of the book's source code is a ch4/coolstore-gw.yaml file.
To create the configuration, run the following:
```sh
kubectl -n istioinaction apply -f ch4/coolstore-gw.yaml
```
Let's see whether our settings took effect:
```sh
istioctl -n istio-system proxy-config listener deploy/istio-ingressgateway

ADDRESSES PORT  MATCH DESTINATION
0.0.0.0   8080  ALL   Route: http.8080
0.0.0.0   15021 ALL   Inline Route: /healthz/ready*
0.0.0.0   15090 ALL   Inline Route: /stats/prometheus*
```
If you see this output, you've exposed the HTTP port (port 80) correctly!
Looking at the routes for virtual services, we see that the gateway doesn't have any at the moment (you may see another route for Prometheus, but you can ignore it for now):
**NOTE** If you are not using Docker Desktop, the name of the listener (in this instance "http.8080") may be different.
Update the command below accordingly.
```sh
istioctl -n istio-system proxy-config route deploy/istio-ingressgateway -o json --name http.8080
```
Our listener is bound to a `blackhole` default route that routes everything to HTTP 404.
In the next section, we set up a virtual host to route traffic from port 80 to a service within the service mesh.

Before we go on, there's an important last point to be made.
The Pod running the gateway, whether that's the default `istio-ingressgateway` or your own custom gateway, must be able to listen on a port or IP that is exposed outside the cluster.
For example, on the local Docker Desktop that we're using for these examples, the ingress gateway is listening on port 80.
If you're deploying on a cloud service like Google Container Engine (GKE), make sure you use a service of type `LoadBalancer`, which gets an externally routable IP address. You can find more information at https://istio.io/v1.13/docs/tasks/traffic-management/ingress/.

Additionally, the default `istio-ingressgateway` does not need privileged access to open any ports, as it does not listen on any system ports (80 for HTTP).
`istio-ingressgateway` by default listens on port 8080;
however, whatever service or load balancer you use to expose the gateway is the actual port.
In our examples with Docker Desktop, we expose the service on port 80.

#### 4.2.2 Gateway routing with virtual services

So far, all we've done is configure an Istio gateway to expose a specific port, expect a specific protocol on that port, and define specific hosts to serve from the port/protocol pair.
When traffic comes into the gateway, we need a way to get it to a specific service within the service mesh;
and to do that, we'll use the `VirtualService` resource.
In Istio, a `VirtualService` resource defines how a client talks to a specific service through its fully qualified domain name, which versions of a service are available, and other routing properties (like retries and request timeouts).
We'll cover `VirtualService` in more depth in the next chapter when we explore traffic routing;
in this chapter, it's sufficient to know that `VirtualService` allows us to route traffic from the ingress gateway to a specific service.

An example of a `VirtualService` that routes traffic for the virtual host `webapp.istioinaction.io` to services deployed in our service mesh looks like this:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: webapp-vs-from-gw
spec:
  hosts:
  - "webapp.istioinaction.io"
  gateways:
  - coolstore-gateway
  http:
  - route:
    - destination:
        host: webapp
        port:
          number: 8080
```
With this `VirtualService` resource, we define what to do with traffic when it comes into the gateway.
In this case, as you can see from the `spec.gateways` field, these traffic rules apply only to traffic coming from the `coolstore-gateway` gateway definition, which we created in the previous section.
Additionally, we specify the virtual host `webapp.istioinaction.io` for which traffic must be destined for these rules to match.
An example of matching this rule is a client querying `http://webapp.istioinaction.io`, which resolves to an IP that the Istio gateway is listening on.
Additionally, a client can explicitly set the `Host` header in the HTTP request to be `webapp.istioinaction.io`, as we'll show through an example.

Again, verify that you're in the root directory of the source code:
```sh
kubectl apply -n istioinaction -f ch4/coolstore-vs.yaml
```
After a few moments (the configuration needs to sync; recall that configuration in the Istio service mesh is eventually consistent), we can re-run our commands to list the listeners and routes:
```sh
istioctl -n istio-system proxy-config route deploy/istio-ingressgateway -o json --name http.8080

[
  {
    "name": "http.8080",
    "virtualHosts": [
      {
      "name": "webapp-vs-from-gw:80",
      "domains": [
          "webapp.istioinaction.io"
      ],
      "routes": [
        {
          "match": {
            "prefix": "/"
          },
          "route": {
            "cluster": "outbound|8080||webapp.istioinaction.svc.cluster.local",
            "timeout": "0.000s"
          }
        }
      ]
      }
    ]
  }
]
```
The output for route should look similar to the previous listing, although it may contain other attributes and information.
The critical part is that we can see how defining a `VirtualService` created an Envoy route in our Istio gateway that routes traffic matching domain `webapp.istioinaction.io` to `webapp` in our service mesh.

We have the routing set up for our services, but we should deploy the services for them to work.
The following commands are meant to be run from the root of the book's source code:
```sh
kubectl config set-context $(kubectl config current-context) --namespace=istioinaction
kubectl apply -f services/catalog/kubernetes/catalog.yaml
kubectl apply -f services/webapp/kubernetes/webapp.yaml
```

Verify that your `Gateway` and `VirtualService` resources are installed correctly:
```sh
kubectl get gateway

kubectl get virtualservice
```

#### 4.2.3 Overall view of traffic flow

In the previous sections, we got hands-on with the `Gateway` and `VirtualService` resources from Istio.
The `Gateway` resource defines ports, protocols, and virtual hosts that we wish to listen for at the edge of our service-mesh cluster.
`VirtualService` resources define where traffic should go once it's allowed in at the edge.
Figure 4.7 shows the full end-to-end flow.

#### 4.2.4 Istio ingress gateway vs. Kubernetes Ingress

#### 4.2.5 Istio ingress gateway vs. API gateways

An API gateway allows an organization to abstract a client that consumes a set of services in a boundary (either network-wise or architectural) from the details of the implementation of those services.
For example, clients may call a set of APIs that are expected to be well documented, evolve with backward- and forward-compatible semantics, and offer self-service and other mechanisms for usage.
To accomplish this, the API gateway needs to be able to identify clients using different security challenges (OpenID Connect [OIDC], OAuth 2.0, Lightweight Directory Access Protocol [LDAP]), transform messages (SOAP to REST, gRPC to Rest, body and header text-based transformations, and so on), provide sophisticated business-level rate limiting, and have a self-signup or developer portal.
Istio's ingress gateway does not do these things out of the box.
For a more capable API gateway — even one built on an Envoy proxy — that can play this role inside and outside your mesh, take a look at something like Solo.io Gloo Edge (https://docs.solo.io/gloo-edge/latest).

### 4.3 Securing gateway traffic

So far, we've shown how to expose basic HTTP services with an Istio gateway using the `Gateway` and `VirtualService` resources.
When connecting services from outside a cluster (let's say, the public internet) to those running inside a cluster, one of the basic capabilities of the ingress gateway in a system is to secure traffic and help establish trust in the system.
We can begin to secure our traffic by giving clients confidence that the service they're hoping to communicate with is indeed the service it claims to be.
Additionally, we want to exclude anyone from eavesdropping on our communication, so we should encrypt the traffic.

Istio's gateway implementation allows us to terminate incoming TLS/SSL traffic, pass it through to the backend services, redirect any non-TLS traffic to the proper TLS ports, and implement mutual TLS.
We'll look at each of these capabilities in this section.
