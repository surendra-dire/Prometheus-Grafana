
<h1 align="center" style="border-bottom: none">
    <a href="https://prometheus.io" target="_blank"><img alt="Prometheus" src="/images-icons/prometheus-logo.svg"></a><br>Prometheus
</h1>


Prometheus, a CNCF open source project, is a systems and service monitoring system. It collects metrics
from configured targets at given intervals, evaluates rule expressions,
displays the results, and can trigger alerts.


## Main Features

| **Feature**                      | **Description**                                                                 |
|-----------------------------------|---------------------------------------------------------------------------------|
| **Multi-dimensional data model**  | Time-series data identified by metric name and key-value labels.                |
| **PromQL**                        | A powerful query language for flexible data analysis.                           |
| **Autonomous servers**            | Single server nodes with no distributed storage dependency.                     |
| **HTTP pull model**               | Time-series data is pulled from monitored targets via HTTP.                     |
| **Push support**                  | Push time-series data via an intermediary gateway for batch jobs.               |
| **Service discovery**             | Automatic target discovery or static configuration.                             |
| **Graphing & dashboarding**       | Integrates with tools like Grafana for visualization.                           |
| **Federation**                    | Supports both hierarchical and horizontal federation for scalability.          |


## Architecture overview

![Architecture overview](/images-icons/prometheous-architecture.jpg)

**Prometheus server**:   

   **Retrieval**: This module handles the scraping of metrics from endpoints, which are discovered either through static configurations or dynamic service discovery methods (Pull based mechanism). 
   
   **TSDB**:  The data scraped from targets is stored in the TSDB, which is designed to handle high volumes of time-series data efficiently.TSDB stores data as time series which is a sequence of data points over time. Each time                        series is identified by a metric name and a set of key-value pairs (called labels), along with timestamps.  
   
   **HTTP server**: This provides an API for querying data using PromQL, retrieving metadata, and interacting with other components of the Prometheus ecosystem including Grafana and Alertmanager.  
   
   **Node (HDD/SSD) - Storage**: Prefers SSD for fast Read/Write Speed, high IOPS (Input/Output per Second) and low Latency.  
    
