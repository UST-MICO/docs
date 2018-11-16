# Source-to-URL deployment

Technical Story: [Evaluate Knative build](https://github.com/UST-MICO/mico/issues/49)

## Context and Problem Statement

We want to have a source-to-URL deployment on Kubernetes based on Dockerfiles.

## Decision Drivers

* Must run on Kubernetes cluster
* Must not require root access
* URL to GitHub repository with Dockerfile must be sufficient

## Considered Options

* Docker in Docker
* img
* Kaniko
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

### Docker in Docker

* Good, because it is a well known approach
* Bad, because it requires privileged mode in order to function, which is a significant security concern
* Bad, because it generally incurs a performance penalty and can be quite slow

### img

[example | description | pointer to more information | …] <!-- optional -->

* Good, because it doesn't require root access
* Good, because [argument b]
* Bad, because it requires ["RawProc access](https://github.com/kubernetes/community/pull/1934/files) to create nested containers

### Kaniko

[example | description | pointer to more information | …] <!-- optional -->

* Good, because it is a popular open source project by Google
* Good, because it is well documented and easy to use
* Good, because it was designed to work on Kubernetes
* Good, because it does not need root access or any special access rights (like "RawProc")

### Knative build

[example | description | pointer to more information | …] <!-- optional -->

* Good, because [argument a]
* Good, because [argument b]
* Bad, because [argument c]
* … <!-- numbers of pros and cons can vary -->
