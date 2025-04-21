# 💡K8s monitoring project

# Exporters to monitor K8s cluster
There are variaty of exporters to monitor the different aspect of the k8s clusters and the application deployed on it. Few of them are:

## 💡Main Features  

| **Exporter**                      | **Area**                             | **Monitors**                         |  
|-----------------------------------|--------------------------------------|--------------------------------------|  
| **Kube State metrics**            |  K8s object state                    |Monitors the desired state of k8s objects|  
| **cAdvisor**                      | A powerful query language for flexible data analysis.|A powerful query language for flexible data analysis.|   
| **Node exporter**                 | Single server nodes with no distributed storage dependency|Single server nodes with no distributed storage dependency.|    




**kube-state-metrics (KSM)**  
Service that listens to the Kubernetes API server and generates metrics about the state of the resources (e.g., deployments, nodes, pods, etc.). It exposes them at an HTTP endpoint, usually on /metrics, for Prometheus to scrape.KSM focuses on cluster desired state.  
**Pods**  
CrashLoop Detection: Detect containers with high restart count in the last 5 minutes  
increase(kube_pod_container_status_restarts_total[5m]) > 3  

Stuck Pods (Pending Too Long):  
kube_pod_status_phase{phase="Pending"} == 1  

Resource Misconfiguration:   

No CPU Limit  
kube_pod_container_resource_limits_cpu_cores == 0  

No Memory Request:  
kube_pod_container_resource_requests_memory_bytes == 0  

Readiness Probe Failing:  
kube_pod_container_status_ready{condition="true"} == 0   


resource_requests:  

resource_limits:  