**Node exporter**:   
Node Exporter is a lightweight agent running on every node which exposes hardware and OS metrics over the HTTP protocol (End point -- > typically at http://<node-ip>:9100/metrics) in a format that Prometheus can scrape.  

**Alert Manager**:   
Alert Manager takes alerts from Prometheus, groups them, eliminates duplicates, and routes them to the appropriate notification channels.

* Grouping: Combines similar alerts to reduce notification noise.
* Inhibition: Suppresses alerts if others are already firing. 
* Silencing: Temporarily mutes specific alerts.  
* Sends notifications to various integrations based on defined rout.

**Service Discovery**  
Service discovery is used to identify and manage the list of scrape targets which are constantly being created and destroyed. This is crucial in the dynamic environments where services are constantly being created and destroyed.
For example, in K8S, Prometheus can automatically discover services, pods, and nodes using Kubernetes API, ensuring it monitors the most up-to-date list of targets.  
Dynamic service discovery mechanisms is configured within the prometheus.yml file.  

**Pushgateway**    
The Pushgateway is used to expose metrics from short-lived jobs or applications that cannot be scraped directly by Prometheus due to the scrape interval settings. The short-lived jobs themselves push the metrics to the Pushgateway and will hold for a certain period. From the Pushgateway server, Prometheus will pull the metrics.  
It is particularly useful for batch jobs or tasks or services that have a limited lifespan and would otherwise not have their metrics collected. For example, AWS Lambda service.     
![Alt text](/images-icons/pushgateway-2.jpeg)  
For monitoring these short-lived jobs, targets must push their metrics to the Pushgateway, which exposes them via an HTTP endpoint for Prometheus to scrape. The Prometheus.yml file must be configured to allow the Pushgateway to scrape the metrics.  

**Exporters**    


**Alert Manager**  



**API Clients**  



**Premetheous targets**



**Service Discovery**


**Challanges**









































## Installation

There are various ways of installing Prometheus : Precompiled binaries, Docker images, helm charts (for k8s) etc.

### Precompiled binaries
Download the Long-Term Support (LTS) binary, extract it, move the files to the /bin directory, and set up Prometheus to run as a service by creating the Prometheus user and updating the systemd service file. Precompiled binaries for released versions are available in the [*download* section](https://prometheus.io/download/) on [prometheus.io](https://prometheus.io).
 
```
# Extract
tar xvfz prometheus-*.tar.gz
sudo mkdir /etc/prometheus /var/lib/prometheus
cd prometheus*

# Move file into directories
sudo mv prometheus promtool /usr/local/bin/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo mv consoles/ console_libraries/ /etc/prometheus/

# Configure the prometheous to run as service. Create user.
sudo useradd -rs /bin/false prometheus
sudo chown -R prometheus: /etc/prometheus /var/lib/prometheus

# Create prometheus service file
sudo vi /etc/systemd/system/prometheus.service 

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090 \
    --web.enable-lifecycle \
    --log.level=info

[Install]
WantedBy=multi-user.target

# Reload Daemon service and enable prometheus to start post reboot
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl status prometheus
sudo systemctl start prometheus

# Ensure port 9090 is open in firewall
```
### Docker images
docker run --name prometheus -d -p 9090:9090 prom/prometheus  
### Helm charts  
Helm is a package manager for Kubernetes applications, and Helm charts are the packages containing the manifest files. Helm simplifies the deployment and management of applications within a Kubernetes cluster.  To install Prometheus, the Kubernetes (K8s) cluster should be up and running, and Helm should be installed.  
#Install Helm  
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash  
helm version  

#Add the Prometheus Helm Chart Repository  
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts  
helm repo update  

#Install Promethous  
kubectl create namespace monitoring   
helm install prometheus prometheus-community/prometheus --namespace monitoring    

#Access Prometheus Web UI via port forwarding
kubectl port-forward svc/prometheus-server -n monitoring 9090:80

## Configuration files
**prometheous.yaml**:  

**Plugins.yaml**:  

## Configuration file

## Observability Basics

Monitoring focuses on tracking system health and performance through metrics and alerts, while Observability enables understanding of system behavior  by analyzing the data it produces, including logs, metrics, and traces to diagnose and investigate issues. Monitoring tells us what is happening, Logging explains why it is happening and Tracing shows how it is happening.
Monitoring is the *`when and what`* of a system error, and observability is the *`why and how`*

| Category       | Monitoring                                   | Observability                                         |
|----------------|----------------------------------------------|------------------------------------------------------|
| **Focus**          | Checking if everything is working as expected| Understanding why things are happening in the system  |
| **Data**           | Collects metrics like CPU usage, memory usage, and error rates | Collects logs, metrics, and traces to provide a full picture |
| **Alerts**         | Sends notifications when something goes wrong| Correlates events and anomalies to identify root causes |
| **Example**        | If a server's CPU usage goes above 90%, monitoring will alert us | If a website is slow, observability helps us trace the user's request through different services to find the bottleneck |
| **Insight**        | Identifies potential issues before they become critical  | Helps diagnose issues and understand system behavior |
| **Tools**          | Prometheus, Grafana, Nagios, Zabbix, PRTG  | ELK Stack (Elasticsearch, Logstash, Kibana), EFK Stack (Elasticsearch, FluentBit, Kibana) Splunk, Jaeger, Zipkin, New Relic, Dynatrace, Datadog |


## Monitoring scenarios
- Infrastructure: CPU usage, memory usage, disk I/O, network traffic.
- Applications: Response times, error rates, throughput.
- Databases: Query performance, connection pool usage, transaction rates.
- Network: Latency, packet loss, bandwidth usage.
- Security: Unauthorized access attempts, vulnerability scans, firewall logs.

## Observability scenarios
- Logs: Detailed records of events and transactions within the system.
- Metrics: Quantitative data points like CPU load, memory consumption, and request counts.
- Traces: Data that shows the flow of requests through various services and components.


## Tools comparison
The table below compares several observability tools widely used in monitoring and troubleshooting

| **Feature**                              | **New Relic**                                        | **ELK Stack (Elasticsearch, Logstash, Kibana)**       | **EFK Stack (Elasticsearch, FluentBit, Kibana)**     | **Dynatrace**                                        | **Datadog**                                          | **Prometheus + Grafana**                            |
|------------------------------------------|------------------------------------------------------|------------------------------------------------------|------------------------------------------------------|------------------------------------------------------|------------------------------------------------------|------------------------------------------------------|
| **Core Focus**                           | Full-stack observability (APM, metrics, logs, RUM)   | Log aggregation, analysis, and visualization         | Log aggregation, analysis, and visualization         | Full-stack observability (APM, infrastructure monitoring, logs) | Full-stack observability (APM, infrastructure monitoring, logs) | Metrics monitoring and visualization (Prometheus) + Dashboards (Grafana) |
| **Application Performance Monitoring (APM)** | Yes (Full APM, Distributed Tracing)                   | No                                                   | No                                                   | Yes (Full APM, Distributed Tracing)                   | Yes (Full APM, Distributed Tracing)                   | No (Requires additional tools like Jaeger)          |
| **Infrastructure Monitoring**            | Yes (Cloud, on-prem, containers, Kubernetes)          | No (Requires separate solution for infra monitoring)  | No (Requires separate solution for infra monitoring)  | Yes (Cloud, on-prem, containers, Kubernetes)          | Yes (Cloud, on-prem, containers, Kubernetes)          | Yes (Prometheus for infra metrics)                   |
| **Distributed Tracing**                  | Yes (Full-stack tracing)                             | No                                                   | No                                                   | Yes (Distributed Tracing)                             | Yes (Distributed Tracing)                             | No (Requires Jaeger or Zipkin integration)          |
| **Log Management**                       | Yes (Integrated log management and analysis)         | Yes (Log aggregation, search, and analysis)          | Yes (Log aggregation, search, and analysis)          | Yes (Integrated logs and metrics)                    | Yes (Integrated logs and metrics)                    | No (Requires external solutions like Loki)           |
| **Real User Monitoring (RUM)**           | Yes                                                  | No                                                   | No                                                   | No                                                   | Yes                                                  | No                                                   |
| **Metrics Monitoring**                   | Yes (Comprehensive metrics across infrastructure and application) | No                                                   | No                                                   | Yes (Comprehensive metrics across infrastructure and application) | Yes (Comprehensive metrics across infrastructure and application) | Yes (Prometheus metrics)                            |
| **Alerting**                             | Yes (Alert thresholds for metrics and logs)          | Yes (Alerting through Kibana or external tools)      | Yes (Alerting through Kibana or external tools)      | Yes (Alerting with AI-based anomaly detection)        | Yes (Alerting with AI-based anomaly detection)        | Yes (Alerting through Prometheus or external tools)  |
| **Dashboards & Visualizations**          | Yes (Custom dashboards with metrics, logs, traces)   | Yes (Kibana for log visualizations)                   | Yes (Kibana for log visualizations)                   | Yes (Custom dashboards, metrics, logs, and traces)   | Yes (Custom dashboards, metrics, logs, and traces)   | Yes (Grafana dashboards with Prometheus data)        |
| **Ease of Use**                          | Easy to deploy, cloud-based, minimal configuration   | Complex setup (requires management of Elasticsearch, Logstash, Kibana) | Slightly easier than ELK, but still requires configuration | Easy to deploy, cloud-based, minimal configuration   | Easy to deploy, cloud-based, minimal configuration   | Requires setup of Prometheus + Grafana, but can be more flexible |
| **Pricing**                              | Subscription-based, flexible pricing model           | Free (self-hosted), but infrastructure cost for scaling | Free (self-hosted), but infrastructure cost for scaling | Subscription-based, high cost for large deployments  | Subscription-based, flexible pricing model           | Free (Prometheus and Grafana), but infrastructure costs can add up |
| **Cloud Integration**                    | Yes (AWS, Azure, GCP, etc.)                          | Yes (via integrations with cloud platforms)          | Yes (via integrations with cloud platforms)          | Yes (AWS, Azure, GCP, etc.)                          | Yes (AWS, Azure, GCP, etc.)                          | Yes (Can integrate with cloud-native solutions)      |
| **AI and Machine Learning Insights**     | Yes (AI-powered insights, anomaly detection)         | No                                                   | No                                                   | Yes (AI-powered insights, anomaly detection)         | Yes (AI-powered insights, anomaly detection)         | No (Requires additional integrations)               |
| **Cost**                                 | Subscription-based, can be expensive at scale        | Free (open-source) but requires management and scaling | Free (open-source) but requires management and scaling | Expensive for large-scale use                        | Flexible pricing, but can get expensive at scale      | Free (Prometheus and Grafana), but infrastructure costs can add up |
| **Extensibility**                        | Limited extensibility for integration                | Highly extensible (can integrate with various tools) | Highly extensible (can integrate with various tools) | Highly extensible (integrates with many third-party tools) | Highly extensible (integrates with many third-party tools) | Highly extensible (via plugins, exporters, etc.)     |


 ## Observability Tools in Azure vs AWS

| **Feature**                              | **Azure**                                            | **AWS**                                              |
|------------------------------------------|------------------------------------------------------|------------------------------------------------------|
| **Full-Stack Observability**             | Azure Monitor + Azure Application Insights   | Amazon CloudWatch + AWS X-Ray                |
| **Application Performance Monitoring (APM)** | Azure Application Insights (Full APM)           | AWS X-Ray (Distributed Tracing)                  |
| **Infrastructure Monitoring**            | Azure Monitor                                       | Amazon CloudWatch (EC2, Lambda, RDS, etc.)       |
| **Log Management & Aggregation**         | Azure Log Analytics                            | Amazon OpenSearch Service + CloudWatch Logs  |
| **Real-Time Log Analysis**               | Azure Log Analytics                              | CloudWatch Logs Insights                         |
| **Distributed Tracing**                  | Azure Application Insights                       | AWS X-Ray                                        |
| **AI/Anomaly Detection**                 | Azure Monitor (AI Insights)                      | CloudWatch Anomaly Detection                     |
| **User Monitoring (RUM)**                | Azure Application Insights                       | CloudWatch RUM                                   |
| **Dashboards & Visualization**           | Azure Monitor Dashboards + Grafana           | AWS Managed Grafana + CloudWatch Dashboards  |
| **Alerting**                             | Azure Monitor Alerts                            | CloudWatch Alarms                                |
| **Security Monitoring**                  | Azure Sentinel                                  | Amazon GuardDuty + CloudTrail                |
| **Network Monitoring**                   | Azure Network Watcher                           | VPC Flow Logs + CloudWatch Network Monitoring |
| **Pricing**                              | Pay-as-you-go or subscription-based                  | Pay-as-you-go                                        |
| **Integration with Other Tools**         | Integrates with various Azure services           | Integrates with various AWS services             |


 ## Limitations on Prometheous & Grafana
- Limited APM (Application Performance Monitoring - metrics based) Features
- No Built-In Log Management
- Manual Instrumentation for Tracing
- Scalability Challenges with High Cardinality Metric
- Lack of Advanced AI and Anomaly Detection

 ## Paramters to select Observability Tool
- Full-stack observability (metrics, logs, and traces)
- Ease of use and setup (simple deployment)
- Scalability (ability to scale with your infrastructure)
- Real-time monitoring, alerting, and anomaly detection
- Unified data correlation (metrics, logs, and traces together)
- Customizability and extensibility (support for integrations)
- Cost efficiency (consider pricing models and scaling)
- Security and compliance features
- Support for distributed systems and microservices (e.g., Kubernetes)
- Strong vendor support, community, and documentation
- Integration with DevOps and CI/CD pipelines
- Historical data retention and trend analysis
- Avoidance of vendor lock-in (data portability)




























### Service discovery plugins

Prometheus is bundled with many service discovery plugins.
When building Prometheus from source, you can edit the [plugins.yml](./plugins.yml)
file to disable some service discoveries. The file is a yaml-formatted list of go
import path that will be built into the Prometheus binary.

After you have changed the file, you
need to run `make build` again.

If you are using another method to compile Prometheus, `make plugins` will
generate the plugins file accordingly.

If you add out-of-tree plugins, which we do not endorse at the moment,
additional steps might be needed to adjust the `go.mod` and `go.sum` files. As
always, be extra careful when loading third party code.

### Building the Docker image

You can build a docker image locally with the following commands:

```bash
make promu
promu crossbuild -p linux/amd64
make npm_licenses
make common-docker-amd64
```

The `make docker` target is intended only for use in our CI system and will not
produce a fully working image when run locally.

## Using Prometheus as a Go Library

### Remote Write

We are publishing our Remote Write protobuf independently at
[buf.build](https://buf.build/prometheus/prometheus/assets).

You can use that as a library:

```shell
go get buf.build/gen/go/prometheus/prometheus/protocolbuffers/go@latest
```

This is experimental.

### Prometheus code base

In order to comply with [go mod](https://go.dev/ref/mod#versions) rules,
Prometheus release number do not exactly match Go module releases.

For the
Prometheus v3.y.z releases, we are publishing equivalent v0.3y.z tags. The y in v0.3y.z is always padded to two digits, with a leading zero if needed.

Therefore, a user that would want to use Prometheus v3.0.0 as a library could do:

```shell
go get github.com/prometheus/prometheus@v0.300.0
```

For the
Prometheus v2.y.z releases, we published the equivalent v0.y.z tags.

Therefore, a user that would want to use Prometheus v2.35.0 as a library could do:

```shell
go get github.com/prometheus/prometheus@v0.35.0
```

This solution makes it clear that we might break our internal Go APIs between
minor user-facing releases, as [breaking changes are allowed in major version
zero](https://semver.org/#spec-item-4).

## React UI Development

For more information on building, running, and developing on the React-based UI, see the React app's [README.md](web/ui/README.md).

## More information

* Godoc documentation is available via [pkg.go.dev](https://pkg.go.dev/github.com/prometheus/prometheus). Due to peculiarities of Go Modules, v3.y.z will be displayed as v0.3y.z (the y in v0.3y.z is always padded to two digits, with a leading zero if needed), while v2.y.z will be displayed as v0.y.z.
* See the [Community page](https://prometheus.io/community) for how to reach the Prometheus developers and users on various communication channels.

## Contributing

Refer to [CONTRIBUTING.md](https://github.com/prometheus/prometheus/blob/main/CONTRIBUTING.md)

## License

Apache License 2.0, see [LICENSE](https://github.com/prometheus/prometheus/blob/main/LICENSE).

[hub]: https://hub.docker.com/r/prom/prometheus/
[quay]: https://quay.io/repository/prometheus/prometheus




Prometheus, a Cloud Native Computing Foundation project, is a monitoring system that gathers metrics from targets, evaluates rules, shows results, and triggers alerts when conditions are met.


# Installing as a binary
- Download prometheus here [Prometheus Download Page](https://prometheus.io/download/#prometheus)
- Extract
```
tar xvfz prometheus-*.tar.gz
```
- Create two new directories for Prometheus to use. The /etc/prometheus directory stores the Prometheus configuration files. The /var/lib/prometheus directory holds application data.
```
sudo mkdir /etc/prometheus /var/lib/prometheus
```
- Move to extracted directory
```
cd prometheus*
```

```
sudo mv prometheus promtool /usr/local/bin/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo mv consoles/ console_libraries/ /etc/prometheus/
```
- Configure prometheus to run as a service
```
sudo useradd -rs /bin/false prometheus
sudo chown -R prometheus: /etc/prometheus /var/lib/prometheus
```
- Create a service file
```
sudo vi /etc/systemd/system/prometheus.service
```
- Write below data to service file
```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --web.listen-address=0.0.0.0:9090 \
    --web.enable-lifecycle \
    --log.level=info

[Install]
WantedBy=multi-user.target
```
- Reload Daemon service and enable prometheus to start post reboot
```
sudo systemctl daemon-reload
sudo systemctl enable prometheus
```

# Managing Prometheus Service
Use below commands to check status, start or stop Prometheus service.
```
sudo systemctl status prometheus
```
```
sudo systemctl start prometheus
```
```
sudo systemctl stop prometheus
```
# Prometheus Configuration File Location
Prometheus YML file is located at following `/etc/prometheus/prometheus.yml`
Here is what a default configuration file will look like.

```
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'codelab-monitor'

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first.rules"
  # - "second.rules"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9090']
```

# Access Prometheus
You can now login to `http://IP_Address:9090` to access Prometheus GUI.
If you can't access the page it can be either because prometheus service is not started or port is not allowed from outside world.

# Commands to open firewall ports on centos 7
- Check ports which are allowed
```
sudo firewall-cmd --zone=public --list-ports
```
- Allow 9090 port
```
sudo firewall-cmd --zone=public --add-port=9090/tcp --permanent
sudo firewall-cmd --reload
```
