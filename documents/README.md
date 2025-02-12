# Consumer Agent

<!-- TOC -->
* [Consumer Agent](#consumer-agent)
  * [Description](#description)
  * [Pre-Requisites](#pre-requisites)
    * [Onboarding](#onboarding)
    * [Tools](#tools)
  * [Installation](#installation)
    * [Prerequisites](#prerequisites)
      * [Create the Namespace](#create-the-namespace)
      * [Verify the Namespace](#verify-the-namespace)
      * [Vault related tasks](#vault-related-tasks)
    * [Deployment](#deployment)
      * [Deployment using ArgoCD](#deployment-using-argocd)
      * [Manual deployment](#manual-deployment)
        * [Files preparation](#files-preparation)
        * [Deployment](#deployment)
    * [Deploy the namespace](#deploy-the-namespace)
  * [Additional steps](#additional-steps)
    * [Monitoring](#monitoring)
* [Troubleshooting](#troubleshooting)
<!-- TOC -->

## Description

This repo contains:
- a master helm chart allowing to deploy a **Consumer** agent using a single command.
- templates of values.yaml files used inside *Integration* environment under `app-values` folder

## Pre-Requisites

### Onboarding

Prior to the installation of the consumer agent, make sure to follow the onboarding steps for a data space participant.

[Onboarding a Participant](https://code.europa.eu/simpl/simpl-open/development/iaa/charts/-/blob/develop/doc/0.7.x/ONBOARD.md?ref_type=heads#onboarding-a-participant)

### Tools
| Pre-Requisites      |     Version     | Description                                                                                                                                                                               |
|---------------------|:---------------:|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| DNS sub-domain name |       N/A       | This domain will be used to address all services of the agent. <br/> example: `*.dataconsumer01.int.simpl-europe.eu`                                                                      |  
| Kubernetes Cluster  | 1.29.x or newer | Other version *might* work but tests were performed using 1.29.x version                                                                                                                  |
| nginx-ingress       | 1.10.x or newer | Used as ingress controller. <br/> Other version *might* work but tests were performed using 1.10.x version. <br/> Image used: `registry.k8s.io/ingress-nginx/controller:v1.10.0`          |
| cert-manager        | 1.15.x or newer | Used for automatic cert management. <br/> Other version *might* work but tests were performed using 1.15.x version. <br/> Image used: `quay.io/jetstack/cert-manager-controller::v1.15.3` |
| Hashicorp Vault     | 1.17.x or newer | Other version *might* work but tests were performed using 1.17.x version. <br/> Image used: `hashicorp/vault:1.17.2`                                                                      |
| argocd              | 2.11.x or newer | Used as GitOps tool . App of apps concept. <br/> Other version *might* work but tests were performed using 2.11.x version. <br/> Image used: `quay.io/argoproj/argocd:v2.11.3`            |

## Installation

### Prerequisites

#### Create the Namespace
Once the namespace variable is set, you can create the namespace using the following kubectl command:

`kubectl create namespace consumer1`

#### Verify the Namespace
To ensure that the namespace was created successfully, run the following command:

`kubectl get namespaces`
<br/>This will list all the namespaces in your cluster, and you should see the one you just created listed.

#### Vault related tasks

##### Secret engine for Signer

Go to Vault UI and define new transit secret engine with path `transit/simpl` create encryption key `gaia-x-key1` with type `ed25519`.

##### Secret for Contract

One secret is needed, its naming syntax is "{{ .Release.Namespace }}-contract", it should be created in created before kv secret engine.
Its content is:

```
{
  "API_KEY": "apikey",
  "DBPASSWORD": "contract",
  "DB_URL": "jdbc:postgresql://postgresql.consumer03.svc.cluster.local:5432/contract",
  "DB_USER": "contract",
  "KAFKA_CLIENT_PASSWORDS": "contract"
}
```

Where you need to modify:

| Variable name                 |     Example         | Description     |
| ----------------------        |     :-----:         | --------------- |
| DB_USER            | contract | User for contract database |
| DBPASSWORD         | contract | Password for contract database  |
| DB_URL             | jdbc:postgresql://postgresql.consumer03.svc.cluster.local:5432/contract | Link to datasource |
| KAFKA_CLIENT_PASSWORDS  | password | Password for kafka connection |

##### Secret engine for EDC

A new KV v2 secret engine is needed for EDC. It should be named "edc". 
You need to create 5 secrets in it, their names are as below (example in brackets - "content" is the key name, value is after the colon)

- edc.ionos.access.key (content: suppliedstring)
- edc.ionos.endpoint (content: s3 server url)
- edc.ionos.endpoint.region (content: two letter country code)
- edc.ionos.secret.key (content: secretkeystring)
- edc.ionos.token (content: tokenstring)

### Deployment

#### Deployment using ArgoCD

You can easily deploy the agent using ArgoCD. All the values mentioned in the sections below you can input in ArgoCD deployment. The repoURL gets the package directly from code.europa.eu.
targetRevision is the package version. 

When you create it, you set up the values below (example values)

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: 'consumer01-deployer'           # name of the deploying app in argocd
spec:
  project: default
  source:
    repoURL: 'https://code.europa.eu/api/v4/projects/903/packages/helm/stable'
    path: '""'
    targetRevision: 1.1.0                   # version of package
    helm:
      values: |
        values:
          branch: v1.1.0                    # branch of repo with values - for released version it should be the release branch
        project: default
        namespaceTag: consumer01            # identifier of deployment and part of fqdn
        domainSuffix: int.simpl-europe.eu   # last part of fqdn
        argocd:
          appname: consumer01               # name of generated argocd app 
          namespace: argocd                 # namespace of your argocd
        cluster:
          address: https://kubernetes.default.svc
          namespace: consumer01             # where the app will be deployed
          commonToolsNamespace: common      # namespace where main monitoring stack is deployed
        authority:
          namespaceTag: authority1          # namespace tag of target authority
        hashicorp:
          token: "aHZzLnFPSHdxbTdnWnV1eDlZZEllYzY4bkt3Uw=="  # token to access the vault base64 encoded
          service: "http://vault-common03.common03.svc.cluster.local:8200"  # local service path to your vault
          secretEngine: dev-int             # secret engine name created in vault
    chart: consumer
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: consumer01                   # where the package will be deployed
```

#### Manual deployment

##### Files preparation

Another way for deployment, is to unpack the released package to a folder on a host where you have kubectl and helm available and configured. 

There is basically one file that you need to modify - values.yaml. 
There are a couple of variables you need to replace - described below. The rest you don't need to change.

```
project: default                   # Project to which the namespace is attached
namespaceTag: consumer01           # identifier of deployment and part of fqdn
authority:
  namespaceTag: authority1         # namespace tag of target authority 
domainSuffix: int.simpl-europe.eu  # last part of fqdn

argocd:
  appname: consumer01              # name of generated argocd app 
  namespace: argocd                # namespace of your argocd

cluster:
  address: https://kubernetes.default.svc
  namespace: consumer01            # where the package will be deployed
  commonToolsNamespace: common     # namespace where main monitoring stack is deployed

hashicorp:
  token: "aHZzLnFPSHdxbTdnWnV1eDlZZEllYzY4bkt3Uw=="  # token to access the vault base64 encoded
  service: "http://vault-common03.common03.svc.cluster.local:8200"  # local service path to your vault
  secretEngine: dev-int             # secret engine name created in vault

values:
  repo_URL: https://code.europa.eu/simpl/simpl-open/development/agents/consumer.git  # repo URL
  branch: develop                                                                    # branch of code in repo
```

##### Deployment

After you have prepared the values file, you can start the deployment. 
Use the command prompt. Proceed to the folder where you have the Chart.yaml file and execute the following command. The dot at the end is crucial - it points to current folder to look for the chart. 

Now you can deploy the agent:

`helm install consumer . `

## Additional steps

:rotating_light: :rotating_light: :rotating_light: **Attention!!!** :rotating_light: :rotating_light: :rotating_light: <br>
<b><i>After installing the agent, you need to get through the onboarding process. 
The entire procedure is described in the code repository:</i></b>

https://code.europa.eu/simpl/simpl-open/development/iaa/charts/-/blob/develop/doc/0.8.x/ONBOARD.md?ref_type=heads

### Monitoring

Filebeat components for monitoring are included in this release.   
Their deployment can be disabled by switching the value monitoring.enabled to false.

# Troubleshooting
If you encounter issues during deployment, check the following:

- Ensure that ArgoCD is properly set up and running.
- Verify that the namespace exists in your Kubernetes cluster.
- Check the ArgoCD application logs and Helm error messages for specific issues.
