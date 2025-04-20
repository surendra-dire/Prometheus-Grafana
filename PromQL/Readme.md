1. Do we just need (in k8s) . kuse state metrics + node expoerter + app specifc exporter  ? any thing else 

 Why Use Both?
Using both Kube-State-Metrics and Node Exporter provides a comprehensive view of your Kubernetes environment:​
Spacelift
+1
Saifeddine Rajhi
+1

Kube-State-Metrics offers insights into the state and configuration of Kubernetes resources.​

Node Exporter provides data on the health and performance of the underlying infrastructure.​

Together, they enable you to monitor both the application layer (Kubernetes resources) and the infrastructure layer (nodes), facilitating proactive management and troubleshooting. 
================================
please tell me the name of metrices for  Cluster Level Monitoring,NameSpace Level Monitoring 
,Node Level Monitoring , SetUp Grafana Dashboard on Persistent Volume (PVC Disk) Level Monitoring 


=============================================================================================================
Best practises / challanges / hwo to fugure out most inposrtant metrics - approach (based on application logs ?)
1. not too much metrics calculation
2. less and meaningful level name
3. secured
4. push gateway - data cleaning, minimum use or find alternative like cloud watch have lambda logs
5. 

================================
**How to secure prometheus and grafana ?? how to give acccess to users ??**

Enable TLS Encryption
Implement Basic Authentication
Restrict Network Access
Securing Grafana
 Enable Authentication
Implement Role-Based Access Control (RBAC)
Assign Permissions to Dashboards and Folders

Managing User Access
. Create and Manage Users
Use Teams for Group Management
Audit User Activities

✅ Best Practices Summary
Prometheus: Use a reverse proxy with TLS and Basic Authentication to secure access.​

Grafana: Implement authentication methods and utilize RBAC for fine-grained access control.​
Grafana Labs

User Management: Cre

===============================
surendra.eu.pagerduty.com

key: ae9a83962aa64e09c1fce813383cd7a9

url: https://events.eu.pagerduty.com/generic/2010-04-15/create_event.json


=======================================================================================================================================
                                                 /etc/alertmanager/alertmanager.yml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.gmail.com:587'
  smtp_from: 'test.devops15@gmail.com'
  smtp_auth_username: 'test.devops15@gmail.com'
  smtp_auth_password: 'ipmc zgwv fbag pazp'  # Use Gmail App Password

route:
  group_by: ['alertname']
  group_wait: 5s
  group_interval: 1m
  repeat_interval: 3h
  receiver: 'default-receiver'

receivers:
  - name: 'default-receiver'
    email_configs:
      - to: 'test.devops15@gmail.com'
        send_resolved: true
    slack_configs:
      - channel: '#alertmanager'  # Name of your Slack channel
        send_resolved: true
        api_url: 'https://hooks.slack.com/services/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'

=============================================================================================================================================
                                                 /etc/prometheus/prometheus.yml
# my global config
global:
  scrape_interval: 5s # Set the scrape interval to every 5 seconds.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
           - 192.168.178.141:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "alert.rules.yml"

scrape_configs:

  #static_configs:
    #- targets: ["localhost:9090"]

  - job_name: 'flask_app'
    static_configs:
      - targets: ['192.168.178.203:5000']
        labels:
          env: dev
          app: flask-demo

    relabel_configs:
      - source_labels: [__address__]
        regex: '(.*):.*'
        target_label: instance
        replacement: '$1'
      - source_labels: [__address__]
        target_label: job
        replacement: 'flask_app'
=======================================================================================================================================
											/etc/prometheus/alert.rules.yml
groups:
  - name: flask_app_alerts
    rules:
      - alert: HighRequestCount
        expr: flask_requests_total > 5
        for: 5s
        labels:
          severity: critical
        annotations:
          summary: "High number of requests to Flask App"
          description: "The number of requests to flask app exceeded 5 in the last minute."

  - name: test-alert
    rules:
      - alert: HostDown
        expr: up == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"

=========================================================================================================================================



ipmc zgwv fbag pazp
how grafana - access control , queries on the fly ?


ipmc zgwv fbag pazp

webhook url :  https://hooks.slack.com/services/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

check message at slack
curl -X POST -H 'Content-type: application/json' --data '{"text":"Test message"}' 'https://hooks.slack.com/services/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'


is node-exortr installed by deaflt ????k8s


Prometheus offers various ways to identify the scrape targets to monitor.
1. Prometheus has native service discovery support where it identifies the scrape targets automatically without configuring the prometheous.yml file. For example once Prometheous is installed on K8S cluster then Prometheous can scrape the metrices of K8S resources such as pod, node, services, Endpoint, Ingress etc.

2.  Prometheus can identify the scrape targets by configuring the prpmetheous.yml file. Targets can be static or dynamic in nature
    1) Static targets: Config the targets. Targets do not change over time.
    2) Dynamic targets: Config the targets. Targets changes over time.


Prometheus does not include built-in authentication, authorization, or encryption features. Therefore, it's essential to implement additional security measures to protect your monitoring system effectively. Reverse Proxy for Authentication and Encryption or Implement Role-Based Access Control (RBAC):, Disable Unnecessary Endpoints: api/v1/status/flags, /api/v1/status/config, Restrict Network Access: firewall and network settings 

