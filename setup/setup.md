# MICO Setup

This section covers the setup of MICO. The current Kubernetes development environment is deployed to a Kubernetes cluster created with the Azure Kubernetes Service (AKS). It is described in [Azure Kubernetes Service](aks.md).

## Installation script

To install MICO to your Kubernetes cluster you can use the interactive setup script `install/kubernetes/setup.sh` from the [MICO repository](https://github.com/UST-MICO/mico). It will install all MICO components and its requirements to your cluster that is configured through `kubectl`. You can find more information about the script in the official [README](https://github.com/UST-MICO/mico#readme).

## Installation structure

The development of MICO is driven by the [monolith-first strategy](https://martinfowler.com/bliki/MonolithFirst.html).

MICO consists of

- Frontend: `mico-admin`
- Backend: `mico-core`

It requires the third-party components:

- Neo4j
- Knative Build
- Prometheus (+Grafana)
- kube-state-metrics

For each component there is an own Kubernetes configuration file (YAML) placed in the directory install/kubernetes. They create following namespace and deployment structure:

- kube-system

  - kubernetes-dashboard
  - kube-state-metrics
  - metrics-server
  - ...

- mico-system

  - mico-admin (_frontend_)
  - mico-core (_backend_)
  - neo4j-core (_database_)

- mico-build-bot

- mico-workspace

- knative-build

  - build-controller
  - build-webhook

- monitoring

  - prometheus-core
  - grafana-core
  - kube-state-metrics
  - alertmanager

The mentioned components are all Kubernetes _Deployment_ resources except `neo4j-core` that is a Kubernetes _StatefulSet_.

At the beginning the namespaces `mico-build-bot` and `mico-workspace` are empty. `mico-build-bot` is used for the build processing of the Docker images using _Knative Build_ and _Kaniko_. `mico-workspace` is used for the deployment of MICO services. Via labels they are composed to MICO applications.

### Authentication

`mico-core` requires super-user access to perform privileged actions on other resources. To give full control over every resource in the cluster and in all namespaces we use a `ClusterRoleBinding`. The configuration file `mico-cluster-admin.yaml` creates such a `ClusterRoleBinding` for the `ServiceAccount` "default" in the namespace `mico-system` that is used by `mico-core`.

### Labels

**Labels are currently under work in progress.**

Labels:

- `ust.mico/name`:

  - The name of the MicoService
  - Example value: "hello"

- `ust.mico/version`

  - The current version of the MicoService (semantic version)
  - Example value: "v1.0.0"

- `ust.mico/interface`

  - The name of the MicoServiceInterface
  - Example value: "hello-service"

- `ust.mico/instance`

  - The unique name of the MICO resource to be able to identify the associated Kubernetes resources
  - Example value: "hello-s6djsekw"

- `ust.mico/component`

  - The component within the architecture of a MicoApplication
  - Example value: "frontend"
