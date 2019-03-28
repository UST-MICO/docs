# Testing

Depending on which component of MICO you want to test, there are different ways to do that.

* [Testing](#testing)
  * [Local testing without cluster](#local-testing-without-cluster)
  * [Local testing of the MICO backend with access to the cluster](#local-testing-of-the-mico-backend-with-access-to-the-cluster)
    * [Testing mico-core](#testing-mico-core)
    * [Neo4j](#neo4j)
  * [Local testing of the MICO dashboard with access to the cluster](#local-testing-of-the-mico-dashboard-with-access-to-the-cluster)
  * [Testing in cluster with own namespace](#testing-in-cluster-with-own-namespace)
    * [Preparation](#preparation)
    * [Usage](#usage)
    * [Update Kubernetes deployment in cluster](#update-kubernetes-deployment-in-cluster)
    * [Check current configuration](#check-current-configuration)
    * [Clean up](#clean-up)
  * [Testing in cluster with Telepresence](#testing-in-cluster-with-telepresence)
    * [Installation](#installation)
    * [Usage with local `mico-core`](#usage-with-local-mico-core)
    * [Usage with local `mico-admin`](#usage-with-local-mico-admin)
    * [Usage with IntelliJ](#usage-with-intellij)

## Local testing without cluster

This is only an option if you don't need the Kubernetes API. Be aware that all calls made by `mico-core` to the Kubernetes API will cause an exception.

Use `docker-compose` to start `mico-core`, `neo4j` and `redis` as the database:
```bash
docker-compose up --build
```

To only start `neo4j` and `redis` use
```bash
docker-compose up neo4j redis
```

## Local testing of the MICO backend with access to the cluster

With this approach you are able to debug `mico-core` locally. Furthermore you are able to connect the Neo4j browser to a local instance (simple way to view what's going on in the database).
It is required to have `kubectl` installed on your system. If you don't have `kubectl` installed yet, you should do that now. For more information see [Setup](../setup/index).

Start Neo4j and Redis locally. The easiest way is to use `docker-compose`:
```bash
docker-compose up neo4j redis
```

To start `mico-core` locally there are basically 3 ways:

* Start with your IDE, to be able to debug `mico-core` (recommended)

* Build and start with maven:
  ```bash
  mvn -B -f mico-core/pom.xml clean package -Dmaven.test.skip=true
  java -jar -Dspring.profiles.active=dev mico-core/target/mico-core-1.0-SNAPSHOT.jar
  ```

* `docker-compose` is currently not an option because `kubectl` and a proper Kubernetes configuration is missing in the Docker image.

Per default the Spring profile `dev` set in `application.properties` configures `mico-core` to use the Kubernetes namespace `mico-testing` as the target for the `Builds` and the finally running *MicoServices*. This namespace must include the Kubernetes `Secret` and the `ServiceAccount` for the authentication to DockerHub. To create the namespace `mico-testing` with this resources included you can use the script [`/install/kubernetes/test-setup-only-build-environment.sh`](https://github.com/UST-MICO/mico/blob/master/install/kubernetes/test-setup-only-build-environment.sh).

Last but not least you should provide a way to connect to Prometheus. This is required if you want to retrieve the status of a deployed MicoApplication and its MicoServices. The easiest way is connect to the running Prometheus instance inside your cluster by making a port-forwarding:
```bash
kubectl port-forward svc/prometheus -n monitoring 9090:9090
```

### Testing mico-core

To test the locally running `mico-core` you can either use `mico-admin` (start it also locally) or make requests with *curl* or *Postman*. For Postman there are some collections and environments provided. See the [postman directory](./postman/index) for more information.

### Neo4j

You can connect to the remote interface of Neo4j via [http://localhost:7474/browser](http://localhost:7474/browser/).
In the browser you are able to use the Cypher Shell and execute cypher statements.

Get all:
```sql
MATCH (n)
RETURN n;
```

Delete all:
```sql
MATCH (n)
DETACH DELETE n;
```

## Local testing of the MICO dashboard with access to the cluster

`mico-admin` needs access to `mico-core`. You can either start `mico-core` locally or access `mico-core` in the Kubernetes cluster by using a port forwarding:
```bash
kubectl port-forward svc/mico-core -n mico-system 8080:8080
```

Be aware that you make requests to `mico-core` inside of the cluster in the namespace `mico-system`.
If you want to test with an own instance of `mico-core`, consider to create an own namespace like described in the following section [Testing in cluster with own namespace](#testing-in-cluster-with-own-namespace).
Even then you are able to use a local instance of `mico-admin` and access `mico-core` inside of the cluster. If you have choosen e.g. the namespace name `mico-testing` you are able to access `mico-core` by using the port forwarding `kubectl port-forward svc/mico-core -n mico-testing 8080:8080`.

## Testing in cluster with own namespace

To be able to test in an exclusive environment, you can deploy MICO in a separate namespace inside the cluster.
For this purpose the script [`/install/kubernetes/test-setup-all-in-own-namespace.sh`](https://github.com/UST-MICO/mico/blob/master/install/kubernetes/test-setup-all-in-own-namespace.sh) in the [MICO repository](https://github.com/UST-MICO/mico) can be used.

### Preparation

Prepare yourself to enter a name for the test namespace (e.g. `mico-testing-1337`) as well as your credentials for DockerHub.

Now you are able to execute the interactive setup script `install/kubernetes/test-setup-all-in-own-namespace.sh`.

The Neo4j database needs ~5min until it is ready.
You can check the current status with
```bash
kubectl get pods -n ${NAMESPACE} --watch
```

### Usage

`mico-admin` gets after some seconds a public IP. To watch the service for changes use
```bash
kubectl get svc mico-admin -n ${NAMESPACE} --watch
```

To get the external IP directly use
```bash
echo $(kubectl get svc mico-admin -n ${NAMESPACE} -o 'jsonpath={.status.loadBalancer.ingress[0].ip}')
```

You can use the IP to access the dashboard in your browser.

`mico-core` can't be accessed from the outside. You can either use `kubectl proxy` or `kubectl port-forward` to get access into the cluster:
* `kubectl proxy --port=8080`
  Example:
  ```bash
  curl localhost:8080/api/v1/namespaces/${NAMESPACE}/services/mico-core/proxy/applications
  ```
* `kubectl port-forward svc/mico-core -n ${NAMESPACE} 8080:8080`
  Example:
  ```bash
  curl localhost:8080/applications
  ```

### Update Kubernetes deployment in cluster

To update a single deployment you can change the Docker image of the specific deployment. Use this approach only in your separate namespace.

Example with deployment `mico-core`:
```bash
# Build a new Docker image (change tag name e.g. to the corresponding issue number) and push it to DockerHub
docker build -f Dockerfile.mico-core -t ustmico/mico-core:42-fix-api . && docker push ustmico/mico-core:42-fix-api

# Set the image of the Kubernetes deployment
kubectl set image deployment/mico-core mico-core=ustmico/mico-core:42-fix-api -n ${NAMESPACE}

# If there was another image set, the pods of the deployment will automatically be restarted with the new image
# If the image was already set, you must restart the pod manually
kubectl -n $NAMESPACE delete pod $(kubectl get pods -n $NAMESPACE --selector=run=mico-core --output=jsonpath={.items..metadata.name})
```

### Check current configuration

`mico-core` offers some additional endpoints that are part of the Spring Boot Actuator plugin.

To get the current configurations have a look at `localhost:8080/actuator/configprops`.
To format it and save it into a file use e.g.
```bash
curl localhost:8080/actuator/configprops | python3 -m json.tool > configprops.json
```

### Clean up

If you are finished with testing, delete the namespace:
```bash
kubectl delete namespace ${NAMESPACE}
```

## Testing in cluster with Telepresence

[Telepresence](https://www.telepresence.io/) is a tool to debug Kubernetes services locally.

**For Windows users:** Telepresence basically supports WSL, however some important features are missing (see [Windows support](https://www.telepresence.io/reference/windows)). Therefore I wasn't able to get it running in my environment.

### Installation

[Installing Telepresence](https://www.telepresence.io/reference/install):
```bash
curl -s https://packagecloud.io/install/repositories/datawireio/telepresence/script.deb.sh | sudo bash
sudo apt install --no-install-recommends telepresence
```

### Usage with local `mico-core`

Build `mico-core` with Docker:
```bash
docker build -f Dockerfile.mico-core -t ustmico/mico-core:localbuild .
```

Swap the Kubernetes deployment with the local Docker image:
```bash
telepresence --namespace mico-system --swap-deployment mico-core --docker-run \
  --rm -it ustmico/mico-core:localbuild
```

### Usage with local `mico-admin`

Build `mico-admin` with Docker:
```bash
docker build -f Dockerfile.mico-admin -t ustmico/mico-admin:localbuild .
```

Swap the Kubernetes deployment with the local Docker image:
```bash
telepresence --namespace mico-system --swap-deployment mico-admin --docker-run \
  --rm -it ustmico/mico-admin:localbuild
```

### Usage with IntelliJ

See [Debugging a Java Rate Limiter Service using Telepresence and IntelliJ IDEA](https://www.telepresence.io/tutorials/intellij).
