# OpenFaaS

## Installation

We use the YAML files of the GitHub repository [openfaas/faas-netes](https://github.com/openfaas/faas-netes) for the installation of OpenFaaS modified for our use with the monitoring of other components.

Install it:
```bash
kubectl apply -f install/kubernetes/openfaas
```

## Function deployment

The fastest method to deploy a function is to use the OpenFaaS Function Store va the OpenFaaS UI. To deploy a custom function the OpenFaaS cli is nesseary. Both methods are described in the following sections.

### Function Store

1. To open the OpenFaaS UI use the button provided in the MICO UI. 

![Link to OpenFaaS UI from MICO](images/OpenFaaSlink.png "Link from MICO UI to OpenFaaS")

The nessesary credentials are configured in the [setup script](https://github.com/UST-MICO/mico/blob/master/install/kubernetes/setup.sh).

2. After the login is completed the UI offers the 'Deploy A New Function' button. 
The button opens a list and it is possible to select a function from the store.

![Function store](images/FunctionStore.png "Function store")

3. After selecting a function, use the 'Deploy' button on the bottom of the list.

### Custom Function



### Further information

* [Website](https://www.openfaas.com/)
* [Tutorial](https://docs.openfaas.com/deployment/kubernetes/)
* [OpenFaaS Doc](https://docs.openfaas.com/)
* [REST API](https://raw.githubusercontent.com/openfaas/faas/master/api-docs/swagger.yml)
