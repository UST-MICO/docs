# Testing

Depending on which component of MICO you want to test, there are different ways to do that.

* [Testing](#testing)
  * [Local testing without cluster](#local-testing-without-cluster)
  * [Local testing of the MICO backend with access to the cluster](#local-testing-of-the-mico-backend-with-access-to-the-cluster)
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

Use `docker-compose` to start `mico-core` and `neo4j` as the database:
```bash
docker-compose up --build
```

To only start Neo4j as a Docker container use
```bash
docker run -p 7687:7687 -p 7474:7474 -e NEO4J_dbms_security_auth__enabled=false --volume=$HOME/neo4j/data:/data --volume=$HOME/neo4j/logs:/logs neo4j:3.5.3
```

## Local testing of the MICO backend with access to the cluster

Currently you can't use `docker-compose` for that, because `kubectl` and a proper Kubernetes configuration is missing in the Docker image.
An alternative is to use `kubectl` that is installed on your host system. If you don't have `kubectl` installed yet, you should do that now. For more information see [Setup](../setup/index).

Start Neo4j either as a Docker container or as a local instance installed on your system (`dbms.security.auth_enabled` must be set to `false`).

To execute `mico-core`, build it first with Maven
```bash
mvn -B -f mico-core/pom.xml clean package -Dmaven.test.skip=true
```

and run the jar file
```bash
java -jar mico-core/target/mico-core-1.0-SNAPSHOT.jar
```

## Local testing of the MICO dashboard with access to the cluster

`mico-admin` needs access to `mico-core`. You can achieve that by using a port forwaring:
```bash
kubectl port-forward svc/mico-core -n mico-system 8080:8080
```

Be aware that you make requests to `mico-core` inside of the cluster in the namespace `mico-system`.
If you want to test with an own instance of `mico-core`, consider to create an own namespace like described in the following section [Testing in cluster with own namespace](#testing-in-cluster-with-own-namespace).
Even then you are able to use a local instance of `mico-admin` and access `mico-core` inside of the cluster. If you have choosen e.g. the namespace name `mico-testing` you are able to access `mico-core` by using the port forwarding `kubectl port-forward svc/mico-core -n mico-testing 8080:8080`.

## Testing in cluster with own namespace

To be able to test in an exclusive environment, you can deploy MICO in a separate namespace inside the cluster.
For this purpose the script [`/install/kubernetes/test-setup.sh`](https://github.com/UST-MICO/mico/blob/master/install/kubernetes/test-setup.sh) in the [MICO repository](https://github.com/UST-MICO/mico) can be used.

### Preparation

Prepare yourself to enter a name for the test namespace (e.g. `mico-testing-1337`) as well as your credentials for DockerHub.

Now you are able to execute the interactive setup script `install/kubernetes/test-setup.sh`.

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
# Build a new Docker image (change tag name)
docker build -f Dockerfile.mico-core -t ustmico/mico-core:mico-1337 .

# Push it to DockerHub
docker push ustmico/mico-core:mico-1337

# Set the image of the Kubernetes deployment
kubectl set image deployment/mico-core mico-core=ustmico/mico-core:mico-1337 -n ${NAMESPACE}

# If the image is already set, you can restart the pod with the new image
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
