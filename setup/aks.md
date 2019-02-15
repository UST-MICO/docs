# Azure Kubernetes Service

The MICO development environment runs on the Azure Kubernetes Service (AKS).

## First steps

To connect to the cluster you need two command-line tools:
* *kubectl*: [Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* Azure CLI *az*: [Install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

To be able to execute the cluster related commands used in this documentation set following environment variables:
```bash
export LOCATION=westeurope
export RESOURCE_GROUP=ust-mico-resourcegroup
export CLUSTER_NAME=ust-mico-cluster
export RESOURCE_GROUP_NODE=MC_$RESOURCE_GROUP\_$CLUSTER_NAME\_$LOCATION
```

Sign in to Azure ([more information](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli)):
```bash
az login
```

Configure `kubectl` to connect to the cluster ([more information](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough#connect-to-the-cluster)):
```bash
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --admin
```

If a different object with the same name already exists in your cluster configuration, use `--overwrite-existing` to override it.

If you use `kubectl` to connect to multiple clusters, it's useful to use multiple contexts.
To switch to the automatically created context of the MICO cluster, use
```bash
kubectl config use-context $CLUSTER_NAME-admin
```


## Cluster details

**Current cluster:**
* VM: Standard_B2s (2 vCPUs, 4 GB RAM)
* Nodes: 3
* OS Disk Size: 30 GB
* Location: westeurope
* Kubernetes version: 1.11.5

**Get the details for a managed Kubernetes cluster:**
```bash
az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```

**Costs:**

With two nodes we currently consume about 3 € per day.

You can check the current costs either in the [Azure Portal](https://portal.azure.com/) under "Visual Studio Enterprise - Cost analysis" or in the [subscription dashboard of your Azure account](https://account.azure.com/Subscriptions).


## Upgrade Kubernetes

[Upgrade an Azure Kubernetes Service (AKS) cluster](https://docs.microsoft.com/en-us/azure/aks/upgrade-cluster)

**Check if there are updates available:**
```bash
az aks get-upgrades --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP --output table
```

**Upgrade Kubernetes to specific version:**
```bash
az aks upgrade --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP --kubernetes-version 1.11.5
```

**Confirm that the upgrade was successful:**
```bash
az aks show --name $CLUSTER_NAME --resource-group $RESOURCE_GROUP --output table
```


## Cluster creation

### Resource group

Our self created resource group is named `ust-mico-resourcegroup`. Whenever a resource group is created, "the AKS resource provider automatically creates the second one during deployment, such as MC_myResourceGroup_myAKSCluster_eastus" ([FAQ AKS](https://docs.microsoft.com/de-de/azure/aks/faq)). This second resource group contains all of the infrastructure resources associated with the cluster (e.g. Kubernetes node VMs, virtual networking, and storage). The automatically created second resource group is named `MC_ust-mico-resourcegroup_ust-mico-cluster_westeurope`. It is a concatention of `MC_$RESOURCE_GROUP\_$CLUSTER_NAME\_$LOCATION`.

### Create Azure Kubernetes Service (AKS)

**Create AKS:**
```bash
az aks create --resource-group $RESOURCE_GROUP \
--name $CLUSTER_NAME \
--generate-ssh-keys \
--kubernetes-version 1.11.5 \
--node-vm-size Standard_B2s \
--node-count 2
```
For more information see [Quickstart: Deploy an Azure Kubernetes Service (AKS) cluster](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough).


### Redeploy an Azure Kubernetes Service (AKS) cluster

If an AKS cluster was deleted by accident or was removed and should be setup again.

**Create AKS:**

Use command to create an AKS cluster like described above in section [Create Azure Kubernetes Service (AKS)](#create-azure-kubernetes-service-aks).

**Setup credentials again:**

* on Jenkins VM:
  * remove old config file in `~/.kube/` (only if new cluster has the same name as the old cluster)
  * get new credentials:
    ```bash
    az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
    ```
  * copy the new generate config file from `~/.kube/` to `/var/lib/jenkins`
  * set ownership of config file to jenkins user + jenkins group
    ```bash
    sudo chown jenkins:jenkins /var/lib/jenkins/config
    ```
* on Kubernetes VM:
  * if new cluster has same name as the old cluster: remove config file in `~/.kube/`
  * get new credentials:
    ```bash
    az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
    ```


## Cluster management

### Open Kubernetes dashboard

```bash
az aks browse --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```

Grant full admin privileges to Dashboard's Service Account [more information](https://github.com/kubernetes/dashboard/wiki/Access-control#admin-privileges):
```bash
kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard \
 && kubectl label clusterrolebinding kubernetes-dashboard k8s-app=kubernetes-dashboard
```

**WARNING:** This configuration of a ClusterRoleBinding may lead to security issues, there is no additional authentication and the dashboard is publicly accessable! [Access the Kubernetes web dashboard in Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/kubernetes-dashboard#for-rbac-enabled-clusters)

### Public Access

**Create static IP address:**

For a stable, public access to the MICO dashboard, a static IP address is recommended. This IP address with the name `MICO-PublicIP` was generated by:
```bash
az network public-ip create \
    --resource-group $RESOURCE_GROUP_NODE \
    --name MICO-PublicIP \
    --allocation-method static
```

Get the assigned IP address:
```bash
az network public-ip show --resource-group $RESOURCE_GROUP_NODE --name MICO-PublicIP --query ipAddress --output tsv
```

Public static IP address: 40.68.22.7

*Side note:* It was not possible to expose the deployment `mico-admin` via the command `kubectl expose deployment mico-admin --external-ip=40.68.22.7 ...` because it created a complete new IP address instead of just using the specified external IP address.

**Dashboard:**

The dashboard of `mico-admin` is accessible via [http://40.68.22.7/dashboard](http://40.68.22.7/dashboard).
