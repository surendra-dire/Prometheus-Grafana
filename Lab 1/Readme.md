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
A service that listens to the Kubernetes API server and exports Kubernetes cluster state metrics.

**Pods Level Metrices:**   
[Pod life cycle status]
| **POD Status** | **Metric** |
|----------------|--------------------------------------------------------------|
| **Running**    | `kube_pod_status_phase{phase="Running", namespace="production", pod="my-app-*"}` <br> or <br> `kube_pod_status_phase{phase="Running", namespace="production", label_app="my-app"}` |
| **Pending**    | `kube_pod_status_phase{phase="Pending", namespace="production", pod="my-app-*"}` |
| **Failed**     | `kube_pod_status_phase{phase="Failed", namespace="production", pod="my-app-*"}` |
| **Succeeded**  | `kube_pod_status_phase{phase="Succeeded", namespace="production", pod="my-app-*"}` |
| **Unknown**    | `kube_pod_status_phase{phase="Unknown", namespace="production", pod="my-app-*"}` |

[Readiness/Liveness]  
The readiness probe answers 'Can the pod serve traffic?' while the liveness probe answers 'Is the pod alive, or should it be restarted?
| **Readiness/Liveness**         | **Metric**                                                                                |
|--------------------------------|-------------------------------------------------------------------------------------------|
| **Readiness Prob Failing**     | `kube_pod_status_ready{namespace="default", pod="nginx-pod", condition="true"} == 0 `     |
| **Liveness Probe falling**     | `rate(kube_pod_container_status_restarts_total{namespace="default", pod="nginx-pod"}[5m]) [OR] kube_pod_container_status_waiting_reason{namespace="default", pod="nginx-pod", reason="CrashLoopBackOff"}`|

## cAdvisor metrics
[Resource Usage]  
[CPU]
| **POD Status** | **Metric** |
|----------------|--------------------------------------------------------------|
| **CPU usage (container)**             |`rate(container_cpu_usage_seconds_total{image!=""}[5m])` |
| **CPU resource limit(container)**     | `kube_pod_container_resource_limits_cpu_cores{namespace="production", pod=~"my-app-.*"}` |
| **CPU usage (pod)**                   |`sum by (pod) (rate(container_cpu_usage_seconds_total{image!=""}[5m]))` |
| **CPU resource limit(pod)**           | `sum by (pod) (kube_pod_container_resource_limits_cpu_cores{namespace="production", pod=~"my-app-.*"})` |
| **CPU no resource limit(pod)**        | `kube_pod_container_resource_limits_cpu_cores == 0` |
| **CPU usage (Node)**                  |`sum by (node) (rate(container_cpu_usage_seconds_total{image!=""}[5m]))` |
| **CPU resource limit(Node)**          | `sum by (node) (kube_pod_container_resource_limits_cpu_cores)` |

[Memory]  
| **POD Status**                            | **Metric** |
|-------------------------------------------|--------------------------------------------------------------|
| **Memory usage (container)**              | `rate(container_memory_usage_bytes{image!=""}[5m])` |
| **Memory resource limit (container)**     | `kube_pod_container_resource_limits_memory_bytes{namespace="production", pod=~"my-app-.*"}` |
| **Memory usage (pod)**                    | `sum by (pod) (rate(container_memory_usage_bytes{image!=""}[5m]))` |
| **Memory resource limit (pod)**           | `sum by (pod) (kube_pod_container_resource_limits_memory_bytes{namespace="production", pod=~"my-app-.*"})` |
| **Memory no resource limit (pod)**        | `kube_pod_container_resource_limits_memory_bytes == 0` |
| **Memory usage (Node)**                   | `sum by (node) (rate(container_memory_usage_bytes{image!=""}[5m]))` |
| **Memory resource limit (Node)**          | `sum by (node) (kube_pod_container_resource_limits_memory_bytes)` |














 ` ` `
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




```
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

-------------  ` ` `
```
