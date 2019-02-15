# Knative

## Installation on Minikube

See chapter [Minikube](./minikube.md) for instructions.

## Installation on Azure

See chapter [Azure Kubernetes Service](./azure.md) for instructions.

## Source-to-Image workflow (only Knative Build)

**This is currently the only usage of Knative for MICO.**

[Knative Build](https://github.com/knative/build) is used to achieve a *Source-to-Image workflow*. [Knative Serving](https://github.com/knative/serving) is currently not used.

Istio is not required to run Knative Build.

### Installation

**[Installing Knative](https://github.com/knative/docs/blob/master/install/Knative-with-AKS.md#installing-knative):**
```bash
# Install Knative Build and its dependencies:
kubectl apply --filename https://github.com/knative/build/releases/download/v0.3.0/release.yaml

# Monitor the Knative components:
kubectl get pods --namespace knative-build
```

### Authentication to a container registry

**Different destinations:**

* Docker Hub: `docker.io/ustmico/<image-name>`
* ACR: `ustmicoregistry.azurecr.io/<image-name>`
* GCR: `gcr.io/iaas/<image-name>`

**Authentication:**

Depending on your target container registry, there are different ways to authenticate.

* Docker Hub: [Basic authentication (Docker)](https://github.com/knative/docs/blob/master/build/auth.md#basic-authentication-docker)
* ACR: [Azure Container Registry authentication with service principals](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-auth-service-principal)
* GCR: [GoogleCloudPlatform/knative-build-tutorials/docker-build/](https://github.com/GoogleCloudPlatform/knative-build-tutorials/tree/master/docker-build)

#### DockerHub

Create a new namespace `mico-build-bot` (if not yet created):
```bash
kubectl create namespace mico-build-bot
```

Create a `Secret` with the name `build-bot-dockerhub-secret`:
```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/basic-auth
metadata:
  name: build-bot-dockerhub-secret
  namespace: mico-build-bot
  annotations:
    build.knative.dev/docker-0: https://index.docker.io/v1/
data:
  # Use 'echo -n "username" | base64 -w 0' to generate this string
  username: ${DOCKERHUB_USERNAME_BASE64}
  # Use 'echo -n "password" | base64 -w 0' to generate this string
  password: ${DOCKERHUB_PASSWORD_BASE64}
```

Create a new `Service Account` manifest which is used to link the build process to the secret. Save this file as `build-bot-dockerhub-service-account.yaml`:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-bot-dockerhub
  namespace: mico-build-bot
secrets:
- name: build-bot-dockerhub-secret
```

Provide your DockerHub credentials in Base64 encoding as environment variables:
```bash
export DOCKERHUB_USERNAME_BASE64=$(echo -n "username" | base64 -w 0)
export DOCKERHUB_PASSWORD_BASE64=$(echo -n "password" | base64 -w 0)
```

Insert the environment variables into the secret configuration file:
```bash
envsubst < build-bot-dockerhub-secret.yaml > build-bot-dockerhub-secret-with-credentials.yaml
```

Apply the manifest files to your cluster:
```bash
kubectl apply -f build-bot-dockerhub-secret-with-credentials.yaml build-bot-dockerhub-service-account.yaml
```

#### Azure Container Registry

**The Authentication to ACR doesn't work yet on Azure!**
```log
failed to push to destination: unexpected end of JSON input
```

ACR is officially not supported by Kaniko: [Issue#425](https://github.com/GoogleContainerTools/kaniko/issues/425)

Therefore DockerHub ist used.

---

There are already at least two `Service Principals` for following requirements:
* Read access from the AKS cluster to `pull` images
* Write access from the Jenkins Pipeline to `push` images

Now it's required to create another `Service Principal` with write access to the registry.

[Create a `Service Principal`](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-auth-service-principal#create-a-service-principal):
```bash
#!/bin/bash
SERVICE_PRINCIPAL_NAME=ust-mico-acr-knative-build

ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)

SP_PASSWD=$(az ad sp create-for-rbac --name $SERVICE_PRINCIPAL_NAME --scopes $ACR_REGISTRY_ID --role contributor --query password --output tsv)
SP_APP_ID=$(az ad sp show --id http://$SERVICE_PRINCIPAL_NAME --query appId --output tsv)

echo "Service principal ID: $SP_APP_ID"
echo "Service principal password: $SP_PASSWD"
```

Create a new namespace `mico-build-bot`:
```bash
kubectl create namespace mico-build-bot
```

Create a `Secret` with the name `build-bot-acr-secret`:
```bash
kubectl create secret docker-registry build-bot-acr-secret --docker-server $ACR_NAME.azurecr.io --docker-username $SP_APP_ID --docker-password $SP_PASSWD --namespace=mico-build-bot
```

Create a `ServiceAccount` with the name `build-bot-acr`:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-bot-acr
  namespace: mico-build-bot
secrets:
- name: build-bot-acr-secret
```


### Usage of Knative Build

Now we are ready to create and run a `Build`.

How a simple `Build` can be created and be executed is written down in [Creating a simple Knative Build](https://github.com/knative/docs/blob/master/build/creating-builds.md).

More `Build` examples: [Knative `Build` resources](https://github.com/knative/docs/blob/master/build/builds.md).

There is already a set of curated and supported `Build Templates` available in the Knative [`build-templates`](https://github.com/knative/build-templates) repository. For a `Dockerfile` build and the subsequent push of the resulting image the usage of *Kaniko* as the `Builder` is recommended. It is also used in the [Source-to-URL workflow](#source-to-url-workflow-knative-build-serving).

**Example application:**

Apply at first a `Secret` and a `Service Account` so that the resulting image can be pushed to a docker registry (here: Docker Hub).

**`build.yaml`:**
```yaml
apiVersion: build.knative.dev/v1alpha1
kind: Build
metadata:
  name: build-hello
  namespace: mico-build-bot
spec:
  serviceAccountName: build-bot-dockerhub
  source:
    git:
      url: https://github.com/dgageot/hello.git
      revision: master
  steps:
  - name: build-and-push
    image: gcr.io/kaniko-project/executor
    args:
    - --dockerfile=/workspace/Dockerfile
    - --destination=docker.io/ustmico/hello
```

Increase log level:
Add `--verbosity=debug` to `args`.


## Deploying an application (only Knative Serving)

**Knative Serving is currently not used in MICO!**

There are multiple sample applications, developed on Knative, available: [Knative serving sample applications](https://github.com/knative/docs/blob/master/serving/samples/README.md)

Here we use the [Hello World - Spring Boot Java sample](https://github.com/knative/docs/blob/master/serving/samples/helloworld-java/README.md)...

**The app is already available as a Docker image. If you just want to deploy it to the Kubernetes Cluster, you can skip the next section.**

### Building the app locally

Create a new empty web project:
```bash
curl https://start.spring.io/starter.zip \
    -d dependencies=web \
    -d name=helloworld \
    -d artifactId=helloworld \
    -o helloworld.zip
unzip helloworld.zip
```

Run the application locally:
```bash
./mvnw package && java -jar target/helloworld-0.0.1-SNAPSHOT.jar
```

Go to [localhost:8080](http://localhost:8080/) to see your Hello World! message.

Create Dockerfile:
```Dockerfile
FROM maven:3.5-jdk-8-alpine as build
ADD pom.xml ./pom.xml
ADD src ./src
RUN mvn package -DskipTests

FROM openjdk:8-jre-alpine
COPY --from=build /target/helloworld-*.jar /helloworld.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/helloworld.jar"]
```

Create service.yaml:
```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: helloworld-java
  namespace: default
spec:
  runLatest:
    configuration:
      revisionTemplate:
        spec:
          container:
            image: docker.io/ustmico/knative-helloworld-java
            env:
            - name: TARGET
              value: "Spring Boot Sample v1"
```

Building and deploying the sample with Docker:
```bash
# Build the container on your local machine
docker build -t ustmico/knative-helloworld-java .

# Push the container to docker registry
docker push ustmico/knative-helloworld-java
```

### Deployment to Kubernetes

Deploy the app into the Kubernetes cluster:
```bash
# Apply the manifest
kubectl apply --filename service.yaml

# Watch the pods for build and serving
kubectl get pods --watch
```

How to access the service is described in section [Access a service](#access-a-service).

## Source-to-URL workflow (Knative Build + Serving)

**Knative Serving is currently not used in MICO!**

This section describes how to use Knative to go from source code in a git repository to a running application with a URL.
Knative Build and Knative Serving are required.

As a sample application we use a Go application like desribes in [Orchestrating a source-to-URL deployment on Kubernetes](https://github.com/knative/docs/blob/master/serving/samples/source-to-url-go/README.md).

### Install the kaniko build template

This sample leverages the kaniko build template to perform a source-to-container build on your Kubernetes cluster.

```bash
kubectl apply --filename https://raw.githubusercontent.com/knative/build-templates/master/kaniko/kaniko.yaml
```

### Register secrets for Docker Hub

In order to push the container that is built from source to Docker Hub, register a secret in Kubernetes for authentication with Docker Hub.

Save this file as `docker-secret.yaml` with your Docker Hub credentials:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: basic-user-pass
  annotations:
    build.knative.dev/docker-0: https://index.docker.io/v1/
type: kubernetes.io/basic-auth
data:
  # Use 'echo -n "username" | base64 -w 0' to generate this string
  username: BASE64_ENCODED_USERNAME
  # Use 'echo -n "password" | base64 -w 0' to generate this string
  password: BASE64_ENCODED_PASSWORD
```

Create a new `Service Account` manifest which is used to link the build process to the secret. Save this file as `service-account.yaml`:
```yaml
 apiVersion: v1
 kind: ServiceAccount
 metadata:
   name: build-bot
 secrets:
 - name: basic-user-pass
 ```

 Apply the manifest files to your cluster:
 ```bash
$ kubectl apply --filename docker-secret.yaml
secret "basic-user-pass" created
$ kubectl apply --filename service-account.yaml
serviceaccount "build-bot" created
 ```

### Deploying the sample

You need to create a service manifest which defines the service to deploy, including where the source code is and which build-template to use. Create a file named service.yaml and copy the following definition. Make sure to replace `{DOCKER_USERNAME}` with your own Docker Hub username:
```yaml
apiVersion: serving.knative.dev/v1alpha1
kind: Service
metadata:
  name: app-from-source
  namespace: default
spec:
  runLatest:
    configuration:
      build:
        apiVersion: build.knative.dev/v1alpha1
        kind: Build
        spec:
          serviceAccountName: build-bot
          source:
            git:
              url: https://github.com/mchmarny/simple-app.git
              revision: master
          template:
            name: kaniko
            arguments:
            - name: IMAGE
              value: docker.io/{DOCKER_USERNAME}/app-from-source:latest
      revisionTemplate:
        spec:
          container:
            image: docker.io/{DOCKER_USERNAME}/app-from-source:latest
            imagePullPolicy: Always
            env:
            - name: SIMPLE_MSG
              value: "Hello sample app!"
```

### Deploying the sample to Kubernetes

Deploy the app into the Kubernetes cluster:
```bash
# Apply the manifest
kubectl apply --filename service.yaml

# Watch the pods for build and serving
kubectl get pods --watch

# Check the state of the service and get the service object
kubectl get ksvc app-from-source --output yaml
```

## Access a service

View the IP address of the service:
```bash
kubectl get svc knative-ingressgateway --namespace istio-system
```

In the following commands replace `<service-name>` with your specified service name (e.g. `helloworld-java` or `app-from-source`).

To find the URL for your service, use
```bash
kubectl get ksvc <service-name> \
    --output=custom-columns=NAME:.metadata.name,DOMAIN:.status.domain
```

Export the Ingress hostname and IP address as environment variables:
```bash
export SERVICE_HOST=`kubectl get route <service-name> --output jsonpath="{.status.domain}"`
export SERVICE_IP=`kubectl get svc knative-ingressgateway --namespace istio-system \
--output jsonpath="{.status.loadBalancer.ingress[*].ip}"`
```

Make a request to the app:
```bash
curl --header "Host:$SERVICE_HOST" http://${SERVICE_IP}
```

## Explore the configuration

**View the created Route resource:**
```bash
kubectl get route --output yaml
```

**View the created Configuration resource:**
```bash
kubectl get configurations --output yaml
```

**View the Revision that was created by our Configuration:**
```bash
kubectl get revisions --output yaml
```

## Clean up

To clean up a service:
```bash
kubectl delete --filename service.yaml
```

## Monitoring

### Accessing logs

See [Accessing logs](https://github.com/knative/docs/blob/master/serving/accessing-logs.md) for more information.

To open the Kibana UI (the visualization tool for Elasticsearch), start a local proxy with the following command:
```bash
kubectl proxy
```
This command starts a local proxy of Kibana on port 8001. For security reasons, the Kibana UI is exposed only within the cluster.
Navigate to the Kibana UI. It might take a couple of minutes for the proxy to work.

### Metrics

See [Accessing metrics](https://github.com/knative/docs/blob/master/serving/accessing-metrics.md) for more information.

To open Grafana, enter the following command:
```bash
kubectl port-forward --namespace knative-monitoring $(kubectl get pods --namespace knative-monitoring --selector=app=grafana --output=jsonpath="{.items..metadata.name}") 3000
```
This starts a local proxy of Grafana on port 3000. For security reasons, the Grafana UI is exposed only within the cluster.
Navigate to the Grafana UI at [localhost:3000](http://localhost:3000).

### Memory usage

**Memory usage of an one-node-cluster (total max: 1,7 GB):**
* kube-system (total: 315 MB)
  * heapster: 22 MB
  * kube-dns-v20 (1): 21 MB
  * kube-dns-v20 (2): 21 MB
  * kube-proxy: 26 MB
  * kube-svc-redirect: 35 MB
  * kubernetes-dashboard: 13 MB
  * metrics-server: 13 MB
  * omsagent-mvrrx: 102 MB
  * omsagent-rs: 52 MB
  * tunnelfront: 10 MB
* istio-system (total: 456 MB)
  * istio-telemetry: 182 MB
  * istio-policy: 138 MB
  * istio-egressgateway: 29 MB
  * istio-ingressgateway: 26 MB
  * istio-pilot: 45 MB
  * istio-citadel: 11 MB
  * istio-galley: 10 MB
  * istio-sidecar-injector: 9 MB
  * istio-statsd-prom-bridge: 6 MB
* knative-build (total: 22 MB)
  * build-controller: 11 MB
  * build-webhook: 11 MB
* other processes running in cluster: 1 GB

## Troubleshooting

**Get logs of a build:**

Check the Init Container (e.g. `pod-name` = `kaniko-build-f9x52`):
```bash
kubectl describe pod <pod-name>
```

Accessing logs from Init Containers (e.g. `init-container` = `build-step-build-and-push`):
```bash
kubectl logs <pod-name> -c <init-container>
```
