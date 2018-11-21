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

### Resource group

Our self created resource group is named `ust-mico-resourcegroup`. Whenever a resource group is created, "the AKS resource provider automatically creates the second one during deployment, such as MC_myResourceGroup_myAKSCluster_eastus" ([FAQ AKS](https://docs.microsoft.com/de-de/azure/aks/faq)). This second resource group contains all of the infrastructure resources associated with the cluster (e.g. Kubernetes node VMs, virtual networking, and storage). The automatically created second resource group is named `MC_ust-mico-resourcegroup_ust-mico-cluster_westeurope`.

### Open Kubernetes dashboard

```bash
az aks browse --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```
