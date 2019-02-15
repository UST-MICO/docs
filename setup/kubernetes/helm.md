# Helm

Currently we are only using Helm for templating. Therefore only the client is required (Tiller as the server-side component is not required).

## Install Helm client

There are several possibilities [how to install Helm](https://docs.helm.sh/using_helm/#installing-the-helm-client).
One of them is by using a script:

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```

To initialize Helm without installing Tiller on your cluster, use
```bash
helm init --client-only
```

## Templating with Helm

1. Fetching the chart templates (`helm fetch`)
2. Rendering the template with configurable values (`helm template`)
3. Applying the result to the cluster (`kubectl apply`)

## Istio

Helm is great for templating. Escpecially for creating an adapted Istio installation. The official Istio [installation steps](https://istio.io/docs/setup/kubernetes/minimal-install/#installation-steps) also recommends the usage of using `helm template`.

For more information see section [Istio](./istio.md).

## Setup Helm & Tiller

**Like already mentioned, Tiller is currently not used!**

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


## Troubleshooting

* Getting error *Error: found in requirements.yaml, but missing in charts/ directory: ...*
  
  Update dependencies (Istio in version 1.1.0-snapshot.4 is used):
  ```bash
  helm init --client-only
  helm repo add istio.io https://storage.googleapis.com/istio-release/releases/1.1.0-snapshot.4/charts
  helm dependency update install/kubernetes/helm/istio
  ```
