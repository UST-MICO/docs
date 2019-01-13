# Prometheus

To open Prometheus dashboard, enter the following command:
```bash
kubectl port-forward -n istio-system $(kubectl get pods --namespace istio-system --selector=app=prometheus --output=jsonpath="{.items..metadata.name}") 9090
```

## API

[HTTP API](https://prometheus.io/docs/prometheus/latest/querying/api/)

Request example:
```bash
curl -g 'http://localhost:9090/api/v1/query?query=container_cpu_load_average_10s{container_name="mico-core"}'
```
