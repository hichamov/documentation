# Istio Documentation

## Concepts

### What is Istio ?

It is an open source service mesh that helps us connect, secure and monitor microservices platforms

Features:

* Encryption
* Requests retries, failover
* Intelligent load balancing
* Routing desicion
* Metrics/Logs/Traces
* Access control

#### What is a service mesh ?

The term service mesh is used to describe the network of microservices that make up such applications and the interactions between them.

It offers load balancing, failure recovery, metrics, and monitoring, a service mesh also often has more complex operational requirements, like A/B testing, canary rollouts, rate limiting, access control, and end-to-end authentication.

#### Why use Istio?

Ease of use, by injecting a sidecar proxy (Envoy) to our application's pods, it intercepts all traffic and makes us able to manage:

* Automatic load balancing for HTTP, gRPC, WebSocket, and TCP traffic.

* Fine-grained control of traffic behavior with rich routing rules, retries, failovers, and fault injection.

* A pluggable policy layer and configuration API supporting access controls, rate limits and quotas.

* Automatic metrics, logs, and traces for all traffic within a cluster, including cluster ingress and egress.

* Secure service-to-service communication in a cluster with strong identity-based authentication and authorization.

<img src="../images/arch.svg"/>


#### Core features

The main capabilites of istio are:

* Traffic management: control the flow of traffic and API calls between services
  
* Security: Istio provides the underlying secure communication channel, and manages authentication, authorization, and encryption of service communication at scale
  
* Observability: Istio’s robust tracing, monitoring, and logging features give you deep insights into your service mesh deployment

### Architecture

An Istio service mesh is logically split into a data plane and a control plane.

* The data plane is composed of a set of intelligent proxies (Envoy) deployed as sidecars. These proxies mediate and control all network communication between microservices. They also collect and report telemetry on all mesh traffic.

* The control plane manages and configures the proxies to route traffic.

Components:

* Envoy: A high-performance proxy developed in C++ to mediate all inbound and outbound traffic for all services in the service mesh, it allows traffic control with rich routing rules for http, grpc .., network resiliency, security and authentication.

* Istiod: Istiod provides service discovery, configuration and certificate management, and converts high-level rules into Envoy specific configuration, enables service-to-service authentication & encryption, and acts as a CA for all services in the mesh. 

### Traffic Management

Traffic management is done by injecting Envoy containers to application's pods, all the traffic that runs in these proxies is called 'data plane'.  

The configuration resides in the 'Control plane' istiod, which translates high-level routing rules to Envoy config.

In order to manage traffic in our cluster, Istio needs to detect all the endpoints of application's services, & populate it to its own service registry.

When installing Istio in k8s, it automatically connects to k8s API & fetches all services & endpoints. 

Envoy then uses these endpoints to distribute traffic to all endpoints of a service in a round-robin fashion.

All the principles we mentioned above are the default configuration of istio, but we can do more, from fine-grained control to A/B testing, through Istio's traffic management API.

Like other Istio configuration, the API is specified using Kubernetes custom resource definitions (CRDs), which you can configure using YAML.

#### Virtual Services:

A virtual service is a traffic routing functionality that lets us decide how ingress requests will be routed to which service in the service mesh.

Based on routes configured in the virtual service, we can tell Envoy to distribute traffic in specific ways, for example, send 50% of traffic to v1 and 50% to v2.

v1 and v2 services are called service subsets, and are specified in 'destination rules'

Example

The following virtual service routes requests to different versions of a service depending on whether the request comes from a particular user.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v3
```

* The host field is the one used with user to access the application
* The routing rule, uses to route http traffix in our case, and has a condition, all requests that have end-user, exact field equal to jason will match this condition
* The destination: when the request matches, we have to specify the service that this request will routed to, this must be a real k8s service, in this case 'reviews'.

Note: We have to specify the full service name, with the namespace included.

* The second destination: we can specify multiple destinations, and they are evaluated in sequential order, from top to bottom, in our case, the second rule is considered as a default route, because it has no routing condition

##### More about routing rules

This is a more advanced example, that routes requests to two different services based on the path and host header.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
    - bookinfo.com
  http:
  - match:
    - uri:
        prefix: /reviews
    route:
    - destination:
        host: reviews
  - match:
    - uri:
        prefix: /ratings
    route:
    - destination:
        host: ratings
```

In addition to using matching conditions, we can use 'weight' to distribute our traffic, this is useful for canary rollouts and A/B testing

ex:

```
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 75
    - destination:
        host: reviews
        subset: v2
      weight: 25
```

We can also do some actions on the traffic:

* Append or remove headers
* Rewrite the URL
* Set a retry policyfor calls to this destination

#### Destination rules:

Virtual services describes how we route traffic to a given destination, while destination rules are used  to configure what happens to traffic for that destination. Destination rules are applied after virtual service routing rules are evaluated, so they apply to the traffic’s “real” destination.

