# MICO Setup

This section covers the setup of MICO. The current Kubernetes development environment is deployed to a Kubernetes cluster created with the Azure Kubernetes Service (AKS). It is described in [Azure Kubernetes Service](aks.md).

## Installation

**Script:**

To install MICO to your Kubernetes cluster you can use the interactive setup script `install/kubernetes/setup.sh` from the [MICO repository](https://github.com/UST-MICO/mico). It will install all MICO components and its requirements to your cluster that is configured through `kubectl`. You can find more information about the script in the official [README](https://github.com/UST-MICO/mico#readme).

**Structure:**

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

**Authentication:**

`mico-core` requires super-user access to perform privileged actions on other resources. To give full control over every resource in the cluster and in all namespaces we use a `ClusterRoleBinding`. The configuration file `mico-cluster-admin.yaml` creates such a `ClusterRoleBinding` for the `ServiceAccount` "default" in the namespace `mico-system` that is used by `mico-core`.

## Deployment

A deployment of a MICO application contains several steps:

1. Build container images for all included MICO services
2. Create Kubernetes *Deployments* and Kubernetes *Services*:
   - MICO Service → Kubernetes Deployment
   - MICO Service Interface → Kubernetes Service
3. Create interface connections

### 1. Build

Container images are build by [kaniko](https://github.com/GoogleContainerTools/kaniko) as part of the MICO system inside the Kubernetes cluster. Kaniko itself runs as a container and is triggered by [Knative Build](https://github.com/knative/build).
Kaniko builds the image based on the given Git clone URL, the Git revision (in our case always the release tag) and the path to the Dockerfile inside the Git repository (per default "Dockerfile"). After the build Kaniko pushes the image to the given Docker registry (per default DockerHub).
The `Build` resources and the build Pods are created in the namespace `mico-build-bot`. The build name is created by `mico-core` based on the name of the MICO service and its version with the prefix `build-` (e.g. `build-hello-v1-0-0`). The build pods get the same name with a random suffix (e.g. `build-hello-v1-0-0-pod-50b30d`). The name is important because subsequent builds reuse existing `Build` resources if they have the same name.

`mico-core` waits 10 seconds and checks if the build is finished. Subsequent checks follow in a 5 seconds cycle.
We consider a build as finished if the build pod has the phase `Succeeded` or `Failed`. If the build was successful the specified image name is stored as part of the MICO service entity in the database.

### 2. Kubernetes Resources

After all builds are finished, the required Kubernetes resources will be created or updated:
- MICO service → Kubernetes *Deployment*
- MICO service interface → Kubernetes *Service*

**Names:**

The names that are used for the Kubernetes resources are based on the names of the respective MICO resources (MICO service or MICO service interface) plus an additional random string with 8 alphanumeric characters.
Examples:
- Kubernetes *Deployment* (name of the MICO service is `hello`): `hello-qluhya8o`
- Kubernetes *Service* (name of the MICO service interface is `http`): `http-aopii3eg`

**Labels:**

Each Kubernetes *Deployment* and Kubernetes *Service* gets the following 4 MICO specific labels:

- `ust.mico/name`:

  - The name of a MICO service
  - Example value: "hello"

- `ust.mico/version`

  - The current version of a MICO service (semantic version)
  - Example value: "v1.0.0"

- `ust.mico/interface`

  - The name of a MICO service interface
  - Example value: "http"

- `ust.mico/instance`

  - The unique name of the MICO resource to be able to identify the associated Kubernetes resources
  - Example value: "hello-s6djsekw"

**Scaling:**

If a MICO service is already deployed (by another application) a scale out is performed (increasing the replicas) instead of recreating the Kubernetes *Deployment*.

**Creation:**

For the creation of a the Kubernetes *Deployment* the following service deployment information are used:
- Replicas
- Labels
- Environment Variables
- Image Pull Policy

**Database:**

The name of the created Kubernetes *Deployment* and the Kubernetes *Services* are stored as part of the MICO service deployment information in the database. That's necessary to know, whether the deployment was already performed with this specific service deployment information (that means that this MicoApplication triggered the deployment). If these information are not given, but the MICO service is already deployed, it's considered to be deployed by another MICO application.

### 3. Interface Connection

To be able to connect different MICO services with each other, `mico-core` creates environment variables with the DNS name of the interface of the other MICO service to connect with. These environment variables are applied on the Kubernetes *Deployment* of the MICO service.
The information that are required for such an interface connection is part of the service deployment information and consists of three properties:
- the short name of the MICO service to connect with (e.g. `spring-boot-realworld-example-app`)
- the name of the MICO service interface to connect with (e.g. `rest`)
- the name of the environment variable used by the frontend (e.g. `BACKEND_REST_API`)
