# Prometheus & Grafana

## Prometheus

Prometheus is designes to monitor containerized distributed systems.

#### Main components

* Prometheus Server: The main piece, which also has 3 subcomponents, `Storage` whiche is a Time series database, `Retrieval` A process that pulls metrics from services & software pieces to monitor which is called a Data retrieval Worker, and finally, `UI` an HTTP server that accepts queries (Written in PromQL) 

Prometheus can monitor (OS, Web servers, DB servers, Applications ...) which are called `Targets`, & each target has units which can be (CPU consumption, Memory, Disk ...), data related to a unit is called a metric which are stored in prometheus database, and are in a Human Readeable format 

Each metric has two attributes:

    * HELP: A description of what is the metric about
    * TYPE: 
  * Counter: Counts haw many time an event happens, ex: how many requests an HTTP server receives (Shouldn't be used for values that go down)
  * Gauge: For values that can can go up & down, ex: CPU utilization, Memory utilization 
  * Histogram: How long or how big an events is, how long an application had an out of memory.
  * Summaries: Similar to histograms, but instead of calculating a rate of values in prometheus, it does it in application level, so we can't calculate the sum of a specific value from different endpoints.

Prometheus pulls data from endpoints using HTTP requests, for that, targets must expose a `/metrics` endpoint.

To make a target expose metrics, we need to install a piece of software called `exporter` alongside the target.

The full list of exporters can be found [Here](https://prometheus.io/docs/instrumenting/exporters/)

For applications, prometheus offers client libraries for the following languages:

* Python
* GO
* Java
* Ruby

While prometheus uses a Pull method to scrape metrics, there are few cases when we need to push metrics to Prometheus, for example, a Batch job, a backup script, in this case, we will use the push gateway.

##### Configuration: 

Promtheus reads its configuration from a yaml file called `prometheus.yaml`, which has 3 main sections:

* global: which has parameters such as `scrape_interval` and `evaluation_interval`
   
* rule-files: For aggregating metrics values or creating alerts when a condition is met
* scrape_configs: What resources prometheus will monitor, since prometheus exposes a `/metrics` endpoint, it can monitor itself, here we can define other jobs to monitor other components.

###### Alert Manager

Is the process responsible of sending alerts through a preconfigured channel (Email, Slack ...), when a specific condition is met, ex: when memory usage is higher then 60%.

When prometheus scrapes data, it stores it in a time series databes, it can be local or remote.

It offers a language `PromQL` to query the database, throught prometheus UI or other advanced tools like Grafana.

### Setup For Kubernetes

We will be using the official promtheus operator

Add the officiel prometheus repository

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Update the repository

```
helm repo update
```

Install the operator

```
helm install prometheus stable/prometheus-operator
```


