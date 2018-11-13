# Knative

## User Stories

### User Story: [Import services from GitHub](https://github.com/UST-MICO/mico/issues/26)

We want to import services by entering a GitHub URL based on meta-data:
* Repo name = service name
* Tags = Service versions
* Repo must contain a Dockerfile
* Dockerfile must be on top-level
* Import dialog could have an input field to specify the relative Dockerfile location

### User Story: [Evaluate Knative build](https://github.com/UST-MICO/mico/issues/49)

Knative could be used to exploit a source-to-URL workflow with Kubernetes when services are specified with a Dockerfile.

## Installation on Minikube

See chapter [Minikube](./minikube.md) for instructions.

## Installation on Azure

See chapter [Azure Kubernetes Service](./aks.md) for instructions.

## Deploying a sample app

There are multiple sample applications, developed on Knative, available: [Knative serving sample applications](https://github.com/knative/docs/blob/master/serving/samples/README.md)

Here we use the [Hello World - Spring Boot Java sample](https://github.com/knative/docs/blob/master/serving/samples/helloworld-java/README.md)...

**The app is already available as a Docker image. If you just want to deploy it to the Kubernetes Cluster, you can skip the next section.**

### Building the app

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

Go to http://localhost:8080/ to see your Hello World! message.

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

## Deploying a source-to-URL example app

A sample that shows how to use Knative to go from source code in a git repository to a running application with a URL.

The app is also available at [Knative serving sample applications](https://github.com/knative/docs/blob/master/serving/samples/README.md)

### Install the kaniko build template

This sample leverages the kaniko build template to perform a source-to-container build on your Kubernetes cluster.

```bash
kubectl apply --filename https://raw.githubusercontent.com/knative/build-templates/master/kaniko/kaniko.yaml
```

### Register secrets for Docker Hub

In order to push the container that is built from source to Docker Hub, register a secret in Kubernetes for authentication with Docker Hub.

Save this file as `docker-secret.yaml` with your Docker Hub credentials:
```
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
```
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
```
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

## Access the service

View the IP address of the service:
```bash
kubectl get svc knative-ingressgateway --namespace istio-system
```

Export the Ingress hostname and IP address as environment variables:
```bash
export SERVICE_HOST=`kubectl get route helloworld-java --output jsonpath="{.status.domain}"`
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
Navigate to the Grafana UI at http://localhost:3000.
