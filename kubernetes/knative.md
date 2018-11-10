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

Get the IP address of the service:
```bash
kubectl get svc knative-ingressgateway --namespace istio-system
```

Optionally export the IP address:
```bash
export IP_ADDRESS=$(kubectl get svc knative-ingressgateway --namespace istio-system --output 'jsonpath={.status.loadBalancer.ingress[0].ip}')
```

Get the URL of the service:
```bash
kubectl get ksvc helloworld-java \
    --output=custom-columns=NAME:.metadata.name,DOMAIN:.status.domain
```

Make a request to the app:
```bash
curl -H "Host: helloworld-java.default.example.com" http://${IP_ADDRESS}
```
