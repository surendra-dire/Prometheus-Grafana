<h1 align="center" style="border-bottom: none">
    <a href="https://prometheus.io" target="_blank"><img alt="Prometheus" src="/images-icons/prometheus-logo.svg"></a><br>Prometheus
</h1>


## 💡Installation - Prometheus [ubuntu]

There are various ways of installing Prometheus : Precompiled binaries, Docker images, helm charts (for k8s) etc.

### Precompiled binaries
Download the Long-Term Support (LTS) binary, extract it, move the files to the /bin directory, and set up Prometheus to run as a service by creating the Prometheus user and updating the systemd service file. Precompiled binaries for released versions are available in the [*download* section](https://prometheus.io/download/) on [prometheus.io](https://prometheus.io).
 
```
# update package list
sudo apt update
# Extract (Go with current LTS version)

wget https://github.com/prometheus/prometheus/releases/download/v2.53.4/prometheus-2.53.4.linux-amd64.tar.gz
tar xvfz prometheus-*.tar.gz
sudo mkdir /etc/prometheus /var/lib/prometheus
cd prometheus-2.53.4.linux-amd64

# Move file into directories
sudo mv prometheus promtool /usr/local/bin/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo mv consoles/ console_libraries/ /etc/prometheus/
```
```
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

**Prometheus**

```
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
```
**Grafana**
```
#Install Helm  
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash  
helm version  

#Add the Grafana Helm Chart Repository  
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

#Install Grafana  
kubectl create namespace monitoring   
helm install grafana grafana/grafana --namespace monitoring

# Retrieve the Admin Password
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

#Access Prometheus Web UI via port forwarding or expose prometheous server on nodeport service
kubectl port-forward --namespace monitoring svc/grafana 3000:80
                [or]

```
## 💡Installation - Alertmanager [ubuntu]
```
# update package list
sudo apt update

# Extract (Go with current LTS version)
wget https://github.com/prometheus/alertmanager/releases/download/v0.28.1/alertmanager-0.28.1.linux-amd64.tar.gz
tar -xvzf alertmanager-0.28.1.linux-amd64

# Move alermanager binary & amtool command-line utility in bin folder
sudo mv alertmanager-0.28.1.linux-amd64/alertmanager alertmanager-0.28.1.linux-amd64/amtool  /usr/local/bin/

# Create Alertmanager folder to store its internal state (Silences, Notification log & Other runtime data)
sudo mkdir -p /var/lib/alertmanager
```
```
[alertmanager.yml]

# The main components in this file are: global (inherited by receivers), route (how alerts are grouped, routed, and sent),
receivers (destinations like email, Slack, PagerDuty, etc.), inhibit_rules (suppress alerts under certain conditions to avoid noise),
and templates (custom formatting of alert notifications).

sudo mkdir -p /etc/alertmanager
sudo nano /etc/alertmanager/alertmanager.yml

# Add the configuration details - gmail, slack & pagerduty.

global:
  resolve_timeout: 5m                            # Alertmanager will wait for 5 minutes to confirm the situation is stable before sending a notification that the issue is resolved.
  smtp_smarthost: 'smtp.gmail.com:587'           # Gmail SMTP server and port
  smtp_from: 'email@gmail.com'                   # Email address from which the alerts will be sent
  smtp_auth_username: 'email@gmail.com'          # username for authentication with the SMTP server
  smtp_auth_password: 'ipmc abc sedg ijkl'       # password for the Gmail account (app password)

route:
  group_by: ['alertname']                        # group alerts by their alertname label. Alerts with the same alertname will be grouped together in a notification.
  group_wait: 30s                                # Alertmanager will wait for 30 seconds before sending the notification
  group_interval: 5m                             # Minimum amount of time between two notifications for the same group of alerts
  repeat_interval: 3h                            # If an alert is still firing after the first notification, it will be repeated every 1 hour
  receiver: 'default-receiver'                   # receiver for the alerts

receivers:
  - name: 'default-receiver'

   email_configs:
      - to: 'email@gmail.com'                    # Email address where the alert notifications will be sent
        send_resolved: true                      # Notification will be sent when an alert is resolved

   slack_configs:
      - channel: '#alertmanager'                                                              # Name of your Slack channel
        send_resolved: true
        api_url: 'https://hooks.slack.com/services/T05J4A05EHM/B0SDSRN7LJ/8PxXwee8DdferFG'    # Slack Incoming Webhook URL

   pagerduty_configs:
      - routing_key: 'ae9a8ffffdf4eredsddsdsd3383cd7a9'                                       # Integration key- update it
        severity: 'error'
        send_resolved: true

# Verify alertmanager config file
alertmanager --config.file=/etc/alertmanager/alertmanager.yml

# Create alertmanager user
sudo useradd --no-create-home --system --shell /bin/false alertmanager

sudo chown -R alertmanager:alertmanager /etc/alertmanager
sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager

# Create service file alertmanager & add the following configuration
sudo nano /etc/systemd/system/alertmanager.service

[[Unit]
Description=Alertmanager
After=network.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager
Restart=always

[Install]
WantedBy=multi-user.target

# Connect Prometheus to Alertmanager - Add target for alertmanager -update prometheus.yml. Restart Prometheus.
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - 'localhost:9093'

# Reload Daemon service and enable alartmanager to start post reboot
sudo systemctl daemon-reload
sudo systemctl enable alertmanager
sudo systemctl start alertmanager
sudo systemctl status alertmanager

# Acess alert manager at port 9093. Ensure port 9093 is open in firewall.
http://<your-server-ip>:9093
```
## Configuration files

**prometheous.yml**:  
This is the primary configuration file specifying how and where to collect metrics, including scrape intervals, targets, and job settings. It can also reference external files for alerting and recording rules, helping to keep the setup more modular and maintainable.
``` 
# ===============================
# Global settings
# ===============================
global:
  scrape_interval: 15s        # How often to scrape targets (default is 60s)
  evaluation_interval: 15s   # How often to evaluate alert rules (default is 60s)
  scrape_timeout: 10s        # Timeout for scraping a target (default is 10s)

# ===============================
# Alertmanager config (optional)
# ===============================
alerting:
  alertmanagers:
    - static_configs:
        - targets: []        # Example: ['localhost:9093'] if you have Alertmanager running

# ===============================
# Rule files (optional)
# ===============================
rule_files:
  # - "alert_rules.yml"      # Load alert rules from this file

# ===============================
# Scrape configs (who to scrape)
# ===============================
scrape_configs:

  # ---- Prometheus itself ----
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # ---- Example: Node Exporter on all nodes ----
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['192.168.1.10:9100', '192.168.1.11:9100']

  # ---- Example: Custom Flask App with metrics ----
  - job_name: 'flask_app'
    static_configs:
      - targets: ['192.168.1.100:5000']
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
```
**alerts.rout.yml**:  
It defines routing rules for how alerts are grouped and sent to different receivers (/etc/prometheus/alert.rules.yml)
```
groups:
  - name: flask_app_alerts
    rules:
      - alert: HighRequestCount
        expr: flask_requests_total > 100
        for: 5s
        labels:
          severity: critical
        annotations:
          summary: "High number of requests to Flask App"
          description: "The number of requests to flask app exceeded 100 in the last minute."

  - name: test-alert
    rules:
      - alert: HostDown
        expr: up == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "Application is down."
  ```
 ```
# Check and apply new config
prometheus --config.file=/etc/prometheus/prometheus.yml
curl -X POST http://<prometheus-server>:9090/-/reload
[OR]
sudo systemctl restart prometheus
 ```

 ```
**recording_rules.yml**  
A recording rule lets you precompute and store the result of a PromQL expression ( complex or frequently-used) so Prometheus
doesn’t have to recalculate it over and over again. This improves performance and makes alerts and dashboards faster and more
efficient.
Here is an example -to check how many total requests per second over the last minute flask app is handling.

groups:
  - name: flask_app_recording_rules
    rules:
      - record: flask_app:requests_per_minute                 # name of the new saved metric
        expr: sum(rate(flask_requests_total[1m]))             # PromQL expression you want to save the result of
        labels:
          app: flask_app                                      # optional extra labels to tag the new metric
 ```

 ```
# Check and apply new config
prometheus --config.file=/etc/prometheus/prometheus.yml
curl -X POST http://<prometheus-server>:9090/-/reload
[OR]
sudo systemctl restart prometheus
 ```

  ```
flask_app:requests_per_minute :
With recording rule, you can just use this alert in alertmanager, prometheus and grafana. It comes with level "app="flask_app".
Update the alerts.route.yml with below:  
 ```
 ```
groups:  
  - name: flask_app_alerts (HighRequestRate)
    rules:  
      - alert: HighRequestRate  
        expr: flask_app:requests_per_minute > 10  
        for: 30s  
        labels:  
          severity: warning  
        annotations:  
          summary: "High request rate on Flask App"  
          description: "The request rate has been above 10 rps for more than 30 seconds."  
 ```
Verify the alert is shown in alertmanager.  

Note:  **Before** (without recording rule):   sum(rate(flask_requests_total[1m]))   **After**:  flask_app:requests_per_minute  



 ```
Note : Prometheous config file folder structure for the reference

prometheus-project/
├── prometheus.yml                   # Prometheus main config
├── alertmanager/
│   └── alertmanager.yml            # Alertmanager config
├── rules/
│   ├── alerts.route.yml            # Your alerting rules
│   └── recording.rules.yml         # Your recording rules
├── data/                            # (Optional) Prometheus storage location
└── docker-compose.yml              # (Optional) to run with Docker
 ```
