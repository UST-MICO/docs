# Istio

## Version

As of time of writing, Istio 1.0.5 ist the latest stable release.
Istio 1.1 is available as a preliminary snapshot release. A first release candidate is expected later in January and a final build sometime in February ([Status of Istio 1.1](https://discuss.istio.io/t/status-of-istio-1-1/33)).

## Installation

**[Prerequisites](https://preliminary.istio.io/docs/setup/kubernetes/helm-install/#prerequisites):**

1. [Download the Istio release](/docs/setup/kubernetes/download-release/).

2. Perform any necessary [platform-specific setup](/docs/setup/kubernetes/platform-setup/).

3. Check the [Requirements for Pods and Services](/docs/setup/kubernetes/spec-requirements/) on Pods and Services.

4. [Install the Helm client](https://docs.helm.sh/using_helm/).

5. Istio by default uses `LoadBalancer` service object types.  Some platforms do not support `LoadBalancer`
   service objects.  For platforms lacking `LoadBalancer` support, install Istio with `NodePort` support
   instead with the flags `--set gateways.istio-ingressgateway.type=NodePort --set gateways.istio-egressgateway.type=NodePort`
   appended to the end of the Helm operation.


### Custom installation - Telemetry

To begin with we want to have a minimal Istio installation that only has its telemetry features enabled.

Required components:
* Envoy + automatic sidecar injection
  * Reporting of telemetry (logs + metrics)
* Mixer
  * Collect telemetry data
  * Provides interfaces ([adapters](https://istio.io/docs/reference/config/policy-and-telemetry/adapters/)) for specific infrastructure backends that support predefined [templates](https://istio.io/docs/reference/config/policy-and-telemetry/templates/).
    Examples: Prometheus (`Metric`), Fluentd (`Log Entry`), Stackdriver (`Log Entry`, `Metric`, `Trace Span`), ...
* Prometheus
  * Collect metrics
* Fluentd
  * Collect logs

We use [Helm](./helm.md) for templating only.

Helm parameters:
* `security.enabled=false`: Citadel won't be installed
* `ingress.enabled=false`: Ingress won't be installed
* `gateways.istio-ingressgateway.enabled=false`: Ingress gateway won't be installed
* `gateways.istio-egressgateway.enabled=false`: Egress gateway won't be installed
* `galley.enabled=false`: Galley won't be installed
* `sidecarInjectorWebhook.enabled=true`: Automatic sidecar-injector will be installed (default)
* `mixer.policy.enabled=false`: Mixer Policy won't be installed
* `mixer.telemetry.enabled=true`: Mixer Telemetry will be installed (default)
* `global.proxy.envoyStatsd.enabled=false`: Disable Statsd (default)
* `pilot.sidecar=false`: ?
* `prometheus.enabled=true`: Prometheus addon will be installed (default)
* `grafana.enabled=true`: Grafana addon will be installed
* `tracing.enabled=false`: Tracing(jaeger) addon will be installed
* `kiali.enabled=false`: Kiali addon will be installed


**Render Istio’s core components to a Kubernetes manifest:**
```bash
helm template install/kubernetes/helm/istio --name istio --namespace istio-system \
  --set security.enabled=false \
  --set ingress.enabled=false \
  --set gateways.istio-ingressgateway.enabled=false \
  --set gateways.istio-egressgateway.enabled=false \
  --set galley.enabled=false \
  --set sidecarInjectorWebhook.enabled=true \
  --set mixer.policy.enabled=false \
  --set mixer.telemetry.enabled=true \
  --set global.proxy.envoyStatsd.enabled=false \
  --set pilot.sidecar=false \
  --set prometheus.enabled=true \
  --set grafana.enabled=true \
  --set tracing.enabled=false \
  --set kiali.enabled=false \
  > $HOME/istio-telemetry.yaml
```

**Install the components via the manifest::**
```bash
kubectl create namespace istio-system
kubectl apply -f $HOME/istio-telemetry.yaml
```

**References:**
* [Minimal Istio Installation](https://istio.io/docs/setup/kubernetes/minimal-install/)
* [Installation Options](https://istio.io/docs/reference/config/installation-options/)
* [Helm Configuration](https://github.com/istio/istio/blob/master/install/kubernetes/helm/istio/README.md#configuration)
* [Helm Values](https://github.com/istio/istio/blob/master/install/kubernetes/helm/istio/values.yaml)

### Sidecar injection

**Label the `default` namespace with `istio-injection=enabled`:**
```bash
kubectl label namespace default istio-injection=enabled
```

**Check for which namespaces the sidecar injection is enabled:**
```bash
kubectl get namespace -L istio-injection
```

## Upgrade Istio

[Upgrading Istio](https://istio.io/docs/setup/kubernetes/upgrading-istio/)

1. [Download the new Istio release](https://github.com/istio/istio/releases) and change directory to the new release directory.

2. Upgrade Istio’s Custom Resource Definitions via `kubectl apply`, and wait a few seconds for the CRDs to be committed in the kube-apiserver:
   ```bash
   kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
   ```

3. Upgrade with Kubernetes rolling update:
   ```bash
   helm template install/kubernetes/helm/istio --name istio \
       --namespace istio-system > install/kubernetes/istio.yaml

   kubectl apply -f install/kubernetes/istio.yaml
   ```

## Architecture

[Simplifying Microservice Architecture With Envoy and Istio](https://dzone.com/articles/simplifying-microservice-architecture-with-envoy-a)

Two components:
* Control Plane - Controls the Data plane and is responsible for configuring the proxies for traffic management, policy enforcement, and telemetry collection.
* Data Plane - Comprises of Envoy proxies deployed as sidecars in each of the pods. All the application traffic flows through the Envoy proxies.

**Pilot** - Provides service discovery for the Envoy sidecars, traffic management capabilities for intelligent routing and resiliency (timeouts, retries, circuit breakers)

**Mixer** - Platform independent component which enforces access control and usage policies across the service mesh, and collects telemetry data from the Envoy proxy and other services.

**Citadel** - Provides strong service-to-service and end-user authentication with built-in identity and credential management.

All traffic entering and leaving the Istio service mesh is routed via the **Ingress/Egress Controller**. By deploying an Envoy proxy in front of services, you can conduct A/B testing, deploy canary services, etc. for user-facing services. Envoy is injected into the service pods inside the data plane using Istioctl kube-inject.

The Envoy sidecar logically calls Mixer before each request to perform precondition checks, and after each request to report telemetry. The sidecar has local caching such that a large percentage of precondition checks can be performed from the cache. Additionally, the sidecar buffers outgoing telemetry such that it only calls Mixer infrequently.

## Metrics

[Default Metrics](https://istio.io/docs/reference/config/policy-and-telemetry/metrics/)

To open Prometheus dashboard, enter the following command:
```bash
kubectl port-forward -n istio-system $(kubectl get pods --namespace istio-system --selector=app=prometheus --output=jsonpath="{.items..metadata.name}") 9090
```

Open [http://localhost:9090](http://localhost:9090).


To open Grafana, enter the following command:
```bash
kubectl port-forward -n istio-system $(kubectl get pods --namespace istio-system --selector=app=grafana --output=jsonpath="{.items..metadata.name}") 3000
```

Open [http://localhost:3000](http://localhost:3000).

## Tracing

Zipkin vs. Jaeger
