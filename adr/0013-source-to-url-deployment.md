# Source-to-URL deployment

Technical Story: [Evaluate Knative build](https://github.com/UST-MICO/mico/issues/49)

## Context and Problem Statement

We want to have a source-to-URL deployment on Kubernetes to import services based on a GitHub repository with an included Dockerfile.

## Decision Drivers

* MUST run on Kubernetes cluster
* MUST run completely in userspace (no root access required)
* URL to GitHub repository (with included Dockerfile) SHOULD be sufficient

## Considered Options

* Google Cloud Build
* Azure Container Registry Tasks
* Knative Build

## Decision Outcome

Chosen option: "[option 1]", because [justification. e.g., only option, which meets k.o. criterion decision driver | which resolves force force | … | comes out best (see below)].

### Positive Consequences <!-- optional -->

* [e.g., improvement of quality attribute satisfaction, follow-up decisions required, …]
* …

### Negative consequences <!-- optional -->

* [e.g., compromising quality attribute, follow-up decisions required, …]
* …

## Pros and Cons of the Options

### Google Cloud Build

[example | description | pointer to more information | …] <!-- optional -->

* Good, because [argument a]
* Good, because [argument b]
* Bad, because [argument c]
* … <!-- numbers of pros and cons can vary -->

### Azure Container Registry Tasks

[example | description | pointer to more information | …] <!-- optional -->

* Good, because [argument a]
* Good, because [argument b]
* Bad, because [argument c]
* … <!-- numbers of pros and cons can vary -->

### Knative Build

[GitHub: Knative Build](https://github.com/knative/build)

* Good, because it has the backing of industry giants (Google, Red Hat, IBM, SAP)
* Good, because it is designed for Kubernetes
* Good, because it provides a standard, portable, reusable, and performance optimized method for defining and running on-cluster container image builds
* Bad, because it is still work-in-progress
* Bad, because Knative requires Istio that adds many complications to the cluster and consumes much resources
