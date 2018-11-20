# Our setup

MICO runs on the Azure Kubernetes Service (AKS).

## Cluster details

**Current cluster:**
* VM: Standard_B2s (2 vCPUs, 4 GB RAM)
* Nodes: 3
* OS Disk Size: 30 GB
* Location: westeurope
* Kubernetes version: 1.11.3

**Get the details for a managed Kubernetes cluster:**
```bash
az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```

## Cluster management

### Login

**Create environment variables:**
```bash
export LOCATION=westeurope
export RESOURCE_GROUP=ust-mico-resourcegroup
export CLUSTER_NAME=ust-mico-cluster
```

**Login:**
```bash
az login
```

**Get credentials:**
```bash
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --admin
```

### Open Kubernetes dashboard

```bash
az aks browse --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```
