
<h1 align="center" style="border-bottom: none">
    <a href="https://prometheus.io" target="_blank"><img alt="Prometheus" src="/images-icons/prometheus-logo.svg"></a><br>Prometheus
</h1>


Prometheus, a CNCF open source project, is a systems and service monitoring system. It collects metrics
from configured targets at given intervals, evaluates rule expressions,
displays the results, and can trigger alerts.


## 💡Main Features

| **Feature**                       | **Description**                                                                  |
|-----------------------------------|---------------------------------------------------------------------------------|
| **Multi-dimensional data model**  | Time-series data identified by metric name and key-value labels.                |
| **PromQL**                        | A powerful query language for flexible data analysis.                           |
| **Autonomous servers**            | Single server nodes with no distributed storage dependency.                     |
| **HTTP pull model**               | Time-series data is pulled from monitored targets via HTTP.                     |
| **Push support**                  | Push time-series data via an intermediary gateway for batch jobs.               |
| **Service discovery**             | Automatic target discovery or static configuration.                             |
| **Graphing & dashboarding**       | Integrates with tools like Grafana for visualization.                           |
| **Federation**                    | Supports both hierarchical and horizontal federation for scalability / multi-node architectures support          |



## 💡Architecture overview

![Architecture overview](/images-icons/prometheous-architecture.jpg)

**Prometheus server**:   

   **Retrieval**: This module handles the scraping of metrics from endpoints, which are discovered either through static configurations or dynamic service discovery methods (Pull based mechanism). 
   
   **TSDB**:  The data scraped from targets is stored in the TSDB, which is designed to handle high volumes of time-series data efficiently.TSDB stores data as time series which is a sequence of data points over time. Each time                        series is identified by a metric name and a set of key-value pairs (called labels), along with timestamps.  
   
   **HTTP server**: This provides an API for querying data using PromQL, retrieving metadata, and interacting with other components of the Prometheus ecosystem including Grafana and Alertmanager.  
   
   **Node (HDD/SSD) - Storage**: Prefers SSD for fast Read/Write Speed, high IOPS (Input/Output per Second) and low Latency.  
    
**Node exporter**:   
Node Exporter is a lightweight agent running on every node which exposes hardware and OS metrics over the HTTP protocol (End point -- > typically at http://<node-ip>:9100/metrics) in a format that Prometheus can scrape.  

**Alert Manager**:   
Alert Manager takes alerts from Prometheus, groups them, eliminates duplicates, and routes them to the appropriate notification channels such as PagerDuty, email, or Slack.

* Grouping: Combines similar alerts into a single notification to reduce alert noise.
* Inhibition: Hiding (or suppressing) some alerts if a more important related alert is already firing
* Silencing: Temporarily mutes specific alerts.  
* Sends notifications to various integrations based on defined rout.

**Service Discovery**  

Service discovery is used to identify and manage the list of scrape targets which are constantly being created and destroyed. This is crucial in the dynamic environments where services are constantly being created and destroyed.
For example, in K8S, Prometheus can automatically discover services, pods, and nodes using Kubernetes API, ensuring it monitors the most up-to-date list of targets.  
Dynamic service discovery mechanisms is configured within the prometheus.yml file.  

<pre style="color: orange;">
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - api_server: 'https://kubernetes.default.svc'
        role: pod
        namespaces:
          names:
            - default
            - kube-system
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_label_app]
        target_label: app
      - source_labels: [__meta_kubernetes_namespace]
        target_label: namespace
</pre>

**Exporters**    
Exporters are small applications that collect metrics from various third-party systems and expose them in a format Prometheus can scrape. They are essential for monitoring systems that do not natively support Prometheus (like Kubernetes, Etcd, Istio, NGINX, RabbitMQ, Grafana, PostgreSQL, etc.).
Common exporters include the Node Exporter (for os and hardware metrics - running on every node which exposes hardware and OS metrics over the HTTP protocol (End point -- > typically at http://<node-ip>:9100/metrics), the MySQL Exporter (for database metrics), and various other application-specific exporters.

**Pushgateway**    
The Pushgateway is used to expose metrics from short-lived jobs or services (such as batch jobs, cron jobs, Lambda functions, etc.) that cannot be scraped directly by Prometheus because they terminate quickly and may not be available during Prometheus's scrape intervals.  
These jobs push their metrics to the Pushgateway, where the data is temporarily stored. Prometheus then pulls the metrics from the Pushgateway server.    
![Alt text](/images-icons/pushgateway-2.jpeg)  
For monitoring short-lived jobs, these jobs/targets must push their metrics to the Pushgateway (via Prometheus client library), which exposes them via an HTTP endpoint for Prometheus to scrape. The Prometheus.yml file must be configured to allow the Pushgateway to scrape the metrics.  

<pre style="color: orange;">
scrape_configs:
  - job_name: 'pushgateway'
    honor_labels: true
    static_configs:
      - targets: ['localhost:9091']
</pre>
Note: Limitations ?   Why to avoid it ?   
**Prometheus Web UI**  
The Prometheus Web UI allows users to explore the collected metrics data, run ad-hoc PromQL queries, and visualize the results directly within Prometheus

**Grafana**  
Grafana is a powerful dashboard and visualization tool that integrates with Prometheus to provide rich, customizable visualizations of the metrics data.

**API Clients**  
Users are allowed to interact programmatically with Prometheus through its HTTP API to fetch data, query metrics, and integrate Prometheus with other applications like Grafana, Alertmanager, service meshes, API gateways, etc.

**Scalability and Backup/Restore**
Prometheus is a single-node system by design but federation is one way to implement a multi-node setup for scalability, data partitioning, or centralized views. Federation in Prometheus means that one Prometheus server scrapes
selected metrics from another Prometheus server. Prometheus supports multinode architecture.

For backup, either stop Prometheus server and copy data or take the snapshot without stoping the server.
```
systemctl stop prometheus
cp -r /var/lib/prometheus /path/to/backup/location
systemctl start Prometheus
[or]
Take the snapshot. In case cloud, take backup of the volume. 
```

