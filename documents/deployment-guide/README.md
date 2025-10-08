# Consumer Agent

<!-- TOC -->
- [Consumer Agent](#consumer-agent)
  - [Description](#description)
  - [Pre-Requisites](#pre-requisites)
    - [Onboarding](#onboarding)
    - [Tools](#tools)
  - [DNS entries](#dns-entries)
  - [Installation](#installation)
    - [Vault related tasks](#vault-related-tasks)
      - [Secret for EDC](#secret-for-edc)
    - [Deployment](#deployment)
      - [Deployment using ArgoCD](#deployment-using-argocd)
      - [Manual deployment](#manual-deployment)
        - [Files preparation](#files-preparation)
        - [Deployment](#deployment)
  - [Additional steps](#additional-steps)
    - [Monitoring](#monitoring)
- [Troubleshooting](#troubleshooting)

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
| DNS sub-domain name |       N/A       | This domain will be used to address all services of the agent. <br/> example: `*.consumer01.example.com` | 
| external-dns    | bitnami/external-dns:0.16.1 | Currently version docker.io/bitnami/external-dns:0.16.1-debian-12-r should be used as externaldns. Unfortunately, using a newer version caused DNS to work incorrectly. |  
| Kubernetes Cluster  | 1.29.x or newer | Other version *might* work but tests were performed using 1.29.x version                                                                                                                  |
| nginx-ingress       | 1.10.x or newer | Used as ingress controller. <br/> Other version *might* work but tests were performed using 1.10.x version. <br/> Image used: `registry.k8s.io/ingress-nginx/controller:v1.10.0`          |
| cert-manager        | 1.15.x or newer | Used for automatic cert management. <br/> Other version *might* work but tests were performed using 1.15.x version. <br/> Image used: `quay.io/jetstack/cert-manager-controller:v1.15.3` |                                                                   |
| argocd              | 2.11.x or newer | Used as GitOps tool . App of apps concept. <br/> Other version *might* work but tests were performed using 2.11.x version. <br/> Image used: `quay.io/argoproj/argocd:v2.11.3`            |

### DNS entries

| Entry Name | Entries |
| :-----------: | :-------------------------------------------------------------------------------------------------: |
| catalogue-ui | catalogue-ui.(namespace).example.com |
| contract-consumption-be| contract-consumption-be.(namespace).example.com |
| edc-connector-adapter | edc-connector-adapter.(namespace).example.com |
| simpl-edc-ingress | edc.(namespace).example.com/management<br>edc.(namespace).example.com/api<br>edc.(namespace).example.com/protocol<br>edc.(namespace).example.com/public<br>  edc.(namespace).example.com/control |
| simpl-fe-ingress | participant.fe.(namespace).example.com/users-roles <br>participant.fe.(namespace).example.com/participant-utility |
| simpl-ingress | participant.be.(namespace).example.com | 
| xfsc-advsearch-be | xfsc-advsearch-be.(namespace).example.com | 

## Installation

### Vault related tasks

You can access vault on <https://secrets.**commonnamespacetag**.**domainSuffix**>
Root token can be found in common namespace, secret secrets-root-token, in key token. 

The description of using vault is in a separate document:

<https://code.europa.eu/simpl/simpl-open/development/agents/common_components/-/blob/main/documents/Using_Vault.md>

Before you proceed with the next steps related to accessing your Vault and changing its contents, please read the document above.<BR>

##### Secret for simpl-edc

Edit the key for Infrastructure-be named "*consumer01*-simpl-edc" replacing "01" in "consumer01" with the appropriate entry and the data mentioned in the table with proper values.

You need to modify:

| Variable name                    |     Example         | Description              |
| ----------------------           |     :-----:         | ---------------          |
| edc_ionos_access_key             | accesskeystring     | Access key for S3 - please contact IONOS to get the correct value. Currently the best way is to send an email requesting this data to Paulo Cabrita: paulo.cabrita@ionos.com |
| edc_ionos_endpoint               | s3-eu-central-1.ionoscloud.com | S3 server url |
| edc_ionos_endpoint_region        | de                  | Two letter country code  |
| edc_ionos_secret_key             | secretkeystring     | Secret key for S3 - please contact IONOS to get the correct value. Currently the best way is to send an email requesting this data to Paulo Cabrita: paulo.cabrita@ionos.com |
| edc_ionos_token                  | tokenstring         | Token for S3 access - please contact IONOS to get the correct value. Currently the best way is to send an email requesting this data to Paulo Cabrita: paulo.cabrita@ionos.com |

All the other necessary secrets are now created automatically with proper data.

### Deployment

### Deployment using ArgoCD

You can easily deploy the agent using ArgoCD. All the values mentioned in the sections below you can input in ArgoCD deployment. The repoURL gets the package directly from code.europa.eu.
targetRevision is the package version. 

In the example below, please replace the marked versions with the ones applicable to your environment.

Please pay special attention to the namespace names: common01, authority01, consumer01 and dataprovider01 and replace them with yours, as well as replace the domain name example.com and the occurrence of the value example itself.

```YAML
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: 'consumer01-deployer'           # name of the deploying app in argocd
spec:
  project: default
  source:
    repoURL: 'https://code.europa.eu/api/v4/projects/903/packages/helm/stable'
    path: '""'
    targetRevision: v2.3.0                  # version of package
    helm:
      values: |
        values:
          branch: v2.3.0                    # branch of repo with values - for released version it should be the release branch
        project: default
        namespaceTag: 
          consumer: consumer01              # identifier of deployment and part of fqdn for this agent
          authority: authority01            # identifier of deployment and part of fqdn for authority
          common: common01                  # identifier of deployment and part of fqdn for common components
        domainSuffix: example.com           # last part of fqdn
        resourcePreset: default             # set to "low" to disable requests of resources
        argocd:
          appname: consumer01               # name of generated argocd app 
          namespace: argocd                 # namespace of your argocd
        cluster:
          address: https://kubernetes.default.svc
          namespace: consumer01             # where the app will be deployed
          commonToolsNamespace: common01    # namespace where main monitoring stack is deployed
          issuer: dev-prod                  # certificate issuer
        authority:
          namespaceTag: authority01         # namespace tag of target authority
        secrets:
          secretEngine: example             # secret engine name created in vault
          role: example-role                # role created in vault for access
        monitoring:
          enabled: true                     # should monitoring be disabled
    chart: consumer
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: consumer01                   # where the package will be deployed

```

### Manual deployment

##### Files preparation

Another way for deployment, is to unpack the released package to a folder on a host where you have kubectl and helm available and configured.

There is basically one file that you need to modify - values.yaml. 
There are a couple of variables you need to replace - described below. The rest you don't need to change.

```YAML
values:
  branch: v2.3.0                    # branch of repo with values - for released version it should be the release branch
project: default
namespaceTag: 
  consumer: consumer01              # identifier of deployment and part of fqdn for this agent
  authority: authority01            # identifier of deployment and part of fqdn for authority
  common: common01                  # identifier of deployment and part of fqdn for common components
domainSuffix: example.com           # last part of fqdn
resourcePreset: default             # set to "low" to disable requests of resources
argocd:
  appname: consumer01               # name of generated argocd app 
  namespace: argocd                 # namespace of your argocd
cluster:
  address: https://kubernetes.default.svc
  namespace: consumer01             # where the app will be deployed
  commonToolsNamespace: common01    # namespace where main monitoring stack is deployed
  issuer: dev-prod                  # certificate issuer
authority:
  namespaceTag: authority01         # namespace tag of target authority
secrets:
  secretEngine: example             # secret engine name created in vault
  role: example-role                # role created in vault for access
monitoring:
  enabled: true                     # should monitoring be disabled
```

#### Deployment

After you have prepared the values file, you can start the deployment.
Use the command prompt. Proceed to the folder where you have the Chart.yaml file and execute the following command. The dot at the end is crucial - it points to current folder to look for the chart.

Now you can deploy the agent:

`helm install consumer . `

After starting the deployment synchronization process, the expected applications in ArgoCD will be created.

Initially, the status observed e.g. in ArgoCD will indicate the creation of new pods.

Be patient!... Depending on the configuration, this step can take up to 30 minutes!

At the end, all pods should be created correctly:

<img src="images/consumer_ArgoCD01.png" alt="ArgoCD01" width="600"><BR>


## Additional steps


### Onboarding

In the current version, after the deployment process is complete, a manual onboarding deployment process is required. 

The steps are described in the document:
https://code.europa.eu/simpl/simpl-open/development/iaa/documentation/-/blob/main/versioned_docs/2.2.x/ONBOARD.md

### Monitoring

Filebeat components for monitoring are included in this release.
Their deployment can be disabled by switching the value monitoring.enabled to false.

## Troubleshooting

If you encounter issues during deployment, check the following:

- Ensure that ArgoCD is properly set up and running.
- Verify that the namespace exists in your Kubernetes cluster.
- Check the ArgoCD application logs and Helm error messages for specific issues.