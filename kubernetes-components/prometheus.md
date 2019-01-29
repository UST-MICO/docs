# Prometheus

## Installation

We use the YAML files of the GitHub repository [giantswarm/prometheus](https://github.com/giantswarm/prometheus) for the installation of Prometheus and Grafana.

Get yaml file:
```bash
wget https://raw.githubusercontent.com/giantswarm/kubernetes-prometheus/master/manifests-all.yaml -O install/kubernetes/monitoring.yaml
```

Install it:
```bash
kubectl apply -f install/kubernetes/monitoring.yaml
```

## Dashboard

To open Prometheus dashboard, enter the following command:
```bash
kubectl port-forward -n monitoring $(kubectl get pods --namespace monitoring --selector="app=prometheus,component=core" --output=jsonpath="{.items..metadata.name}") 9090:9090
```

To open Grafana dashboard, enter the following command:
```bash
kubectl port-forward -n monitoring $(kubectl get pods --namespace monitoring --selector="app=grafana,component=core" --output=jsonpath="{.items..metadata.name}") 3000:3000
```

Login to Grafana:
* username: admin
* password: admin

Currently installed dashboards:
* [Kubernetes Cluster Monitoring](http://localhost:3000/dashboard/db/kubernetes-cluster-monitoring-via-prometheus)
* [Kubernetes Pod Resources](http://localhost:3000/dashboard/db/kubernetes-pod-resources)
* [Prometheus Stats](http://localhost:3000/dashboard/db/prometheus-stats)

## API

[Prometheus HTTP API Documentation](https://prometheus.io/docs/prometheus/latest/querying/api/)

**Example 1:**
```bash
curl -g 'http://localhost:9090/api/v1/query?query=container_cpu_load_average_10s{container_name="mico-core"}'
```

**Example 2:**
```bash
curl -g 'http://localhost:9090/api/v1/query?query=container_memory_working_set_bytes{pod_name="POD_NAME"}'
```
This query uses the metric `container_memory_working_set_bytes`. This metric provides us
with the memory useage. It is limited to the memory useage of the pod with a spezified name. 
Prometheus provides 3 different values for this request. The total memory useage of the pod, the 
memory useage of the container running in the pod and the difference. 

**Example 3:**
```bash
curl -g 'http://localhost:9090/api/v1/query?query=sum(container_memory_working_set_bytes{pod_name="POD_NAME",container_name=""})'
```
This query adds the requirement `container_name=""`. This filters the three values from example 2. Only the total memory usage value of the pod has an empty container name (because it is a whole pod, not just the container inside). The sum function removes metadata provided together with the metric itself.

## Further information

- [Metrics](https://github.com/google/cadvisor/blob/master/docs/storage/prometheus.md#prometheus-metrics)
- [Prometheus Doc](https://prometheus.io/docs/prometheus/latest/querying/examples/)
