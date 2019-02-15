# Azure

## Azure CLI

**Install:**
[Install Azure CLI with apt](https://docs.microsoft.com/de-de/cli/azure/install-azure-cli-apt?view=azure-cli-latest)

```bash
AZ_REPO=$(lsb_release -cs) && \
echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list

curl -L https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -

sudo apt-get update && sudo apt-get install apt-transport-https azure-cli
```

**Login:**
```bash
az login
```

## Azure Kubernetes Service

### Knative test installation

Following instructions are based on the GitHub wiki [Knative Install on Azure Kubernetes Service (AKS)](https://github.com/knative/docs/blob/master/install/Knative-with-AKS.md).

**Create environment variables:**
```bash
export LOCATION=westeurope
export RESOURCE_GROUP=ust-mico-knative-test-group
export CLUSTER_NAME=knative-test-cluster
```

**Create a resource group for AKS:**
```bash
az group create --name $RESOURCE_GROUP --location $LOCATION
```

**Create a Kubernetes cluster using AKS:**

Enable AKS:
```bash
az provider register -n Microsoft.ContainerService
```

Find out what Kubernetes versions are currently available:
```bash
az aks get-versions --location $LOCATION --output table
```

Create the AKS cluster:
```bash
az aks create --resource-group $RESOURCE_GROUP \
--name $CLUSTER_NAME \
--generate-ssh-keys \
--kubernetes-version 1.11.4 \
--node-vm-size Standard_DS3_v2 \
--node-count 1
```

Information about the used VM `Standard_DS3_v2` [DSv2-series](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-general#dsv2-series):
* vCPU: 4
* Memory: 14 GiB
* Temp storage: 28 GiB
* Costs [estimation](https://azure.com/e/630fd452808c4ba08028007dffdd2c75): 0,229 € per hour (per node) -> 5,51 € per day (per node)

Default node code is 3. Manual scaling is possible (see [Scaling](#scaling)).

Configure kubectl to use the new cluster:
```bash
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --admin
```

Verify your cluster is up and running
```bash
kubectl get nodes
```

**Installing Istio:**
```bash
# Install Istio
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.2.2/istio.yaml

# Label the default namespace with istio-injection=enabled.
kubectl label namespace default istio-injection=enabled
```

Monitor the Istio components:
```bash
kubectl get pods --namespace istio-system --watch
```

**Install Knative components:**

*Install either Knative Serving + Build or only Knative Build.*

Installing Knative Serving and Build components:
```bash
# Install Knative and its dependencies:
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.2.2/release.yaml

# Monitor the Knative components:
kubectl get pods --namespace knative-serving --watch
kubectl get pods --namespace knative-build --watch
```

Installing Knative Build only:
```bash
# Install Knative Build and its dependencies:
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.2.2/build.yaml

# Monitor the Knative components:
kubectl get pods --namespace knative-build --watch
```

**Deployment:**

How to deploy applications with Knative is described in chapter [Knative](./knative.md).

### Scaling

**Manual scaling of AKS nodes:**
```bash
az aks scale --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --node-count 1
```

### SSH into AKS cluster

[Connect with SSH to Azure Kubernetes Service (AKS) cluster nodes for maintenance or troubleshooting](https://docs.microsoft.com/de-de/azure/aks/ssh)

```bash
az aks show --resource-group myResourceGroup --name myAKSCluster --query nodeResourceGroup -o tsv

az vm list --resource-group MC_ust-mico-knative_knative-cluster_westeurope -o table

az vm user update \
  --resource-group MC_ust-mico-knative_knative-cluster_westeurope \
  --name aks-nodepool1-35709218-0 \
  --username azureuser \
  --ssh-key-value ~/.ssh/id_rsa.pub

az vm list-ip-addresses --resource-group MC_ust-mico-knative_knative-cluster_westeurope -o table

kubectl run -it --rm aks-ssh --image=debian

apt-get update && apt-get install openssh-client -y

kubectl cp ~/.ssh/id_rsa aks-ssh-66cf68f4c7-4pz6m:/id_rsa

chmod 0600 id_rsa

ssh -i id_rsa azureuser@10.240.0.4
```

### Helful commands

Cleaning up Kubernetes cluster:
```bash
az aks delete --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --yes --no-wait
```

Delete cluster and context from kubectl config:
```bash
kubectl config delete-cluster $CLUSTER_NAME
kubectl config delete-context $CLUSTER_NAME-admin
```

Open Kubernetes Dashboard:
```bash
az aks browse --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```

## Azure Container Registry (ACR)

**All images are stored in DockerHub in public. ACR is not used anymore.**

At the beginning of the development of MICO the Azure Container Registry (ACR) was used to store the container images of the MICO system components.
Because DockerHub was used as the container registry for the images build by MICO anyway, there was no reason two use two container registry at the same time.

**Preparation:**

Set environment variable:
```bash
export ACR_NAME=ustmicoregistry
```

Login:
```bash
az acr login --name $ACR_NAME
```

### Create Azure Container Registry (ACR)

**Create ACR:**
```bash
az acr create --resource-group $RESOURCE_GROUP \
--name $ACR_NAME \
--sku Basic
```

For more information see [Tutorial: Deploy and use Azure Container Registry](https://docs.microsoft.com/en-us/azure/aks/tutorial-kubernetes-prepare-acr).

### Grant Kuberentes Cluster access to ACR

When you create an AKS cluster, Azure also creates a service principal. We will use this auto-generated service principal for authentication with the ACR registry.

**[Grant AKS read access to ACR](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-auth-aks#grant-aks-access-to-acr)**:
```bash
#!/bin/bash

AKS_RESOURCE_GROUP=$RESOURCE_GROUP
AKS_CLUSTER_NAME=$CLUSTER_NAME
ACR_RESOURCE_GROUP=$RESOURCE_GROUP
ACR_NAME=$ACR_NAME

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

**List image tags (latest 10 tags of repository `mico-core`):**
```bash
az acr repository show-tags --name $ACR_NAME --repository mico-core --top 10 --orderby time_desc --output table
```

**Get credentials:**
```bash
az acr credential show --name $ACR_NAME
```

**Get the fully qualified name of the ACR login server:**
```bash
az acr list --resource-group $RESOURCE_GROUOP --query "[].{acrLoginServer:loginServer}" --output table
```

[Authenticate with Azure Container Registry from Azure Kubernetes Service](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-auth-aks)

**Get the id of the service principal configured for AKS:**
```bash
az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query "servicePrincipalProfile.clientId" --output tsv
```

**Get the ACR registry resource id:**
```bash
az acr show --name $ACR_NAME --resource-group $RESOURCE_GROUP --query "id" --output tsv
```

## Static IP addresses

List all static IP addresses:
```bash
az network public-ip list
```

Show IP address of `MICO-PublicIP`:
```bash
az network public-ip show --resource-group $RESOURCE_GROUP_NODE --name MICO-PublicIP --query ipAddress --output tsv
```

Create new static IP address with name `MICO-PublicIP`:
```bash
az network public-ip create --resource-group $RESOURCE_GROUP_NODE --name MICO-PublicIP --allocation-method static
```

Delete static IP address with name `kubernetes-aa7005...`:
```bash
az network public-ip delete -g $RESOURCE_GROUP -n kubernetes-aa70055e8fc8911e88c2ad233499c1fd
```
