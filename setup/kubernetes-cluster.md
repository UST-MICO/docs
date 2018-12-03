# Kubernetes Cluster

MICO runs on the Azure Kubernetes Service (AKS).

## Environment

**Names:**
```bash
export LOCATION=westeurope
export RESOURCE_GROUP=ust-mico-resourcegroup
export CLUSTER_NAME=ust-mico-cluster
export ACR_NAME=ustmicoregistry
```

**Switch context of `kubectl`:**
```bash
kubectl config use-context $CLUSTER_NAME-admin
```

**Login:**
```bash
az login
```

**Get credentials:**
```bash
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --admin
```

If a different object with the same name already exists in your cluster configuration, use `--overwrite-existing` to override it.


## Cluster details

**Current cluster:**
* VM: Standard_B2s (2 vCPUs, 4 GB RAM)
* Nodes: 3
* OS Disk Size: 30 GB
* Location: westeurope
* Kubernetes version: 1.11.4

**Get the details for a managed Kubernetes cluster:**
```bash
az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```


## Cluster creation

### Resource group

Our self created resource group is named `ust-mico-resourcegroup`. Whenever a resource group is created, "the AKS resource provider automatically creates the second one during deployment, such as MC_myResourceGroup_myAKSCluster_eastus" ([FAQ AKS](https://docs.microsoft.com/de-de/azure/aks/faq)). This second resource group contains all of the infrastructure resources associated with the cluster (e.g. Kubernetes node VMs, virtual networking, and storage). The automatically created second resource group is named `MC_ust-mico-resourcegroup_ust-mico-cluster_westeurope`.

### Create Azure Kubernetes Service (AKS)

**Create AKS:**
```bash
az aks create --resource-group $RESOURCE_GROUP \
--name $CLUSTER_NAME \
--generate-ssh-keys \
--kubernetes-version 1.11.4 \
--node-vm-size Standard_B2s \
--node-count 3
```
For more information see [Quickstart: Deploy an Azure Kubernetes Service (AKS) cluster](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough).


## Cluster management

### Open Kubernetes dashboard

```bash
az aks browse --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```

How to access the Kubernetes dashboard is described in the Kubernetes Cluster Resource in Microsoft Azure.

If Cluster uses RBAC: Usage of Kubernetes Dashboard is restricted and most of the operations are not permitted. To be able to operate through the dashboard, execute the following command:
```bash
kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
```

**WARNING:** This configuration of a ClusterRoleBinding may lead to security issues, there is no additional authentication and the dashboard is publicly accessable! [Access the Kubernetes web dashboard in Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/kubernetes-dashboard#for-rbac-enabled-clusters)

### Public Access

* Public static IP address: 40.118.21.236


## Container Registry

MICO uses the Azure Container Registry (ACR) to store container images.

**Login:**
```bash
az acr login --name $ACR_NAME
```

### Create Azure Container Registry (ACS)

**Create ACR:**
```bash
az acr create --resource-group $RESOURCE_GROUP \
--name $ACR_NAME \
--sku Basic
```
For more information see [Tutorial: Deploy and use Azure Container Registry](https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-prepare-acr).

### Grant Kuberentes Cluster access to ACR

**[Grant AKS read access to ACR](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-auth-aks#grant-aks-access-to-acr)**:
```bash
#!/bin/bash

AKS_RESOURCE_GROUP=ust-mico-resourcegroup
AKS_CLUSTER_NAME=ust-mico-cluster
ACR_RESOURCE_GROUP=ust-mico-resourcegroup
ACR_NAME=ustmicoregistry

# Get the id of the service principal configured for AKS
CLIENT_ID=$(az aks show --resource-group $AKS_RESOURCE_GROUP --name $AKS_CLUSTER_NAME --query "servicePrincipalProfile.clientId" --output tsv)

# Get the ACR registry resource id
ACR_ID=$(az acr show --name $ACR_NAME --resource-group $ACR_RESOURCE_GROUP --query "id" --output tsv)

# Create role assignment
az role assignment create --assignee $CLIENT_ID --role Reader --scope $ACR_ID
```

### Manage ACR

**List container images:**
```bash
az acr repository list --name $ACR_NAME --output table
```
