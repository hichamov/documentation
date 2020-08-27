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


