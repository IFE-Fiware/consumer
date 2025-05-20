monitor# Consumer Agent

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
        * [Secret for EDC](#secret-for-edc)
    * [Deployment](#deployment)
      * [Deployment using ArgoCD](#deployment-using-argocd)
      * [Manual deployment](#manual-deployment)
        * [Files preparation](#files-preparation)
        * [Deployment](#deployment)
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
In the current version, the automatic onboarding process has already been implemented using: init-participant-job. 
For this reason, manual onboarding activities are no longer necessary.

### Tools
| Pre-Requisites      |     Version     | Description                                                                                                                                                                               |
|---------------------|:---------------:|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| DNS sub-domain name |       N/A       | This domain will be used to address all services of the agent. <br/> example: `*.onsumer01.int.simpl-europe.eu`                                                                      |  
| Kubernetes Cluster  | 1.29.x or newer | Other version *might* work but tests were performed using 1.29.x version                                                                                                                  |
| nginx-ingress       | 1.10.x or newer | Used as ingress controller. <br/> Other version *might* work but tests were performed using 1.10.x version. <br/> Image used: `registry.k8s.io/ingress-nginx/controller:v1.10.0`          |
| cert-manager        | 1.15.x or newer | Used for automatic cert management. <br/> Other version *might* work but tests were performed using 1.15.x version. <br/> Image used: `quay.io/jetstack/cert-manager-controller::v1.15.3` |                                                                   |
| argocd              | 2.11.x or newer | Used as GitOps tool . App of apps concept. <br/> Other version *might* work but tests were performed using 2.11.x version. <br/> Image used: `quay.io/argoproj/argocd:v2.11.3`            |

## Installation

#### Create the Namespace

As an update from previous version, all the agent namespaces now are created automatically. 

#### Vault related tasks

You can access vault on https://vault.**commonnamespacetag**.**domainsuffix**
Root token can be found in common namespace, secret vault-unseal-keys, in key vault-root. 

##### Secret for EDC

Create a secret named "*namespace*-simpl-edc" replacing the data mentioned in the table with proper values. 

```
{
  "contractmanager_apikey": "apikey",
  "edc_datasource_default_password": "edc",
  "edc_datasource_policy_password": "edc",
  "edc_ionos_access_key": "accesskeystring",
  "edc_ionos_endpoint": "s3-eu-central-1.ionoscloud.com",
  "edc_ionos_endpoint_region": "de",
  "edc_ionos_secret_key": "secretkeystring",
  "edc_ionos_token": "tokenstring"
}
```

| Variable name                    |     Example         | Description              |
| ----------------------           |     :-----:         | ---------------          |
| contractmanager_apikey           | apikey              | api key string           |
| edc_datasource_default_password  | dbpassstring        | take the password from *namespace*-postgres-passwords vault secret, key *namespace*-edc |
| edc_datasource_policy_password   | dbpassstring        | take the password from *namespace*-postgres-passwords vault secret, key *namespace*-edc |
| edc_ionos_access_key             | accesskeystring     | Access key for S3        |
| edc_ionos_endpoint               | s3-eu-central-1.ionoscloud.com | S3 server url |
| edc_ionos_endpoint_region        | de                  | Two letter country code  |
| edc_ionos_secret_key             | secretkeystring     | Secret key for S3        |
| edc_ionos_token                  | tokenstring         | Token for S3 access      |

All the other necessary secrets are now created automatically with proper data.

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
    targetRevision: 1.3.2                   # version of package
    helm:
      values: |
        values:
          branch: v1.3.2                    # branch of repo with values - for released version it should be the release branch
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
          issuer: dev-int-dns01             # certificate issuer
        authority:
          namespaceTag: authority1          # namespace tag of target authority
        hashicorp:
          service: "https://vault.common.domainsuffix"  # link to your vault ingress (apply domain suffix)
          secretEngine: dev-int             # secret engine name created in vault
          role: dev-int-role                # role name in vault 
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
values:
  repo_URL: https://code.europa.eu/simpl/simpl-open/development/agents/consumer.git  # repo URL
  branch: v1.3.2                    # branch of repo with values - for released version it should be the release branch

project: default                   # Project to which the namespace is attached
namespaceTag: consumer01           # identifier of deployment and part of fqdn
domainSuffix: int.simpl-europe.eu  # last part of fqdn

argocd:
  appname: consumer01              # name of generated argocd app 
  namespace: argocd                # namespace of your argocd

cluster:
  address: https://kubernetes.default.svc
  namespace: consumer01            # where the package will be deployed
  commonToolsNamespace: common     # namespace where main monitoring stack is deployed
  issuer: dev-int-dns01            # certificate issuer

authority:
  namespaceTag: authority1         # namespace tag of target authority 

hashicorp:
  service: "https://vault.common.domainsuffix"  # link to your vault ingress (apply domain suffix)
  secretEngine: dev-int            # secret engine name created in vault
  role: dev-int-role               # role name in vault   
```

##### Deployment

After you have prepared the values file, you can start the deployment. 
Use the command prompt. Proceed to the folder where you have the Chart.yaml file and execute the following command. The dot at the end is crucial - it points to current folder to look for the chart. 

Now you can deploy the agent:

`helm install consumer . `

## Additional steps

In the current version, the automatic onboarding process has already been implemented using: init-participant-job.
For this reason, manual onboarding activities are no longer necessary.

### Monitoring

Filebeat components for monitoring are included in this release.   
Their deployment can be disabled by switching the value monitoring.enabled to false.

# Troubleshooting
If you encounter issues during deployment, check the following:

- Ensure that ArgoCD is properly set up and running.
- Verify that the namespace exists in your Kubernetes cluster.
- Check the ArgoCD application logs and Helm error messages for specific issues.
