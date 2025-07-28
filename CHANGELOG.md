# Changelog
All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [2.2.0] - 2025-08-01
- Updated many components to implement Consumer version 2.2.0.

## [2.1.0] - 2025-07-17
- Updated many components to implement Consumer version 2.1.0.
- Remove component simpl-cli


### Simpl Cloud gateway (Tier 1)

#### 2.0.0 (2025-06-03)

#### Added
- Added Configuration Properties section
- Added the API Documentation section (merge request)


### Users Roles

#### 2.0.0 (2025-06-03)

#### Added
- Drop table identity attribute
- Added Configuration Properties section
- Added API Documentation section
- Maven goal to automatically add openapi from simpl-api-iaa 
- Search roles by multiple names

#### Changed
- Removed deprecated API
- Removed ingress spec from chart
- Removed keycloak.client-to-realm-role-migration properties
- SIMPL-11765 Remove version v0 APIs fom IAA components


### SIMPL FE

#### 2.0.1 (2025-06-05)

#### Changed
- Fixed the display of assigned identity attributes and those for the user

#### 2.0.0 (2025-06-03)

#### Added
- SIMPL-10530
- SIMPL-10533
- SIMPL-11766
- SIMPL-8228
- SIMPL-8227

#### Changed
- SIMPL-8338


### TLS Gateway (Tier 2)

#### 2.0.0 (2025-06-03)
No changes.


### Authentication Provider

#### 2.0.0 (2025-06-03)

#### Added
- Maven goal to automatically add openapi from simpl-api-iaa
- SIMPL-12367 Integrate the reviewed APIs into the Keycloak Authenticator extension
- Added Configuration Properties section
- Added the API Documentation section

#### Changed
- Removed microservice.users-roles.url property 
- Removed deprecated API 
- CredentialInitializerImpl 
- Removed ingress spec from chart 
- SIMPL-11765 Remove version v0 APIs fom IAA components


### xsfc-advsearch-be

#### 1.11.1 (2025-06-30)

#### Changed
- SIMPL-13505

#### 1.11.0 (2025-06-19)

#### Added
- SIMPL-13521 Added ArgoCD manifests.

#### Changed
- SIMPL-14205
- error responses aligned to belgif problem model
- spring upgraded from 3.4.4 to 3.4.5 to fix tomcat security issue on


### edc connector adapter

#### 1.3.0 (2025-06-19)

#### Added
- added new key in values.yaml to specify a different service name in open
- SIMPL-13521 Added ArgoCD manifests.

#### Changed
- error responses aligned to belgif problem model
- aligned to simpl-data1-common version 1.1.0 to support belgif problem

#### Fixed
- fixed RegistationControlle register() error handling for missing


### contract-consumption-be

#### 1.7.0 (2025-05-29)

#### Added
- SIMPL-10304 
- SIMPL-13198 
- SIMPL-13198 
- SIMPL-12999

#### Changed
- SIMPL-12727
- SIMPL-13575 Service Account management


### simpl-edc

#### 1.0.7 (2025-07-04)

#### Added
- SIMPL-14638 added logger

#### Changed
- SIMPL-14638 changed auth provider url
- SIMPL-14638 solved sonar issues
- SIMPL-14638 removed creds
- SIMPL-14638 update SIMPL-EDC with new version of the simpl-http library
- SIMPL-14638 updated connector-core to 1.1.5


### simpl-catalogue-client

#### 1.2.1 (2025-05-09)

#### Added
- Classes to elements to aid testing
- Unit tests to increase code coverage to 80%


### Filebeat

#### 0.1.15 (2025-06-05)

#### Changed
- Edited dashboard for heartbeat
- SIMPL-13099
- SIMPL-12666 Removed unused fields 
- Changed configuration because of change in business pods names.
- SIMPL-12666 Remove unused fields