Destination rules are used to group services subsets, such as grouping v1 and v2 versions of a specific application in one subset, and allows us to customize envoy's traffic policies, such as loadbalancing policy, tls security, or circuit breaker.

#### Load Balancing options

By default, Istio uses a round-robin load balancing policy, it also has the following options:

* Random
* Weighted
* Least requests: Requests are forwarded to instances with the least number of requests.

ex:

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  host: my-svc
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: v3
    labels:
      version: v3
```

#### Gateways

You use a gateway to manage inbound and outbound traffic for your mesh, letting you specify which traffic you want to enter or leave the mesh

ex:

```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: ext-host-gwy
spec:
  selector:
    app: my-gateway-controller
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - ext-host.example.com
    tls:
      mode: SIMPLE
      serverCertificate: /tmp/tls.crt
      privateKey: /tmp/tls.key
```

This gateway configuration lets HTTPS traffic from ext-host.example.com into the mesh on port 443, but doesn’t specify any routing for the traffic.

To specify routing and for the gateway to work as intended, you must also bind the gateway to a virtual service. You do this using the virtual service’s gateways field, as shown in the following example:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: virtual-svc
spec:
  hosts:
  - ext-host.example.com
  gateways:
  - ext-host-gwy
```

#### Service entries

We use a service entry to add an entry to the service registry of istio so wa can reach external services, configuring services entries allows us to manage traffic for services running outside of the mesh, it offers the following options:

* Redirect and forward traffic for external destinations, such as APIs consumed from the web, or traffic to services in legacy infrastructure.
* Define retry, timeout, and fault injection policies for external destinations.
* Run a mesh service in a Virtual Machine (VM) by adding VMs to your mesh.
* Logically add services from a different cluster to the mesh to configure a multicluster Istio mesh on Kubernetes.

We cannot control traffic to destinations that aren't registred in the service registry.

ex:

```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: svc-entry
spec:
  hosts:
  - ext-svc.example.com
  ports:
  - number: 443
    name: https
    protocol: HTTPS
  location: MESH_EXTERNAL
  resolution: DNS
```

We specify the external resource in the 'hosts' field, we can configure virtual services and destination rules to control traffic to a service entry in a more granular way.


#### Sidecars

By default, Envoy sidecars can reach all other service in the mesh, this is not recommended, because of the high resources consumption, a better way is to restrict the access to the needed services

In the following example, all sidecars in 'bookinfo' namespace can reach all services in the same namespace, and 'istio-system' namespace:

```
apiVersion: networking.istio.io/v1alpha3
kind: Sidecar
metadata:
  name: default
  namespace: bookinfo
spec:
  egress:
  - hosts:
    - "./*"
    - "istio-system/*"
```

#### Network resilience and testing

As well as helping you direct traffic around your mesh, Istio provides opt-in failure recovery and fault injection features that you can configure dynamically at runtime. Using these features helps your applications operate reliably, ensuring that the service mesh can tolerate failing nodes and preventing localized failures from cascading to other nodes.

#### Timeouts

A timeout is the amount of time that an Envoy proxy should wait for replies from a given service, controlling timeout is disabled in istio by default, we can configure timeouts in the virtual service:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
    timeout: 10s
```

#### Retries

A retry setting specifies the maximum number of times an Envoy proxy attempts to connect to a service if the initial call fails. the default retry behavior is to retry twice beforce returning an error, if the default setting doesn't suit our application, we can set new ones in the virtual service, ex:

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - route:
    - destination:
        host: ratings
        subset: v1
    retries:
      attempts: 3
      perTryTimeout: 2s
```

#### Circuit breakers

Circuit breakers are another useful mechanism Istio provides for creating resilient microservice-based applications. In a circuit breaker, you set limits for calls to individual hosts within a service, such as the number of concurrent connections or how many times calls to this host have failed. Once that limit has been reached the circuit breaker “trips” and stops further connections to that host. Using a circuit breaker pattern enables fast failure rather than clients trying to connect to an overloaded or failing host.

Circuit breakers are set in the destination rule, ex:

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 100
```

#### Fault injection

After you’ve configured your network, including failure recovery policies, you can use Istio’s fault injection mechanisms to test the failure recovery capacity of your application as a whole. Fault injection is a testing method that introduces errors into a system to ensure that it can withstand and recover from error conditions

Istio allows us to inject fault at the application layer, we can inject two types of faults:

* Delays: Delays are timing failures. They mimic increased network latency or an overloaded upstream service.
* Aborts: Aborts are crash failures. They mimic failures in upstream services. Aborts usually manifest in the form of HTTP error codes or TCP connection failures.

For example, this virtual service introduces a 5 second delay for 1 out of every 1000 requests to the ratings service.

```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: ratings
spec:
  hosts:
  - ratings
  http:
  - fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
    route:
    - destination:
        host: ratings
        subset: v1
```
