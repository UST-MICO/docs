# Azure Kubernetes Service

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

## Knative installation

Following instructions are based on the GitHub wiki [Knative Install on Azure Kubernetes Service (AKS)](https://github.com/knative/docs/blob/master/install/Knative-with-AKS.md).

**Create a resource group for AKS:**
```bash
export LOCATION=westeurope
export RESOURCE_GROUP=ust-mico-knative-group
export CLUSTER_NAME=knative-cluster

az group create --name $RESOURCE_GROUP --location $LOCATION
```

**Create a Kubernetes cluster using AKS:**

Enable AKS:
```bash
az provider register -n Microsoft.ContainerService
```

Create the AKS cluster:
```bash
az aks create --resource-group $RESOURCE_GROUP \
--name $CLUSTER_NAME \
--generate-ssh-keys \
--kubernetes-version 1.11.3 \
--node-vm-size Standard_DS3_v2
```

Side note about the resources of `Standard_DS3_v2` [DSv2-series](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes-general#dsv2-series):
* vCPU: 4
* Memory: 14 GiB
* Temp storage: 28 GiB

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
kubectl apply --filename https://raw.githubusercontent.com/knative/serving/v0.2.0/third_party/istio-1.0.2/istio.yaml

# Label the default namespace with istio-injection=enabled.
kubectl label namespace default istio-injection=enabled
```

Monitor the Istio components:
```
kubectl get pods --namespace istio-system --watch
```

**Install Knative components:**
```bash
# Install Knative and its dependencies:
kubectl apply --filename https://github.com/knative/serving/releases/download/v0.2.0/release.yaml

# Monitor the Knative components:
kubectl get pods --namespace knative-serving --watch
kubectl get pods --namespace knative-build --watch
```

**Helful commands:**

Cleaning up cluster:
```bash
az aks delete --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --yes --no-wait
```

Open Kubernetes Dashboard:
```bash
az aks browse --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```
