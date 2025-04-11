
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
| **Federation**                    | Supports both hierarchical and horizontal federation for scalability.           |


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

## 💡Installation

There are various ways of installing Prometheus : Precompiled binaries, Docker images, helm charts (for k8s) etc.

### Precompiled binaries
Download the Long-Term Support (LTS) binary, extract it, move the files to the /bin directory, and set up Prometheus to run as a service by creating the Prometheus user and updating the systemd service file. Precompiled binaries for released versions are available in the [*download* section](https://prometheus.io/download/) on [prometheus.io](https://prometheus.io).
 
```
# Extract (Go with current LTS version)
wget https://github.com/prometheus/prometheus/releases/download/v2.53.4/prometheus-2.53.4.linux-amd64.tar.gz
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

**alert.rules **
**alertmanager.yml**

**prometheus.rules (Recording Rules File)**

** prometheus.d**


























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
