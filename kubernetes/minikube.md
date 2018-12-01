# Minikube

Minikube can be used to run a single node Kubernetes cluster on your local machine.

## Installation

Read the official instructions how to [Install Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/).

### On Windows 10

**Pre-reqs:**
* Hyper-V enabled

**Installation with chocolatey:**
```bash
choco install minikube -y
choco install kubernetes-cli -y
```

**Network Configuration:**

Using PowerShell (run as Administrator):
```bash
Get-NetAdapter
New-VMSwitch -Name "ExternalSwitch" -NetAdapterName "Ethernet 2" -AllowManagement $True
```

## Important commands

**Run minikube:**
```bash
minikube start --vm-driver hyperv --hyperv-virtual-switch "ExternalSwitch"
```

**Stop minikube:**
```bash
minikube stop
```

**Get IP address:**
```bash
minikube ip
```

**Open dashboard:**
```bash
minikube dashboard
```

**Update minikube:**
```bash
minikube update-check
```

**Cleaning up cluster:**
```bash
minikube delete
```

**SSH (Minikube command):**
```bash
minikube ssh
```

**SSH (alternative command on WSL):**
```bash
chmod 0600 "$(wslpath -u "$(minikube ssh-key)")"
ssh -a -i "$(wslpath -u "$(minikube ssh-key)")" -l docker "$(minikube ip)"
```

## Knative installation on Minikube

The following instructions are based on the official GitHub wiki [Knative Install on Minikube](https://github.com/knative/docs/blob/master/install/Knative-with-Minikube.md).

Minikube has to be started with priviledged rights (root / administrator).
Important note: At least **8 GiB of free memory** are required. Also 4 vCPUs are required, otherwise later on some pods will fail (status "OutOfcpu").

Before starting Minikube, ensure that a virtual network switch called "ExternalSwitch" exists.
If you are using Mac OS or Linux you have to change the `vm-driver`. For Linux specify `--vm-driver=kvm2`. Omitting the `--vm-driver` option will use the default driver.

**Start Minikube with Hyper-V:**
```bash
minikube start --memory=8192 --cpus=4 \
  --kubernetes-version=v1.11.4 \
  --vm-driver=hyperv \
  --hyperv-virtual-switch "ExternalSwitch" \
  --bootstrapper=kubeadm \
  --extra-config=apiserver.enable-admission-plugins="LimitRanger,NamespaceExists,NamespaceLifecycle,ResourceQuota,ServiceAccount,DefaultStorageClass,MutatingAdmissionWebhook"
```

Confirm that your kubectl context is pointing to the new cluster:
```bash
kubectl config current-context
```

**Install Istio:**
```bash
# Install Istio
curl -L https://raw.githubusercontent.com/knative/serving/v0.2.0/third_party/istio-1.0.2/istio.yaml \
  | sed 's/LoadBalancer/NodePort/' \
  | kubectl apply --filename -

# Label the default namespace with istio-injection=enabled.
kubectl label namespace default istio-injection=enabled
```

Monitor the Istio components:
```bash
kubectl get pods --namespace istio-system --watch
```

**Install Knative Serving:**
```bash
curl -L https://github.com/knative/serving/releases/download/v0.2.0/release-lite.yaml \
  | sed 's/LoadBalancer/NodePort/' \
  | kubectl apply --filename -
```

Monitor the Knative components:
```bash
kubectl get pods --namespace knative-serving --watch
```

**Helpful commands:**

Get IP address and NodePort for the `knative-ingressgateway`:
```bash
echo $(minikube ip):$(kubectl get svc knative-ingressgateway --namespace istio-system --output 'jsonpath={.spec.ports[?(@.port==80)].nodePort}')
```

## Troubleshooting

* `minikube stop` does not work, command hangs and finally crashes. As a workaround use SSH instead [minikube issue 2914](https://github.com/kubernetes/minikube/issues/2914)

  **Poweroff via SSH:**
  ```bash
  minikube ssh "sudo poweroff"
  ```

* Kubernetes cluster shutsdown after a few minutes MiniKube VM still running. Under Windows 10 the Hyper-V functionality "Dynamic Memory" could be the problem [minikube issue 2326](https://github.com/kubernetes/minikube/issues/2326)
  
  **Disable Dynamic RAM allocation in Hyper-V:**
  ```bash
  Set-VMMemory minikube -DynamicMemoryEnabled $false
  ```

* If there is any problem with Hyper-V on Windows, you can try to restart the Hyper-V Virtual Machine Management service

  **Restarting Hyper-V Virtual Machine Management service:**
  ```bash
  net.exe stop vmms
  net.exe start vmms
  ```

* If you have problems to start Minikube, you can delete the cache (everything is lost!)

  **On Linux:**
  ```bash
  rm -rf ~/.minikube
  ```
  **On Windows:**
  ```bash
  rm -rf %USERPROFILE%/.minikube
  ```
