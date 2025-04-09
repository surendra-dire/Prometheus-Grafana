
<h1 align="center" style="border-bottom: none">
    <a href="https://prometheus.io" target="_blank"><img alt="Prometheus" src="/images-icons/prometheus-logo.svg"></a><br>Prometheus
</h1>


Prometheus, a CNCF open source project, is a systems and service monitoring system. It collects metrics
from configured targets at given intervals, evaluates rule expressions,
displays the results, and can trigger alerts.


## Main Features

**Multi-dimensional data model**: Time-series data identified by metric name and key-value labels.  
**PromQL**: A powerful query language for flexible data analysis.  
**Autonomous servers**: Single server nodes with no distributed storage dependency.  
**HTTP pull model**: Time-series data is pulled from monitored targets via HTTP.  
**Push support**: Push time-series data via an intermediary gateway for batch jobs.  
**Service discovery**: Automatic target discovery or static configuration.  
**Graphing & dashboarding**: Integrates with tools like Grafana for visualization.  
**Federation**: Supports both hierarchical and horizontal federation for scalability.  

## Architecture overview

![Architecture overview](/images-icons/prometheous-architecture.jpg)

**Prometheus server**:   

   **Retrieval**: Pull based mechanism, actively pulls (scrapes) data from target endpoints over HTTP.   
   
   **TSDB**:  TSDB stores data as time series. A sequence of data points over time. Each time series is identified by a metric name and a set of key-value pairs (called labels), along with timestamps for each data point.  
   
   **HTTP server**: Enables query (metrices exposed by node exporter over http) and visualize metrics over http, integration with other tools, like Grafana and Alertmanager.  
    
   **Node (HDD/SSD)**: Prefers SSD for fast Read/Write Speed, high IOPS (Input/Output per Second) and low Latency.  
    
**Node exporter**:   
Node Exporter is a lightweight agent running on every node which exposes hardware and OS metrics over the HTTP protocol (End point -- > typically at http://<node-ip>:9100/metrics) in a format that Prometheus can scrape.  

**Alert Manager**:   
Alert Manager takes alerts from Prometheus, groups them, eliminates duplicates, and routes them to the appropriate notification channels.

* Grouping: Combines similar alerts to reduce notification noise.
* Inhibition: Suppresses alerts if others are already firing. 
* Silencing: Temporarily mutes specific alerts.  
* Sends notifications to various integrations based on defined rout.

## Install  

There are various ways of installing Prometheus.

### Precompiled binaries

Precompiled binaries for released versions are available in the
[*download* section](https://prometheus.io/download/)
on [prometheus.io](https://prometheus.io). Using the latest production release binary
is the recommended way of installing Prometheus.
See the [Installing](https://prometheus.io/docs/introduction/install/)
chapter in the documentation for all the details.

### Docker images

Docker images are available on [Quay.io](https://quay.io/repository/prometheus/prometheus) or [Docker Hub](https://hub.docker.com/r/prom/prometheus/).

You can launch a Prometheus container for trying it out with

```bash
docker run --name prometheus -d -p 127.0.0.1:9090:9090 prom/prometheus
```

Prometheus will now be reachable at <http://localhost:9090/>.

### Building from source

To build Prometheus from source code, You need:

* Go [version 1.22 or greater](https://golang.org/doc/install).
* NodeJS [version 22 or greater](https://nodejs.org/).
* npm [version 8 or greater](https://www.npmjs.com/).

Start by cloning the repository:

```bash
git clone https://github.com/prometheus/prometheus.git
cd prometheus
```

You can use the `go` tool to build and install the `prometheus`
and `promtool` binaries into your `GOPATH`:

```bash
GO111MODULE=on go install github.com/prometheus/prometheus/cmd/...
prometheus --config.file=your_config.yml
```

*However*, when using `go install` to build Prometheus, Prometheus will expect to be able to
read its web assets from local filesystem directories under `web/ui/static` and
`web/ui/templates`. In order for these assets to be found, you will have to run Prometheus
from the root of the cloned repository. Note also that these directories do not include the
React UI unless it has been built explicitly using `make assets` or `make build`.

An example of the above configuration file can be found [here.](https://github.com/prometheus/prometheus/blob/main/documentation/examples/prometheus.yml)

You can also build using `make build`, which will compile in the web assets so that
Prometheus can be run from anywhere:

```bash
make build
./prometheus --config.file=your_config.yml
```

The Makefile provides several targets:

* *build*: build the `prometheus` and `promtool` binaries (includes building and compiling in web assets)
* *test*: run the tests
* *test-short*: run the short tests
* *format*: format the source code
* *vet*: check the source code for common errors
* *assets*: build the React UI

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
