# Debbuging

## Neo4j

To execute arbitrary queries you can use the Cypher Shell that is included in the Neo4j Docker image.

Access `cypher-shell`:
```bash
# Get the pod name of the Neo4j deployment (change the namespace if necessary)
NEO4J_POD_NAME=$(kubectl get pods -n mico-system --selector=run=neo4j --output=jsonpath={.items..metadata.name})

# Run bash inside the container
kubectl exec $NEO4J_POD_NAME -n mico-system -it /bin/bash

# Run cypher-shell
bash-4.4# cypher-shell
```

Now you are able to execute queries.

Get all nodes:
```sql
MATCH (n) RETURN n
```

Delete all nodes and relationships:
```sql
MATCH (n)
DETACH DELETE n
```

For more queries see the official documentation of [Cypher](https://neo4j.com/docs/cypher-manual/current/introduction/).
