# Kubernetes Cluster

MICO runs on the Azure Kubernetes Service (AKS).

## First steps

To connect to the cluster you need two command-line tools:
* *kubectl*: [Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* Azure CLI *az*: [Install the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

To be able to execute the cluster related commands set following environment variables:
```bash
export LOCATION=westeurope
export RESOURCE_GROUP=ust-mico-resourcegroup
export CLUSTER_NAME=ust-mico-cluster
export ACR_NAME=ustmicoregistry
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
--node-count 1
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

How to access the Kubernetes dashboard is described in the Kubernetes Cluster Resource in Microsoft Azure.

If Cluster uses RBAC: Usage of Kubernetes Dashboard is restricted and most of the operations are not permitted. To be able to operate through the dashboard, execute the following command:
```bash
kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
```

**WARNING:** This configuration of a ClusterRoleBinding may lead to security issues, there is no additional authentication and the dashboard is publicly accessable! [Access the Kubernetes web dashboard in Azure Kubernetes Service (AKS)](https://docs.microsoft.com/en-us/azure/aks/kubernetes-dashboard#for-rbac-enabled-clusters)

### Public Access

**Create static IP address:**

For a stable, public access, a static IP address is required. This IP address with the name `MICO-PublicIP` was generated by:
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

**Create service for remote access:**

To access the MICO dashboard (frontend) over the internet, a `Service` for the running pods is required.
Create it with following yaml configuration:
```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    run: mico-admin
  name: mico-admin-service
  namespace: default
spec:
  loadBalancerIP: 40.68.22.7
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    run: mico-admin
```

To check the external endpoints of a service:
```bash
kubectl get svc mico-admin-service
```

The dashboard of the service `mico-admin` is accessible via [http://40.68.22.7/dashboard](http://40.68.22.7/dashboard).

*Side note:* It was not possible to expose the deployment `mico-admin` via the command `kubectl expose deployment mico-admin --external-ip=40.68.22.7 ...` because it created a complete new IP address instead of just using the passed external IP address.


## Container Registry

MICO uses the Azure Container Registry (ACR) to store container images.

**Login:**
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


## Knative Build

[Knative Build](https://github.com/knative/build) is used to achieve a *Source-to-Image workflow*. [Knative Serving](https://github.com/knative/serving) is currently not used.

### Installation

**[Installing Knative](https://github.com/knative/docs/blob/master/install/Knative-with-AKS.md#installing-knative):**
```bash
# Install Knative Build and its dependencies:
kubectl apply --filename https://github.com/knative/build/releases/download/v0.3.0/release.yaml

# Monitor the Knative components:
kubectl get pods --namespace knative-build
```

### Authentication to Azure Container Registry

There are already at least two `Service Principals` for following requirements:
* Read access from the AKS cluster to `pull` images
* Write access from the Jenkins Pipeline to `push` images

Now it's required to create another `Service Principal` with write access to the registry.

[Create a `Service Principal`](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-auth-service-principal#create-a-service-principal):
```bash
#!/bin/bash
SERVICE_PRINCIPAL_NAME=ust-mico-acr-knative-build

ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)

SP_PASSWD=$(az ad sp create-for-rbac --name $SERVICE_PRINCIPAL_NAME --scopes $ACR_REGISTRY_ID --role contributor --query password --output tsv)
SP_APP_ID=$(az ad sp show --id http://$SERVICE_PRINCIPAL_NAME --query appId --output tsv)

echo "Service principal ID: $SP_APP_ID"
echo "Service principal password: $SP_PASSWD"
```

Create a new namespace `build-bot`:
```bash
kubectl create namespace build-bot
```

Create a `Secret` with the name `build-bot-acr-secret`:
```bash
kubectl create secret docker-registry build-bot-acr-secret --docker-server $ACR_NAME.azurecr.io --docker-username $SP_APP_ID --docker-password $SP_PASSWD --namespace=build-bot
```

Create a `ServiceAccount` with the name `build-bot-acr`:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-bot-acr
  namespace: build-bot
secrets:
- name: build-bot-acr-secret
```

### Authentication to Docker Hub

Create a new namespace `build-bot` (if not yet created):
```bash
kubectl create namespace build-bot
```

Create a `Secret` with the name `build-bot-dockerhub-secret`:
```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/basic-auth
metadata:
  name: build-bot-dockerhub-secret
  namespace: build-bot
  annotations:
    build.knative.dev/docker-0: https://index.docker.io/v1/
data:
  # Use 'echo -n "username" | base64 -w 0' to generate this string
  username: BASE64_ENCODED_USERNAME
  # Use 'echo -n "password" | base64 -w 0' to generate this string
  password: BASE64_ENCODED_PASSWORD
```

Create a new `Service Account` manifest which is used to link the build process to the secret. Save this file as `service-account.yaml`:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-bot-dockerhub
  namespace: build-bot
secrets:
- name: build-bot-dockerhub-secret
```

Apply the manifest files to your cluster:
```bash
kubectl apply -f docker-secret.yaml && kubectl apply -f service-account.yaml
```

### Build

**Examples:**

Build and push to ACR:
```yaml
apiVersion: build.knative.dev/v1alpha1
kind: Build
metadata:
  name: build-hello
  namespace: build-bot
spec:
  serviceAccountName: build-bot-acr
  source:
    git:
      url: https://github.com/dgageot/hello.git
      revision: master
  steps:
  - name: build-and-push
    image: gcr.io/kaniko-project/executor:debug # debug includes a shell
    args:
    - --dockerfile=/workspace/Dockerfile
    - --destination=ustmicoregistry.azurecr.io/samples/hello:v1
```

Build and push to DockerHub:
```yaml
apiVersion: build.knative.dev/v1alpha1
kind: Build
metadata:
  name: build-hello
  namespace: build-bot
spec:
  serviceAccountName: build-bot-dockerhub
  source:
    git:
      url: https://github.com/dgageot/hello.git
      revision: master
  steps:
  - name: build-and-push
    image: gcr.io/kaniko-project/executor
    args:
    - --dockerfile=/workspace/Dockerfile
    - --destination=docker.io/ustmico/hello
```


## Setup Helm & Tiller

(<https://docs.microsoft.com/en-us/azure/aks/kubernetes-helm>)

* Install Helm on your local machine (<https://github.com/helm/helm#install>)
* Create a service account and an according role-binding for Tiller (necessary in a RBAC-enabled cluster).

    `helm-rbac.yaml`:
    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
    name: tiller
    namespace: kube-system
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
    name: tiller
    roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: ClusterRole
    name: cluster-admin
    subjects:
    - kind: ServiceAccount
        name: tiller
        namespace: kube-system
    ```

* Apply it to Kubernetes with:
    ```bash
    kubectl apply -f helm-rbac.yaml
    ```

* Alternatively via command-line:
    ```bash
    kubectl create serviceaccount --namespace kube-system tiller \
    kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller \
    helm init --service-account tiller --upgrade
    ```
