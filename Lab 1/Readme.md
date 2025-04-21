# 💡Kubernetes Metrics

# Exporters
There are a variety of **exporters** available to monitor different aspects of a Kubernetes cluster and the applications deployed on it
| **Exporter**                      | **Area**                             | **Monitors**                         |  
|-----------------------------------|--------------------------------------|--------------------------------------|  
| **Kube State metrics**            |  K8s object state                    |Monitors the desired state of k8s objects|  
| **cAdvisor**                      | A powerful query language for flexible data analysis.|A powerful query language for flexible data analysis.|   
| **Node exporter**                 | Single server nodes with no distributed storage dependency|Single server nodes with no distributed storage dependency.|    


# 💡Metrics

## kube-state-metrics (KSM) 
Service that listens to the Kubernetes API server and generates metrics about the state of the resources (e.g., deployments, nodes, pods, etc.). It exposes them at an HTTP endpoint, usually on /metrics, for Prometheus to scrape.KSM focuses on cluster desired state.  

**Pods Level Metrices:**   
[Pod life cycle status]
| **POD Status** | **Metric** |
|----------------|--------------------------------------------------------------|
| **Running**    | `kube_pod_status_phase{phase="Running", namespace="production", pod="my-app-*"}` <br> or <br> `kube_pod_status_phase{phase="Running", namespace="production", label_app="my-app"}` |
| **Pending**    | `kube_pod_status_phase{phase="Pending", namespace="production", pod="my-app-*"}` |
| **Failed**     | `kube_pod_status_phase{phase="Failed", namespace="production", pod="my-app-*"}` |
| **Succeeded**  | `kube_pod_status_phase{phase="Succeeded", namespace="production", pod="my-app-*"}` |
| **Unknown**    | `kube_pod_status_phase{phase="Unknown", namespace="production", pod="my-app-*"}` |

[Pod Resource Usage]
| **POD Resources** | **Metric** |
|----------------|---------------------------------------------------------------------------------------------------------|
| **CPU usage**                | rate(container_cpu_usage_seconds_total{image!=""}[5m])                                    |
| **CPU resource limit**       | kube_pod_container_resource_limits_cpu_cores                                              |
| **Memory usage**             | sum(container_memory_usage_bytes{pod="your-pod-name", namespace="your-namespace"})        |
| **Memory resource limit**    |sum(kube_pod_container_resource_limits_memory_bytes{pod="nginx-pod", namespace="default"}) |

[Readiness/Liveness]
| **Readiness/Liveness**         | **Metric**                                                                                |
|--------------------------------|-------------------------------------------------------------------------------------------|
| **Readiness**                  | kube_pod_status_ready{namespace="default", pod="nginx-pod"}                               |
| **Liveness**                   | kube_pod_container_status_restarts_total                                              |












| Resource Misconfiguration                                               |                                                                                   |
| └─ No CPU Limit                                                         | `kube_pod_container_resource_limits_cpu_cores == 0`                               |
| └─ No Memory Request                                                    | `kube_pod_container_resource_requests_memory_bytes == 0`                          |
| Readiness Probe Failing                                                 | `kube_pod_container_status_ready{condition="true"} == 0`                          |
| Readiness Probe Failing                                                 |  kube_pod_container_status_ready{condition="true"}                                |


**Container Level Metrices:** 

| Issue                            | Metric/Alert Expression                                                                                         |
|----------------------------------------------------------|------------------------------------------------------------------------------------|
| Container lifecycle status [Waiting/Running/Terminated]  | sum(kube_pod_container_status_terminated{namespace="production", label_app="my-app", container!="", reason="OOMKilled"}) > 0 [OR] sum(kube_pod_container_status_terminated{namespace="production", pod=~"my-app-.*", container!="", reason="OOMKilled"}) > 0|  
| Resource Misconfiguration                                |                                                                                   |
| └─ No CPU Limit                                          | `kube_pod_container_resource_limits_cpu_cores == 0`                               |
| └─ No Memory Request                                     | `kube_pod_container_resource_requests_memory_bytes == 0`                          |
| Readiness Probe Failing                                  | `kube_pod_container_status_ready{condition="true"} == 0`                          |
| Readiness Probe Failing                                  |  kube_pod_container_status_ready{condition="true"}                                |

CrashLoop Detection: Detect containers with high restart count in the last 5 minutes  
increase(kube_pod_container_status_restarts_total[5m]) > 3  





------------------------------------
- alert: HighMemoryUsagePod
  expr: (sum(container_memory_usage_bytes{pod="nginx-pod", namespace="default"}) 
         / 
         sum(kube_pod_container_resource_limits_memory_bytes{pod="nginx-pod", namespace="default"})) > 0.8
  for: 2m
  labels:
    severity: critical
  annotations:
    summary: "Memory usage for pod nginx-pod is above 80% of its memory limit."
    description: "The pod nginx-pod in namespace default is using more than 80% of its memory limit for more than 2 minutes."
------------------------
- alert: HighCpuUsage
  expr: (rate(container_cpu_usage_seconds_total{image!=""}[5m]) / kube_pod_container_resource_requests_cpu_cores) > 0.8
  for: 2m
  labels:
    severity: warning
  annotations:
    summary: "High CPU usage detected"
    description: "Pod is using more than 80% of its requested CPU for over 2 minutes."
-----------------------------

- alert: PodNotReady
  expr: kube_pod_status_ready{namespace="default", pod="nginx-pod"} == 0
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "Pod nginx-pod in the default namespace is not ready for more than 5 minutes."
    description: "The pod nginx-pod is not ready and is not receiving traffic."

-------------
