# Consumer Agent

## Description

This repo contains:
- a master helm chart allowing to deploy a **Consumer** agent using a single command.
- templates of values.yaml files used inside *Integration* environment under `app-values` folder

## Pre-Requisites

| Pre-Requisites         |     Version     | Description                                                                                                                                     |
| ---------------------- |     :-----:     | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| DNS sub-domain name    |       N/A       | This domain will be used to address all services of the agent. <br/> example: `*.dataconsumer01.int.simpl-europe.eu`                            |  
| Kubernetes Cluster     | 1.29.x or newer | Other version *might* work but tests were performed using 1.29.x version                                                                        |
| nginx-ingress          | 1.10.x or newer | Used as ingress controller. <br/> Other version *might* work but tests were performed using 1.10.x version. <br/> Image used: `registry.k8s.io/ingress-nginx/controller:v1.10.0`  |
| cert-manager           | 1.15.x or newer | Used for automatic cert management. <br/> Other version *might* work but tests were performed using 1.15.x version. <br/> Image used: `quay.io/jetstack/cert-manager-controller::v1.15.3` |
| Hashicorp Vault        | 1.17.x or newer | Other version *might* work but tests were performed using 1.17.x version. <br/> Image used: `hashicorp/vault:1.17.2`                            |
| argocd                 | 2.11.x or newer | Used as GitOps tool . App of apps concept. <br/> Other version *might* work but tests were performed using 2.11.x version. <br/> Image used: `quay.io/argoproj/argocd:v2.11.3` |

## Installation

### Prerequisites

#### Create the Namespace
Once the namespace variable is set, you can create the namespace using the following kubectl command:

`kubectl create namespace consumer1`

#### Verify the Namespace
To ensure that the namespace was created successfully, run the following command:

`kubectl get namespaces`
<br/>This will list all the namespaces in your cluster, and you should see the one you just created listed.

#### Create volumes

Two volumes needs to be created manually at the moment:
* nfs-storage-pvc-xsfc
* nfs-storage-pvc-sdapibe

This will be fixed in future versions.

### Deployment using ArgoCD

You can easily deploy the agent using ArgoCD. All the values mentioned in the sections below you can input in ArgoCD deployment. The repoURL gets the package directly from code.europa.eu.
targetRevision is the package version. 

When you create it, you set up the values below (example values)

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: 'consumer01-deployer'                            # name of the deploying app in argocd
spec:
  project: default
  source:
    repoURL: 'https://code.europa.eu/api/v4/projects/903/packages/helm/stable'
    path: '""'
    targetRevision: 0.3.1
    helm:
      values: |
        values:
          branch: develop                                 # branch of repo with values - this is develop by default
        secretEngine: dev-int                             # container for your secrets in vault
        project: default
        namespaceTag: consumer01                          # identifier of deployment and part of fqdn
        domainSuffix: int.simpl-europe.eu                 # last part of fqdn
        argocd:
          appname: consumer01-iaa                         # name of generated argocd app 
          namespace: argocd                               # namespace of your argocd
        cluster:
          address: https://kubernetes.default.svc
          namespace: consumer01-iaa                       # where the app will be deployed
          kubeStateHost: kube-prometheus-stack-kube-state-metrics.devsecopstools.svc.cluster.local:8080    # link to kube-state-metrics svc
        authority:
          keycloakClientID: federated-catalogue           # name of the client in authority keycloak
          keycloakSecret: clientsecretfromkeycloak        # secret of that client (from its credentials)
          namespaceTag: authority1                        # namespace tag of target authority
        monitoring:
          enabled: true                                   # "true" enables the deployment of ELK stack for monitoring
    chart: consumer
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: consumer01-iaa                             # where the package will be deployed
```

### Manual deployment

#### Files preparation

The suggested way for deployment, is to unpack the released package to a folder on a host where you have kubectl and helm available and configured. 

There is basically one file that you need to modify - values.yaml. 
There are a couple of variables you need to replace - described below. The rest you don't need to change.

```
authority:
  namespaceTag: authority1                        # namespace tag of target authority
  keycloakClientID: federated-catalogue           # name of the client in authority keycloak
  keycloakSecret: clientsecretfromkeycloak        # secret of that client (from its credentials)

argocd:
  appname: consumer01-iaa                         # name of generated argocd app 
  namespace: argocd                               # namespace of your argocd

project: default                                  # Project to which the namespace is attached

cluster:
  address: https://kubernetes.default.svc
  namespace: consumer01-iaa                       # where the package will be deployed
  kubeStateHost: kube-prometheus-stack-kube-state-metrics.devsecopstools.svc.cluster.local:8080    # link to kube-state-metrics svc

namespaceTag: consumer01                          # identifier of deployment and part of fqdn
domainSuffix: int.simpl-europe.eu                 # last part of fqdn

values:
  repo_URL: https://code.europa.eu/simpl/simpl-open/development/agents/consumer.git  # repo URL
  branch: develop                                                                    # branch of code in repo
```

### Deploy the namespace
Deploying a dedicated namespace, such as **consumer**, helps isolate resources and applications within a Kubernetes cluster.

Filling the namespace with content requires the following activity:

Go to master charts directory:

`cd .\charts\`

Now you can deploy the namespace:

`helm install consumer . `

:rotating_light: :rotating_light: :rotating_light: **Attention!!!** :rotating_light: :rotating_light: :rotating_light: <br>
<b><i>After installing the namespace, there are services that connect using the TLS protocol (e.g. EJBCA). In the current phase of application development, this element must be configured manually.
The entire procedure is described in confuence:</i></b>

https://confluence.simplprogramme.eu/display/SIMPL/EJBCA+Configuration

<b><i>For the namespace consumer to work correctly, it is necessary to perform the actions described in the link above.</i></b>

## Change the namespace

The process of implementing changes is analogous to deploying the namespace for the first time:

`helm upgrade consumer . `

## Delete the deployment:

`helm uninstall consumer .` 

## Monitoring

ELK stack for monitoring is added with this release.  
Its deployment can be disabled by switch the value monitoring.enabled to false.  
When it's enabled, after the stack is deployed, you can access the ELK stack UI by https://kibana.**namespacetag**.**domainsuffix**  
Default user is "elastic", its password can be extracted by kubectl command. `kubectl get secret elastic-elasticsearch-es-elastic-user -o go-template='{{.data.elastic | base64decode}}' -n {namespace}`

# Troubleshooting
If you encounter issues during deployment, check the following:

- Ensure that ArgoCD is properly set up and running.
- Verify that the test01 namespace exists in your Kubernetes cluster.
- Check the ArgoCD application logs and Helm error messages for specific issues.
