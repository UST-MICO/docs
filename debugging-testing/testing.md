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

## Local testing without cluster

This is only an option if you don't need the Kubernetes API. Be aware that all calls made by `mico-core` to the Kubernetes API will cause an exception.

Use `docker-compose` to start `mico-core` and `neo4j` as the database:
```bash
docker-compose up --build
```

## Local testing of the MICO backend with access to the cluster

Currently you can't use `docker-compose` for that, because `kubectl` and a proper Kubernetes configuration is missing in the Docker image.
An alternative is to use `kubectl` that is installed in your system. If you don't have installed `kubectl` yet, you should do that. For more information see [/Setup/Kubernetes](./setup/kubernetes.md).

Start Neo4j either as a Docker container or as a local instance installed in your system (`dbms.security.auth_enabled` must be set to `false`).

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
kubectl get pods -n ${MICO_TEST_NAMESPACE} --watch
```

### Usage

`mico-admin` gets after some seconds a public IP. To watch the service for changes use
```bash
kubectl get svc mico-admin -n ${MICO_TEST_NAMESPACE} --watch
```

To get the external IP directly use
```bash
echo $(kubectl get svc mico-admin -n ${MICO_TEST_NAMESPACE} -o 'jsonpath={.status.loadBalancer.ingress[0].ip}')
```

You can use the IP to access the dashboard in your browser.

`mico-core` can't be accessed from the outside. You can either use `kubectl proxy` or `kubectl port-forward` to get access into the cluster:
* `kubectl proxy --port=8080`
  Example:
  ```bash
  curl localhost:8080/api/v1/namespaces/${MICO_TEST_NAMESPACE}/services/mico-core/proxy/applications
  ```
* `kubectl port-forward svc/mico-core -n ${MICO_TEST_NAMESPACE} 8080:8080`
  Example:
  ```bash
  curl localhost:8080/applications
  ```

### Update Kubernetes deployment in cluster

To update a single deployment you can change the Docker image of the specific deployment. Use this approach only in your separate namespace.

Example with deployment `mico-core`:
```bash
# Build a new Docker image
docker build -t ustmico/mico-core:localbuild -f Dockerfile.mico-core .

# Push it to DockerHub
docker push ustmico/mico-core:localbuild

# Set the image of the Kubernetes deployment
kubectl set image deployment/mico-core mico-core=ustmico/mico-core:localbuild -n ${MICO_TEST_NAMESPACE}

# If the image is already set, you can restart the pod with the new image
MICO_CORE_POD_NAME=$(kubectl get pods -n ${MICO_TEST_NAMESPACE} --selector=run=mico-core --output=jsonpath={.items..metadata.name})
kubectl delete pod ${MICO_CORE_POD_NAME} -n ${MICO_TEST_NAMESPACE}
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
kubectl delete namespace ${MICO_TEST_NAMESPACE}
```
