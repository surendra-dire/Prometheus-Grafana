# 💡Observability
Observability is the ability to understand the internal state of a system by analyzing the data it produces, including logs, metrics, and traces. It tells "**Why** it is happening".  

**logs**: Involves the collection of log data from various components of a system. Logging explains why it is happening.  
**metrics**: Involves tracking system metrics like CPU usage, memory usage, and network performance and send alerts based on predefined thresholds and conditions. Main focus is "What is happening".  
**Traces**: Involves tracking the flow of a request or transaction as it moves through different services and components within a system. Tracing shows "how it is happening".  



# 💡Monitoring
Monitoring focuses on tracking system's health and performance through metrics (like CPU usage, memory usage, and network performance) and raise alerts when something goes wrong. It tells " **What** is happening". It is a part of observability.

# 💡Monitoring vs Observability 

| Category       | Monitoring                                   | Observability                                         |
|----------------|----------------------------------------------|------------------------------------------------------|
| **Focus**          | Checking if everything is working as expected else raising alerts| Understanding why things are happening in the system, Identifying the root cause |
| **Data**           | Collects metrics like CPU usage, memory usage, and error rates | Collects logs, metrics, and traces to provide a full picture |
| **Example**        | If a server's CPU usage goes above 90%, monitoring will alert us | If a website is slow, observability helps us trace the user's request through different services to find the bottleneck |
| **Benefit**        | Identifies potential issues before they become critical  | Helps diagnose issues and understand system behavior |
| **Tools**          | Prometheus, Grafana, Nagios, Zabbix, PRTG  | ELK Stack (Elasticsearch, Logstash, Kibana), EFK Stack (Elasticsearch, FluentBit, Kibana) Splunk, Jaeger, Zipkin, New Relic, Dynatrace, Datadog |

# 💡Monitoring and Observability Scenarios

## Monitoring scenarios
- **Infrastructure**: CPU usage, memory usage, disk I/O, network traffic.
- **Applications**: Response times, error rates, throughput.
- **Databases**: Query performance, connection pool usage, transaction rates.
- **Network**: Latency, packet loss, bandwidth usage.
- **Security**: Unauthorized access attempts, vulnerability scans, firewall logs.

## Observability scenarios
- **Logs**: Detailed records of events and transactions within the system.
- **Metrics**: Quantitative data points like CPU load, memory consumption, and request counts.
- **Traces**: Data that shows the flow of requests through various services and components.


## 💡Tools comparison
The table below compares several observability tools widely used in monitoring and observability

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


 ## 💡Observability Tools in Azure & AWS cloud

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


 ## 💡Limitations on Prometheous & Grafana
- Limited APM (Application Performance Monitoring - metrics based) Features
- No Built-In Log Management
- Manual Instrumentation for Tracing
- Scalability Challenges with High Cardinality Metric
- Lack of Advanced AI and Anomaly Detection

 ## 💡Paramters to select Observability Tool
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

