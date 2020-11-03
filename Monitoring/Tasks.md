#### Book

Chapter 1: Introduction to Prometheus: (Done)

* Architecture
* Client libraries
* Exporters
* Service discovery
* Scraping 
* Storage
* Dashboards
* Recording rules & alerts
* Alert Management
* Long-term storage 

Chapter 2: Getting started with Prometheus (Done)

* Running prometheus
* Using the expression browser (rate function, operators (==, /, -, +))
* Running the node exporter
* Alerting

Chapter 3: Instrumentation

* Python program
* The counter
* The gauge
* The summary
* The histogram
* Unit testing instrumentation
* Approaching instrumentation

...

Chapter 5: Labels



Chapter 7: Node exporter (CPU, Memory, Disk, Network ...)

* CPU collector:

To calculate the average of each state of the cpu for all CPUs:
avg without(cpu, mode)(rate(node_cpu_seconds_total{mode="idle"}[1m]))

To calculate the cpu load, we will use the idle time (when the cpu has nothing to perform), (100% - idle time)
cpu load by machine: '100 - (avg by (instance) (irate(node_cpu_seconds_total{job="node",mode="idle"})) * 100)'

* Filesystem collector (Disk Usage)

Free disk space:
node_filesystem_avail_bytes / node_filesystem_size_bytes

Disk Usage of /:
100 - ((node_filesystem_avail_bytes{mountpoint="/",fstype!="rootfs"} * 100) / node_filesystem_size_bytes{mountpoint="/",fstype!="rootfs"})

* Diskstats collector (Disk Inout Output Stats)

Disk I/O utilization:
rate(node_disk_io_time_seconds_total[1m])

average time for read I/O:
rate(node_disk_io_time_seconds_total[1m])

Disk IOps Completed:
irate(node_disk_reads_completed_total{device="sda"}[1h])

Disk R/W Data:
irate(node_disk_read_bytes_total[5m])
irate(node_disk_written_bytes_total[5m])

* Netdev collector (Network Device Stats)

Number of packets received of a specific interface:
rate(node_network_receive_bytes_total{device="wlp2s0"}[1m]) 

Network traffic by packets:
irate(node_network_receive_packets_total[5m]) 
irate(node_network_transmit_packets_total[5m])

Network Traffic Errors:
irate(node_network_receive_errs_total{instance=~"$node:$port",job=~"$job"}[5m])
irate(node_network_transmit_errs_total[5m])

Network Traffic Drop:
irate(node_network_receive_drop_total[5m])
irate(node_network_transmit_drop_total[5m])

* Meminfo collector

Used RAM Memory:
100 - ((node_memory_MemAvailable_bytes * 100) / node_memory_MemTotal_bytes)

Swap:
((node_memory_SwapTotal_bytes - node_memory_SwapFree_bytes) / (node_memory_SwapTotal_bytes )) * 100

* HWmon collector (Hardware Monitor, used in bare metal env)

Labels of each cpu core:
node_hwmon_sensor_label

Temperature of each component: (use labels ti retrieve specific core)
node_hwmon_temp_celsius

* Stat collector

Uptime:
time() - node_boot_time_seconds

* Uname collector
* Loadavg collector
* Textfile collector

...

Chapter 9: Containers and kubernetes

* cAdvisor: Exporter that provides metrics about cgroups

Kubernetes:

Service discovery:

* nodes
* endpoints
* service
* Pod
* Ingress