How nginx as a reverse proxy implemented for - Authentication and Encryption (https)

Did anyone experienced in the that they managed the targets in an external file (json or yaml)


Did anyone used file based service discovery in the Prometheus to find the targets ? 

How to create metrices from logs + traces , by default premethpus shows metrices ??????

Lab ` : Install - loki, premethpus, Grafana gaegar in k8s cluster

Just checked on chatgpt that these can used together
New Relic : for APM
Zabbix:  for network + infra
Nagios: for  network device monitoring  (Routers, switches, Load Balancers etc)

Categorize metrices : 

1. Hardware related
    1. CPU
    2. Memory
    3. Network - 
        1.  
        2.  
    4. Disk usage
    5. RAM
2. System level
3. Application level
     1. error rate
     2. app response time (login failure)
     3. app transaction rate (how many account are open)
     4. Security related - plugins
     5. 

how security of premetheous + Grafana managed ? 

Promethous installation preferred way : helm chart or binary or repo ??

Splunk itself can do what both Prometheus and Grafana can. So, if Splunk is in place, there is no need for Prometheus and Grafana. Also, Splunk can be a datasource for Grafana. ?

Approch for tool selelction ? 

scrape_interval: 30s ?? apprach disk usage -1 minute, cpu usage - 15-30 seconds ?

https://www.youtube.com/watch?v=Ke_Wr5zPE0A = tictactoc aproject


push gateway - https://devopscube.com/setup-prometheus-pushgateway-vm/


Challanges ? Shiva

splunk ? 

Key Metrics for Monitoring by Level
Here are the most common metrics monitored across different levels of IT infrastructure:
Hardware Level

CPU utilization percentage
Memory usage (total, available, used)
Disk space (total, available, used)
Disk I/O (read/write operations per second, latency)
Network throughput (bandwidth usage, packets per second)
Network latency/packet loss
Hardware temperature
Power consumption
Fan speed
Hardware errors (ECC memory errors, disk SMART data)

Operating System Level

System load average
Process count
Thread count
CPU context switches
CPU run queue length
System uptime
Kernel metrics (page faults, interrupts)
Swap usage
File descriptor usage
Network connections (established, TIME_WAIT, etc.)
System calls
I/O wait time

Application Level

Response time
Throughput (requests per second)
Error rate
Active sessions/connections
Queue length/backlog
Cache hit/miss ratio
Garbage collection metrics (for languages like Java)
Thread pool utilization
Database connections
Transaction rate
Custom business metrics
Health check status

Database Level

Query execution time
Transactions per second
Connection pool usage
Buffer cache hit ratio
Table/index size
Lock contention
Deadlocks
Query cache effectiveness
Replication lag
Index usage statistics

Network Level

Bandwidth utilization
Packet loss
Latency
Jitter
DNS resolution time
TCP retransmission rate
Connection establishment time
Routing table size
Interface errors

Container/Virtualization Level

Container/VM CPU usage
Container/VM memory usage
Container restarts
Pod/container health status
Image pull time
Container startup time
Storage provisioning metrics

User Experience Level

Page load time
Time to first byte (TTFB)
Time to interactive (TTI)
First contentful paint
Bounce rate
Error rates by user action
User session length
Feature usage metrics

Is there a specific level you'd like me to expand on with more detailed metrics?

Prometheus offers various ways to identify the scrape targets to monitor.
1. Prometheus has native service discovery support where it identifies the scrape targets automatically without configuring the prometheous.yml file. For example once Prometheous is installed on K8S cluster then Prometheous can scrape the metrices of K8S resources such as pod, node, services, Endpoint, Ingress etc.

2.  Prometheus can identify the scrape targets by configuring the prpmetheous.yml file. Targets can be static or dynamic in nature
    1) Static targets: Config the targets. Targets do not change over time.
    2) Dynamic targets: Config the targets. Targets changes over time.
	
	
Grafana alerts vs prometheous alerts ? show premethous alerts in grafana dashboard ? what details dashboard shows ? alerts + cuurrent system behaviour 

Federration ?

scalability  : multiple prometheous servers, remote storage on cloud s3, blob storage ? relabeling, inhibition ?

Efficint - prometheous [ cardanility, rebel few ? + multiple premetheous + federation ] + Alert manager [inhibition, silencing, grouping ] + Grafana [ ????? + storage of grafana]

How grafana + CloudWatch - work together  ?

Prometheus Logs vs Alertmanager Logs vs grafana logs 

prometheus irectory structure ? 

IDeally premethous , alertmanager and grafana isntalled togeter or not on same server  ?

scalability, reliability (failire of one shoudl not fail the other) , simplicity 


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

==================================================================================================================================================================================

Sceanrio 1:

1. A monitoring tool (I guess devops team was using splunk or other monitoring tool) picks up the error/warnings from the app logs.
2. then it triggers an alert 
3. The incident management tool ( for example - ServiceNow) receives the alert and automatically creates a ticket for the issue. details are added as much as possible.
4. The incident management tool, sends email with priority to devops team in a particular folder in the outlook. lets “Prod-issues” . Email contains link for incident with incident id and high level description.

Sceanrio 2: 

Send email and create jira ticket for pipeline failure.

Sceanrio 3: 
